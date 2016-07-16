---
title: Java NIO用allocateDirect()方法创建直接缓存区导致的释放问题
date: 2016-07-16 20:00:00
categories:
- Java NIO
tags:
- Java
comments: true
---
在Java nio中针对缓存区的直接分配有两种方法分别为`allocate(int capacity)`和`allocateDirect(int capacity)`,区别是前者分配的是堆内内存，后者分配的是堆外内存。当数据处理时操作系统总是将数据读取到系统内存中，再由JVM将数据复制到堆内内存中,所以使用和allocateDirect分配的直接缓存区使得I/O效率大大高于分配间接缓存区,但是使用直接缓存区在分配和销毁时代价比堆内存要大，所以在使用上应具体分析。以下是Sun文档对直接缓存区的描述:

>给定一个直接字节缓冲区，Java 虚拟机将尽最大努力直接对它执行本机 I/O 操作。也就是说，它会在每一次调用底层操作系统的本机 I/O 操作之前(或之后)，尝试避免将缓冲区的内容拷贝到一个中间缓冲区中(或者从一个中间缓冲区中拷贝数据)。

<!-- more -->
正因为这种特性，使用直接缓存区时会存在堆外内存释放的问题，因为是堆外内存所以不会在普通GC（或Minor GC）阶段进行垃圾回收,只有在Full GC时才会回收这部分内存，这就导致一个问题可能会造成堆外内存溢出异常：当虚拟机运行时如果给JVM充足的堆空间，因为堆空间充足所以并不会触发Full GC来进行垃圾回收，当程序不断申请堆外内存时，系统的本地内存将越来少而此时使用完成的缓存区又得不到回收，最终将导致OutofMemoryError:Direct buffer memory异常。

在Java中缓冲区对象Buffer并没有直接方法可以手动销毁缓存区,而查看Java源代码`DirectBuffer`接口可以看到：
``` java
package sun.nio.ch;

import sun.misc.Cleaner;
public interface DirectBuffer {
    long address();

    Object attachment();

    Cleaner cleaner();
}
```
此接口中存在一个Cleaner对象，再来看ByteBuffer具体实现类`DirectByteBuffer`中的构造方法：
``` java
DirectByteBuffer(int cap) {                   
        super(-1, 0, cap, cap);
        boolean pa = VM.isDirectMemoryPageAligned();
        int ps = Bits.pageSize();
        long size = Math.max(1L, (long)cap + (pa ? ps : 0));
        Bits.reserveMemory(size, cap);

        long base = 0;
        try {
            base = unsafe.allocateMemory(size);
        } catch (OutOfMemoryError x) {
            Bits.unreserveMemory(size, cap);
            throw x;
        }
        unsafe.setMemory(base, size, (byte) 0);
        if (pa && (base % ps != 0)) {
            // Round up to page boundary
            address = base + ps - (base & (ps - 1));
        } else {
            address = base;
        }
        cleaner = Cleaner.create(this, new Deallocator(base, size, cap));
        att = null;
    }
```
当创建一个DirectByteBuffer时将会产生创建一个Cleaner对象,我们通过调用该对象的`clean()`方法即可以释放通过`allocateDirect(int capacity)`分配的堆外内存。
``` java
if (rBuffer == null) 
	return;
Cleaner cleaner = ((DirectBuffer) rBuffer).cleaner();
if (cleaner != null) 
	cleaner.clean();
```

转载请注明出处，谢谢。
