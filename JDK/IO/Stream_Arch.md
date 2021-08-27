# Java I/O 流知识体系

原文：https://www.toutiao.com/a6998437404717908510/



## 1 I/O，

​        I/O (Input/Output) 流，是Java实现输入/输出的基础，他可以方便地实现数据的输入和输出操作。

### 1.1 同步与异步、阻塞与非阻塞

* **同步**：一个任务在完成之前不做其他操作，必须等待
* **异步**：一个任务在完成之前，可以进行其他操作
* **阻塞**：是相对于CPU来说的，挂起当前线程，不能做其他操作，只能等待
* **非阻塞**：无需挂起当前线程，可以去执行其他操作

### 1.2 什么是BIO

​        **BIO，同步并阻塞**，**服务器实现模式为：一个连接一个线程**，即客户端有连接请求时，服务器端就需要启动一个线程进行处理，没有处理完之前，此线程不能做其他操作（如果是单线程的情况下，传输的文件很大怎么办？），对于多连接，可以通过连接迟机制来改善。**BIO方式适用于连接数目比较小且固定的架构**，这种方式对服务器资源要求比较高，并发局限于应用中，JDK1.4之前的唯一选择。但是这种程序直观，易于理解。

### 1.3 什么是NIO

​        **NIO，同步非阻塞**，**服务器实现模式为：一个连接一个线程**，即客户端发送的连接请求都会注册到多路复用器上，多路复用器轮询到连接有 **I/O** 请求时才启动一个线程进行处理。**NIO 方式适用于连接数目多且连接比较短（轻操作）的架构**，比如聊天服务器，并发局限于应用中，编程比较复杂，JDK1.4后开始支持。

### 1.4 什么是AIO

​        **AIO，异步非阻塞**，**服务器实现模式为一个有效请求一个线程**，客户端的 I/O 请求都是由操作系统先完成了再通知服务器应用去启动线程进行处理。**AIO 方式适用于连接数目多且连接比较长（重操作）的架构**，比如相册服务器，充分调用操作系统参与并发操作，编程比较复杂，JDK1.7之后开始支持。

​        AIO 属于 NIO 包中的类实现，其实 IO 主要分为 BIO 和 NIO，AIO 只是附加品，解决 I/O 不能异步的实现。在以前，很少有 Linux 系统支持 AIO， Windows 的 IOCP 就是该 AIO 模型。但是现在的服务器一般都支持 AIO 操作。

### 1.5 I/O 流分类

1. **字节流**和**字符流**

   根据流操作的数据单位的不同，可以分为字节流和字符流

2. **输入流**和**输出流**

   根据流传输方向的不同，又可以分为输入流和输出流

3. **节点流**和**处理流**

   根据流的功能不同，可以分为节点流和处理流

### 1.6 I/O流结构

​        Java 中的 I/O 流主要是定义在 java.io 包中，其中有4个类为流的顶级类，分别是 *InputStream* 和 *OutputStream*、*Reader* 和 *Writer*。

![1](./images/Stream_Arch/1.png)

#### 1.6.1 说明

​        *InputStream* 和 *OutputStream* 是字节流，而 *Reader* 和 *Writer* 是字符流；

​       *InputStream* 和 *Reader* 是输入流，而 *OutputStream* 和 *Writer* 是输出流；

## 2 字节流

​        在计算机中，无论是文本、图片、音频还是视频，所有文件都是以二进制（字节）形式存在的，I/O流中针对字节的输入/输出提供了一系列的流，统称为字节流。

### 2.1 说明

​        字节流是程序中最常用的流。

​        在JDK中，所有的字节输入流都继承自*InputStream*，所有的字节输出流都继承自*OutputStream*。

### 2.2 InputStream 和 outputSteam示意图

![2](./images/Stream_Arch/2.jpg)

​        InputStream 被看成一个输入管道，OutputSteam 被看成一个输出管道，数据通过 InputSteam 从源设备输入到程序，通过 OutputStream 从程序输出到目标设备，从而实现数据的传输。

### 2.3 InputStream 的常用方法

| 方法声明                             | 功能描述                                                     |
| ------------------------------------ | ------------------------------------------------------------ |
| int read()                           | 从输入流读取**一个8位的字节**，把它转换为 0 ~ 255 之间的整数，并返回这一整数。**当没有可用字节时，返回 -1** |
| int read(byte[] b)                   | 从输入流读取若干字节，把它们保存到参数 b 指定的字节数组中，返回的整数表示读取字节的数目 |
| int read(byte[] b, int off, int Len) | 从输入流读取若干字节，把它们保存到参数 b 指定的字节数组中，off 指定字节数组开始保存数据的起始下标，len 表示读取的字节数目 |
| void close()                         | 关闭此输入流并释放与该流关联的所有系统资源                   |

说明：

* 前三个 read 方法都是用来读数据的，分按字节读取和按字节数组读取
* 进行 I/O 操作时，应该调用 close 方法关闭流，从而释放当前 I/O 流所占的系统资源

### 2.4 OutputStream 的常用方法

| 方法声明                               | 功能描述                                                   |
| -------------------------------------- | ---------------------------------------------------------- |
| void write()                           | 向输出流写入**一个字节**                                   |
| void write(byte[] b)                   | 把参数 b 指定的字节数组的所有字节写到输出流                |
| void write(byte[] b, int off, int Len) | 将制定 byte 数组中从偏移量 off 开始的 len 个字节写入输出流 |
| void flush()                           | 刷新此输出流并强制写出所有缓冲的输出字节                   |
| void close()                           | 关闭此输出流并释放与该流关联的所有系统资源                 |

说明：

* 前三个 write 方法都是用来写数据的，分按字节写入和按字节数组写入
* flush 方法用来将当前输出流缓冲区（通常是字节数组）中的数据强制写入到目标设备，此过程称为**刷新**
* close 方法是用来关闭流并释放与当前IO流相关的系统资源

### 2.5 InputStream 与 OutputStream 的继承体系

#### 2.5.1 InputStream

![3](./images/Stream_Arch/3.png)

#### 2.5.2 OutputStream

![4](./images/Stream_Arch/4.png)

### 2.6 字节流读写文件

​        针对文件的读写操作，JDK提供了两个类：FileInputStream 和 FileOutputStream。

​        FileInputStream 是 InputStream 的子类，它是操作文件的字节输入流，专门用于读取文件中的数据。从文件读取数据是重复的操作，因此需要通过循环来实现数据的持续读取。

#### 2.6.1 读取文件示例

​        本示例用于读取test.txt文件的内容

```java
...
FileInputStream in = new FileInputStream("test.txt");
int b = 0;
while( (b = in.read()) != -1) {
  System.out.println(b);
}
in.close();
```

注意：

​        在读取文件时，必须保证文件在相应的目录下存在，并且可读，否则会抛出FileNotFoundException。

#### 2.6.2 写入文件示例

​        本示例把字符串内容写入文件 out.txt：

```java
...
FileOutputStream out = new FileOutputStream("out.txt");
String str = "hello";
out.write(str.getBytes());
out.close();
```

注意：

​        通过 FileOutputStream 向一个已经存在的文件中写入数据，该文件中的数据首先会被清空，再写入新的数据。若希望在已经存在的文件内容之后追加新内容，则可以使用构造函数 *FileOutputStream(String fileName, boolean append)* 来创建文件输出流对象，并把append参数的值设置为 *true*。如：

```java
...
FileOutputStream out = new FileOutputStream("out.txt", true);
String str = " world";
out.write(str.getBytes());
out.close();
```

执行后，out.txt的内容为： hello world

#### 2.6.3 close 的处理

​        I/O 流在进行数据读写操作时会出现异常，为了保证 I/O 流的 close() 方法一定被执行，释放占用的系统资源，通常会将关闭流的操作写在 finally 代码块中：

```java
finally {
  try {
    if ( in != null) in.close();  
  } catch(Exception e) {
    e.printStackTrace();
  }
  try {
    if (out != null) out.close();
  } catch(Exception e) {
    e.printStackTrace();
  }
}
```

### 2.7 文件拷贝

​        I/O 流通常都是成对出现，即输入流和输出流一起使用。例如文件拷贝需要通过输入流读取源文件中的数据，并通过输出流将数据写入新文件。看一下示例：

```java
FileInputStream in = new FileInputStream("sources/src.jpg");
FileOutputStream out = new FileOutputStream("target/dest.jpg");
int len = 0;
long beginTime = System.currentTimeMillis();
while ( (len = in.read()) != -1) {
  out.write(len);
}
long endTime = System.currentTimeMillis();
System.out.println("花费时间为："+(endTime-beginTime) +"毫秒");
in close();
out.close();
```

说明：

​        上述示例在拷贝过程中，通过while循环将字节逐个进行拷贝。在拷贝文件时，由于计算机性能等各方面原因，会导致拷贝文件所消耗的时间不确定，因此每次运行程序的结果并不一定相同。

### 2.8 字节流缓冲区

​        在文件拷贝过程中，通过以字节形式逐个拷贝，效率非常低。为此，可以设定一个字节数组作为缓冲区，在拷贝文件时，就可以一次性读取多个字节的数据。

#### 2.8.1 拷贝文件示例

```java
...
FileInputStream in = new FileInputStream("sources/src.jpg");
FileOutputStream out = new FileOutputStream("target/dest.jpg");
int len = 0;
byte[] buff = new byte[1024]; // 1k bytes
long beginTime = System.currentTimeMillis();
while ((len = in.read(buff)) != -1) {
  out.write(buff, 0, len);
}
ong endTime = System.currentTimeMillis();
System.out.println("花费时间为："+(endTime-beginTime) +"毫秒");
in.close();
out.close();
```

​        程序中的缓冲区就是一块内存，该内存主要用于存放暂时输入/输出的数据，由于使用缓冲区减少了对文件的操作次数，所以可以提高读写数据的效率。

### 2.9 字节缓冲流

​        除了定义**字节缓冲区**来提高文件拷贝的效率外，I/O 中还提供了两个**字节缓冲流**来提高文件拷贝效率：*BufferedInputStream 和 BufferedOutputStream*。它们的构造方法中，分别接收 *InputStream* 和 *OutputStream* 类型的参数作为对象，在读写数据时提供缓冲功能。

​        缓冲流的示意图：

![5](./images/Stream_Arch/5.png)

#### 2.9.1 代码示例

```java
...
BufferedInputStream bis = new BufferedInputStream( new FileInputStream("sources/src.jpg"));
BufferedOutputStream bos = new BufferedOutputSteram(new FileOutputStream("target/dest.jpg"));
int len = 0;
long beginTime = System.currentTimeMillis();
while ((len = bis.read())!= -1) {
  bos.write(len);
}
long endTime = System.currentTimeMillis();
System.out.println("花费时间为："+(endTime-beginTime) +"毫秒");
bis.close();
bos.close();
```

​        拷贝文件所消耗的时间明显减少了很多，这说明使用字节缓冲流同样可以有效的提高程序的传输效率。

​        这种方式与字节流的缓冲区类似，都对数据进行了缓冲，从而有效的提高了数据的读写效率。

## 3 字符流

​        JDK提供了用于**实现字符操作**的字符流，同字节流一样，字符流也有两个抽象的顶级父类，分别是 *Reader* 和 *Writer*。

​         Reader 和 Writer 的继承关系：

<img src="./images/Stream_Arch/6.png" alt="6" style="zoom:80%;" />

<img src="./images/Stream_Arch/7.png" alt="7" style="zoom:80%;" />

### 3.1 字符流操作文件

​        想从文件中直接读取字符便可以使用字符输入流FileReader，通过此流可以从文件中读取一个或一组字符。

​        **逐个字符读取文件**示例：

```java
FileReader fileReader = new FileReader("reader.txt");
int len = 0;
while ((len = fileReader.read()) != -1) {
    System.out.print((char)len);
}
fileReader.close();
```

​        写入字符就需要使用FileWriter类，该类是Writer的一个子类（**逐个字符写入**文件）：

```java
FileWriter fileWriter = new FileWriter("writer.txt");
fileWriter.write("轻轻的我走了，\r\n");
fileWriter.write("正如我轻轻的来；\r\n");
fileWriter.write("我轻轻的招手，\r\n");
fileWriter.write("作别西天的云彩。\r\n");
fileWriter.close();
```

​        同理：使用字符流向文件追加写入数据，需要调用重载的构造方法：

```java
FileWriter fileWriter = new FileWriter("writer.txt",true)
```

​        使用字符流逐个字符的读写文件也需要频繁的操作文件，效率仍非常低。为此，同字节流操作文件一样，也可以使用提供的字符流缓冲区（类似于字节流缓冲区）和字符缓冲流（类似于字节缓冲流）进行读写操作，来提高执行效率。

​        字符流缓冲区需要定义一个字符数组作为字符缓冲区，通过操作字符缓冲区来提高文件读写效率。

​        字符缓冲流需要使用 *BufferedReader* 和 *BufferedWriter*，其中 *BufferedReader* 用于对字符输入流进行操作，*BufferedWriter* 用于对字符输出流进行操作。**在 *BufferedReader* 中有一个 *readLine()* 方法，用于一次读取一行文本**。

### 3.2 使用字符流缓冲区拷贝文件示例

```java
FileReader fileReader = new FileReader("reader.txt");
FileWriter fileWriter = new FileWriter("writer.txt");
int len  = 0;
char[] buff = new char[1024]; // 1k
while((len = fileReader.read(buff)) != -1) {
  fileWriter.write(buff, 0, len);
}
fileReader.close();
fileWriter.close();
```

### 3.3 使用字符缓冲流拷贝文件

```java
BufferedReader br = new BufferedReader(new FileReader("reader.txt"));
BufferedWriter bw = new BufferedWriter(new FileWriter("writer.txt"));
String str = null;
while((str = br.readLine()) != null) {
  bw.write(str);
  bw.newLine();
}
br.close();
bw.close();
```

### 3.4 转换流

​        在JDK中，提供了两个类用于实现将***字节流转换为字符流***，它们分别是 *InputStreamReader* 和 *OutputStreamWriter*。

​        *InputStreamReader* 是 Reader的子类，它可以将一个字节流转换成字符流，方便直接读取字符；*OutputStreamWriter* 是 Writer 的子类，可以将一个字节流转换成字符流，方便直接写入字符。

#### 3.4.1 转换流操作文件的示意图

![8](./images/Stream_Arch/8.png)

#### 3.4.2 使用转换流拷贝文件示例

```java
FileInputStream in = new FileInputStream("reader.txt");
InputStreamReader isr = new InputStreamReader(in);
BufferedReader br = new BufferedReader(isr);
FileOutputStream out = new FileOutputStream("writer.txt");
OutputStreamWriter osw = new outputStreamWriter(out);
BufferedWriter bw = new BufferedWriter(osw);
String line = null;
while ((line = br.readLine()) != null) {
  bw.write(line);
  bw.newLine();
}
br.close();
bw.close();
```

## 4 File 类

### 4.1 File 类的作用

​        File 类用于封装一个路径，这个路径可以是从系统盘符开始的绝对路径，也可以是相对于当前目录而言的相对路径。封装的路径可以指向一个文件，也可以指向一个目录。在 File 类中提供了针对这些文件或目录的一些常规操作。

### 4.2 File 类常用的构造方法

| 方法声明                          | 功能描述                                                     |
| --------------------------------- | ------------------------------------------------------------ |
| File(String pathName)             | 通过指定一个字符串类型的文件路径来创建一个新的 File 对象     |
| File(String parent, String child) | 根据指定的一个**字符串类型**的父路径和一个**字符串类型**的子路径（包括文件名称）创建一个 File 对象 |
| File(File parent, String child)   | 根据指定的**File 类型**的父路径和**字符串类型**的子路径（包括文件名称）创建一个 File 对象 |

### 4.3 File 类的常用方法

| 方法声明                 | 功能描述                                                     |
| ------------------------ | ------------------------------------------------------------ |
| boolean exists()         | 判断File对象对应的文件或目录是否存在，若存在则返回ture，否则返回false |
| boolean delete()         | 删除File对象对应的文件或目录，若成功删除则返回true，否则返回false |
| boolean createNewFile()  | 当File对象对应的文件不存在时，该方法将新建一个此File对象所指定的新文件，若创建成功则返回true，否则返回false |
| String getName()         | 返回File对象表示的文件或文件夹的名称                         |
| String getPath()         | 返回File对象对应的路径                                       |
| String getAbsolutePath() | 返回File对象对应的绝对路径（在Unix/Linux等系统上，如果路径是以正斜线/开始，则这个路径是绝对路径；在Windows等系统上，如果路径是从盘符开始，则这个路径是绝对路径） |
| String getParent()       | 返回File对象对应目录的父目录（即返回的目录不包含最后一级子目录） |
| boolean canRead()        | 判断File对象对应的文件或目录是否可读，若可读则返回true，反之返回false |
| boolean canWrite()       | 判断File对象对应的文件或目录是否可写，若可写则返回true，反之返回false |
| boolean isFile()         | 判断File对象对应的是否是文件（不是目录），若是文件则返回true，反之返回false |



## 5 RandomAccessFile

## 6 对象序列化

## 7 NIO

## 8 NIO 2

