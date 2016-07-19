---
title: Hello Maven中央仓库
date: 2016-07-19 10:30:00
categories:
- Java
tags:
- Maven
comments: true
---
前段时间做课题相关测试涉及到排序算法，排序算法作为一个程序员必须要求掌握算法，在很多环境下还是蛮有用的，所以就利用点时间整理了下现有常用的排序算法，详见Github [sort-tool](https://github.com/szpssky/sort-tool),包含了冒泡排序、选择排序、归并排序、快速排序、插入排序、希尔排序、堆排序等，有时间打算继续完善下这个工具包。

为了方便以后项目中可能会使用到这些排序算法，避免重复造轮子，就进行了整理和实现，打包成了jar文件方便使用，并将其上传至Maven中央仓库，虽然代码比较简单，但也是我的第一个Maven构建，小小纪念一下。

这里记录下发布至Maven中央仓库的过程。
<!-- more -->
## 提交issue
Maven 中央仓库一直是由Sonatype公司出资维护的，所以要想提交至中央仓库首先要在该网址上提交一个issue，工作人员会对你进行审核等，其中group id的域名必须是你本人所拥有否则审核不会通过，其实没有自己的域名也没关系，可以使用com.github.你的用户名作为group id.在此注册并提交issue, https://issues.sonatype.org/secure/Dashboard.jspa

## 等待审批
当工作人员告诉你可以做一次发布时即表示通过审核啦

## GPG密钥
[Download and install GPG Tool](https://www.gnupg.org/download/)

- 使用`gpg --gen-key`命令生成密钥，此过程会让你输入用户名、邮箱、注释信息。
- 使用`gpg --list-keys`命令查看生成的密钥信息,形如:`pub   2048R/7E97F37D 2016-07-18`,pub代表公钥，sub为私钥。
- 使用`gpg --keyserver hkp://pool.sks-keyservers.net --send-keys 7E97F37D`将公钥上次至验证服务器，以便其他用户进行校对使用。
- 使用`gpg --keyserver hkp://pool.sks-keyservers.net --recv-keys 7E97F37D`可以查看密钥是否上传成功，通常执行上一步上传后需要等待10分钟左右才能查看到。

## 修改setting.xml
Maven的发布方式有很多种详见[OSSRH Guide](http://central.sonatype.org/pages/ossrh-guide.html)，这里通过Maven方式进行发布所以需要配置setting.xml。setting.xml分为本地设置和全局设置，本地设置只对当前用户有效全局则对所有用户均起左右，本地的配置文件通常在`~/m2/`文件夹中即本地仓库的目录，全局的setting.xml文件在Maven主程序的`conf`文件夹中。在setting.xml的`<servers> </servers>`标签下添加如下配置:
``` xml 
<server> 
    <id>oss</id> 
    <username>sonatpye注册的用户名</username> 
    <password>sonatype注册的密码</password> 
</server> 
```

## 修改pom.xml文件
发布的项目Maven pom.xml的配置必须要求有name description url licenses developers scm这些字段,参考如下:
``` xml
    <groupId>com.tifosi-m</groupId>
    <artifactId>sort-tool</artifactId>
    <version>1.0</version>
    <packaging>jar</packaging>

    <name>sort-tool</name>
    <description>
        This is Sort Algorithm tool,you can easy to use many sort algorithm.
    </description>
    <url>https://github.com/szpssky/sort-tool</url>

    <licenses>
        <license>
            <name>The Apache Software License, Version 2.0</name>
            <url>http://www.apache.org/licenses/LICENSE-2.0.txt</url>
        </license>
    </licenses>

    <developers>
        <developer>
            <name>szpssky</name>
            <email>szpssky@gmail.com</email>
        </developer>
    </developers>

    <scm>
        <connection>scm:git@github.com:szpssky/sort-tool.git</connection>
        <developerConnection>
            scm:git@github.com:szpssky/sort-tool.git
        </developerConnection>
        <url>git@github.com:szpssky/sort-tool.git</url>
    </scm>
```
其次发布jar的话还需要包括source、javadoc、gpg签名，这些可以通过相应maven插件完成，这里的配置可以使用profile，这样当不进行发布时这些过程不会被执行。参考如下:
``` xml
<profiles>
        <profile>
            <id>release</id>
            <distributionManagement>
                <snapshotRepository>
                    <id>oss</id>
                    <url>https://oss.sonatype.org/content/repositories/snapshots/</url>
                </snapshotRepository>
                <repository>
                    <id>oss</id>
                    <url>https://oss.sonatype.org/service/local/staging/deploy/maven2/</url>
                </repository>
            </distributionManagement>
            <build>
                <plugins>
                    <!-- Source -->
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-source-plugin</artifactId>
                        <version>3.0.1</version>
                        <executions>
                            <execution>
                                <id>attach-sources</id>
                                <goals>
                                    <goal>jar-no-fork</goal>
                                </goals>
                            </execution>
                        </executions>
                    </plugin>
                    <!-- Javadoc -->
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-javadoc-plugin</artifactId>
                        <version>2.10.3</version>
                        <executions>
                            <execution>
                                <phase>package</phase>
                                <goals>
                                    <goal>jar</goal>
                                </goals>
                            </execution>
                        </executions>
                    </plugin>
                    <!-- Gpg Signature -->
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-gpg-plugin</artifactId>
                        <version>1.6</version>
                        <executions>
                            <execution>
                                <id>sign-artifacts</id>
                                <phase>verify</phase>
                                <goals>
                                    <goal>sign</goal>
                                </goals>
                            </execution>
                        </executions>
                    </plugin>
                </plugins>
            </build>
        </profile>
    </profiles>
```
## 发布到OSS
一切准备就绪就可以正式发布了，使用`mvn clean deploy -P release -Dgpg.passphrase=生成密钥时的密码`命令即可自动将要发布的内容上传至服务器上，-P参数代表使用profile，后面的release是使用的profile对应的id，上传成功后登陆[OSS](https://oss.sonatype.org)，点击左侧staging Repositories后查找你上传的构建,如果没什么问题点击close按钮将其状态由open变为close，此时系统会进行自动检测，如果没有问题状态会变为closed，如果有问题返回修改pom配置重新上传。最后点击release将构建状态从closed变为release。

## 回到issue
回到之前开的issue中，按要求第一次发布完成后需要告诉工作人员，你已经完成了第一次发布，他们会告诉你在一段时间后自动的同步到Maven中央仓库中并关闭这个issue，至此一次Maven发布完成，此外以后在你申请的group id下发布构建不需要使用issue进行审核了，直接发布即可。

