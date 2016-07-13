---
title: Java NIO初探
date: 2016-06-22 16:08:27
tags:
- Java
categories:
- Java NIO
comments: true
---
因为研究项目实验论证方面的需要所以接触到了Java NIO,之前因为使用普通的Java IO导致性能十分低下，实验数据相当不理想。至此了解到了Java NIO技术。
>新的输入/输出 (NIO) 库是在 JDK 1.4 中引入的。NIO 弥补了原来的 I/O 的不足，它在标准Java代码中提供了高速的、面向块的I/O。通过定义包含数据的类，以及通过以块的形式处理这些数据，NIO不用使用本机代码就可以利用低级优化，这是原来的 I/O 包所无法做到的。

<!-- more -->
NIO由以下几个核心部分组成：
- Channels
- Buffers
- Selectors

## Java IO和Java NIO的区别

| IO 	        | NIO           |
|:-------------:|:-------------:|
| 面向流	        | 面向缓存		|
| 阻塞			| 非阻塞			|
| 无				| 选择器			|

### 面向流与面向缓存
所谓面向流即读取时每次从流读取一个或多个字符直到所有字符全部读取完毕，你不能前后移动的从流中读取某一数据，而Java NIO则是面向缓存，所有数据读取到一个缓存区中，你可以在缓存区中任意移动来读取所在位置的数据，使得该方法对数据的处理更加灵活。
### 阻塞与非阻塞
阻塞型IO，每当进行读取或写入操作的时候该线程被阻塞直到读取或写入完成，在此期间当前线程将不能做其他事情了，只能等待完成，而非阻塞型IO，到发出读或写指令的时候，当前线程可以执行其他命令，而不会被阻塞。

### 选择器
Java NIO的选择器允许一个单独的线程来监视多个通道。
## Java NIO使用
``` Java
/**
* 写文件
*/
File srcFile = new File("/root/spill_out/" + count + ".txt");
RandomAccessFile raf = new RandomAccessFile(srcFile, "rw");
FileChannel fileChannel = raf.getChannel();
ByteBuffer rBuffer = ByteBuffer.allocate(32 * 1024 * 1024);
try {
    	int size = mappedKeyValue.size();
        for (int i = 0; i < size; i++) {
        KeyValue<String, Integer> keyValue = mappedKeyValue.remove(0);
        rBuffer.put((keyValue.getKey().toString() + " " + keyValue.getValue().toString() + "\n").getBytes());
		}
}catch (Exception e) {
 	e.printStackTrace();
}
rBuffer.flip();
fileChannel.write(rBuffer);
fileChannel.close();
raf.close();
```
该代码是一个写文件操作，通道FileChannel的实例只有唯一的方法可以获取，那就是调用.getChannel()方法，ByteBuffer为字节行缓存区，需要手动分配缓存区大小，使用put()方法往缓存区中写入数据。
``` Java
/**
* 读文件
*/
RandomAccessFile raf = new RandomAccessFile(new File(filename), "r");
FileChannel fc = raf.getChannel();
MappedByteBuffer mbb =  fc.map(FileChannel.MapMode.READ_ONLY, 0, fc.size());
StringBuffer sbf = new StringBuffer();
while(mbb.remaining()>0){
	char data = (char)mbb.get();
	if(data!='\n'){
		sbf.append(data);
	}else{
		wcMR.addKeyValue(0,sbf.toString());
		sbf.setLength(0);
		if(count == 800000){
			wcMR.startMap();
			count=0;
		}
		count++;
	}
}
fc.close();
raf.close();
```
这里为了提高超大文件读取的性能采用MappedByteBuffer，即内存映射文件，通过map()方法将文件映射成内存映射文件，通过调用get()方法获取数据，该示例采用按行读取数据。

### 参考[IBM developerWorks](http://www.ibm.com/developerworks/cn/education/java/j-nio/j-nio.html)

转载请注明出处，谢谢。
