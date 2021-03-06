---
title: I/O 流
date: 2017-03-22 22:10
categories: Java
tags: 
- 待完善
- I/O 流
---

Java I/O 流大致分为：字节流 文件字节流 缓冲流 字符流 对象流。

<!--more-->

## 文件操作
java.io.File 类表示文件，也就是说通过 File 类在程序中我们可以操作硬盘上的文件或目录。

File 的每一个实例可以表示硬盘上的一个文件或目录。

通过 File 可以
1.访问文件或目录的属性信息（名字、大小、隐藏等）
2.操作文件或目录（创建、删除）
3.访问一个目录下的所有子项
>但是不能操作文件内容

## 核心方法 ##
isFile() 
          测试此抽象路径名表示的文件是否是一个标准文件。

isDirectory() 
          测试此抽象路径名表示的文件是否是一个目录。

getName() 
          返回由此抽象路径名表示的文件或目录的名称。

length() 
          返回由此抽象路径名表示的文件的长度。占用字节量

exists() 
          测试此抽象路径名表示的文件或目录是否存在。

delete() 
          删除此抽象路径名表示的文件或目录。如果此路径名表示一个目录，则该目录必须为空才能删除。 

	System.out.println(File.separator);
	//创建文件或目录
	File file = new File("D:" + File.separator);

mkdir() 
          创建此抽象路径名指定的目录。

mkdirs() 
          创建此抽象路径名指定的目录，包括所有必需但不存在的父目录也会创建。

listFiles() 
          返回一个抽象路径名数组，这些路径名表示此抽象路径名表示的目录中的文件。

>"."表示当前类所在项目的根目录

>File.separator 匹配不同系统的分隔符，比如 windows 下表示 "\"


## 输入和输出流 ##
当我们编写程序时想读取一个文件或者将程序中某些数据写入一个文件中，这时，我们就要使用输入和输出流。

输入流：用来读取数据。

输出流：用来写出数据。


## 字节流
InputStream 是所有字节输入流的父类。定义了各种读取方法
read() 
          从输入流中读取数据的下一个字节。以 int 形式返回，返回的是 0-255 范围内的 int 字节值，若返回 -1， 则读取到末尾，处理效率低，基本不用。

public int read(byte[] b)
         throws IOException 从输入流中读取一定数量的字节，并将其存储在缓冲区数组 b 中。以整数形式返回实际读取的字节数。在输入数据可用、检测到文件末尾或者抛出异常前，此方法一直阻塞。
如果 b 的长度为 0，则不读取任何字节并返回 0；否则，尝试读取至少一个字节。如果因为流位于文件末尾而没有可用的字节，则返回值 -1；否则，至少读取一个字节并将其存储在 b 中。 

将读取的第一个字节存储在元素 b[0] 中，下一个存储在 b[1] 中，依次类推。读取的字节数最多等于 b 的长度。设 k 为实际读取的字节数；这些字节将存储在 b[0] 到 b[k-1] 的元素中，不影响 b[k] 到 b[b.length-1] 的元素。


OutputStream 是所有字节输出流的父类，其定义写出方法:

public abstract void write(int b)
                    throws IOException 将指定的字节写入此输出流。write 的常规协定是：向输出流写入一个字节。要写入的字节是参数 b 的八个低位。b 的 24 个高位将被忽略。
OutputStream 的子类必须提供此方法的实现。


public void write(byte[] b)
           throws IOException 将 b.length 个字节从指定的 byte 数组写入此输出流。write(b) 的常规协定是：应该与调用 write(b, 0, b.length) 的效果完全相同。

## 文件字节流
FileOutputStream 是文件的字节输出流，可以以字节为单位将数据写入文件。

FileOutputStream 有两个常用构造方法：

public FileOutputStream(File file)
                 throws FileNotFoundException 创建一个向指定 File 对象表示的文件中写入数据的文件输出流。创建一个新 FileDescriptor 对象来表示此文件连接。
首先，如果有安全管理器，则用 file 参数表示的路径作为参数来调用 checkWrite 方法。 

如果该文件存在，但它是一个目录，而不是一个常规文件；或者该文件不存在，但无法创建它；抑或因为其他某些原因而无法打开，则抛出 FileNotFoundException。 


public FileOutputStream(String name)
                 throws FileNotFoundException 创建一个向具有指定名称的文件中写入数据的输出文件流。创建一个新 FileDescriptor 对象来表示此文件连接。
首先，如果有安全管理器，则用 name 作为参数调用 checkWrite 方法。 

如果该文件存在，但它是一个目录，而不是一个常规文件；或者该文件不存在，但无法创建它；抑或因为其他某些原因而无法打开它，则抛出 FileNotFoundException。 


public void close()
           throws IOException 关闭此文件输出流并释放与此流有关的所有系统资源。此文件输出流不能再用于写入字节。
如果此流有一个与之关联的通道，则关闭该通道。 


使用以上两种构造期创建的文件字节输出流向指定文件写出数据，
若指定的文件以及包含了内容，那么当使用 fos 对其 写入就是覆盖操作，当前的文件的内容全部清空。

若想在文件中将原有的数据保存，在其之后追加新数据，则需要在构造器中添加第二个 boolean 参数，当参数为 true 时则追加，false 则覆盖。


FileInputStream 是文件字节输入流：
使用该流可以以字节为单位从文件中读取数据。





## 缓冲流 ##
缓冲是高级流，提高读写效率。但是缺乏即时性

### BufferedOutputStream 是缓冲输出流，用来写入。
内部维护一个缓冲区，每当我们通过该流写出数据时，都会现将数据存入一个缓冲区，当缓冲区已满的时候，缓冲流才会将数据一次性写出。
关闭流之前，缓冲输出流会将缓冲区内容一次性写出。

public void flush()
           throws IOException 刷新此缓冲的输出流。这迫使所有缓冲的输出字节被写出到底层输出流中。


### BufferedInputStream 是缓冲输入流，用来读取。
内部维护了一个缓冲区（字节数组），使用该流在读取一个字节时，该流尽可能地一次性读取若干个字节并存放到缓冲区，然后注意的将字节返回，直到缓冲区的数据全部读取完毕，才再次去读若干字节。
这样可以减少读取次数，从而提高读取效率。




## 对象流
对象存在于内存中，有时候需要将对象保存到硬盘上（持久化），这个时候我们需要：
将对象转换为字节序列，而这个过程称之为**对象序列化**。将字节序列转换为对象，这个过程称之为**反序列化**

ObjectOutputStream：用来对对象进行序列化的输出流。

实现对象序列化的方法为 writeObject(Object o),可以将给定的对象转换为字节序列后写出，没有返回值。

transient 关键字可以对象瘦身效果，通过该属性，可以修饰类中不需要序列化的属性。那么该属性在进行序列化是会被忽略，从而达到瘦身效果。


ObjectInputStream 是用来对对象进行反序列化的输入流。
实现对象反序列化的方法为
readObject()，可以从流中读取字节序列，将字节序列转换为对象，并返回该对象。
当使用 ObjectInputStream 对一个已经序列化的对象进行反序列化的时候，首先会检查该对象的版本号与当前类的版本号是否一致。不一致则直接反序列化失败。

### 序列化
ObjectOutputStream 在对象进行序列化的时候有一个要求，需要序列化的对象所属的类必须实现 serializable 接口，实现该接口不需要重写或定义任何方法，只是作为序列化的一个标识。

通常实现接口的类需要提供一个常量，serialVersionUID 该常量表示类的版本，该值直接决定当前对象在进行反序列化时成功与否。如果不写，Java 也会通过某种算法提供一个默认值（不建议）。



## 字符流 ##
底层依然是字节流，以 char 为单位进行读写数据。

###　Reader(抽象类)：是字符输入流的父类。
public int read()
         throws IOException 读取单个字符。在字符可用、发生 I/O 错误或者已到达流的末尾前，此方法一直阻塞。
用于支持高效的单字符输入的子类应重写此方法。 （基本不用）
返回：
作为整数读取的字符，范围在 0 到 65535 之间 (0x00-0xffff)，如果已到达流的末尾，则返回 -1 


public int read(char[] cbuf)
         throws IOException 将字符读入数组。在某个输入可用、发生 I/O 错误或者已到达流的末尾前，此方法一直阻塞。
返回：
读取的字符数，如果已到达流的末尾，则返回 -1

### Writer(抽象类)：是字符输出流的父类。
public void write(int c)
           throws IOException 写入单个字符。要写入的字符包含在给定整数值的 16 个低位中，16 高位被忽略。
用于支持高效单字符输出的子类应重写此方法。 （基本不用）

参数：
c - 指定要写入字符的 int。 


public void write(char[] cbuf)
           throws IOException 写入字符数组。

参数：
cbuf - 要写入的字符数组 



public void write(String str)
           throws IOException 写入字符串。

参数：
str - 要写入的字符串 





## 转换流：
OutputStreamWreiter 字符输出流。
使用该流可以设置字符集，并按照指定字符集将字符转换为对应的字节通过流写出。

InputStreamReader 字符输入流
使用该流可以设置字符集，并按照指定字符集，从流中按照该编码将字节数据转换为字符并读取。





### PrintWriter
PrintWriter 是具有自动行刷新的缓冲字符输出流。提供了丰富的构造方法，具体请查看 API 文档。

常用构造器
public PrintWriter(OutputStream out,
                   boolean autoFlush)通过现有的 OutputStream 创建新的 PrintWriter。此便捷构造方法创建必要的中间 OutputStreamWriter，后者使用默认字符编码将字符转换为字节。 

参数：
out - 输出流
autoFlush - boolean 变量；如果为 true，则 println、printf 或 format 方法将刷新输出缓冲区


public PrintWriter(Writer out,
                   boolean autoFlush)创建新 PrintWriter。 

参数：
out - 字符输出流
autoFlush - boolean 变量；如果为 true，则 println、printf 或 format 方法将刷新输出缓冲区
>特点，可以安航写字符串，内部默认嵌套一个 bufferedreader 作为


### BufferedReader
BufferedReader 缓冲字符输入流。按行读。内部提供了缓冲区，可以提高读取效率。

BufferedReader 只有两个构造方法。

public BufferedReader(Reader in,
                      int sz)创建一个使用指定大小输入缓冲区的缓冲字符输入流。 只能处理其他字符输入流。

参数：
in - 一个 Reader
sz - 输入缓冲区的大小 


方法：
public String readLine()
                throws IOException 读取一个文本行。通过下列字符之一即可认为某行已终止：换行 ('\n')、回车 ('\r') 或回车后直接跟着换行。

返回：
包含该行内容的字符串，不包含任何行终止符，如果已到达流末尾，则返回 null