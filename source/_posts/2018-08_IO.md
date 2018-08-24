---
title: 理解IO流
date: 2018-08-16 19:55:19
tags: [Java进阶]
categories: Java进阶
---
流是一种抽象概念，它代表了数据的无结构化传递。用来进行输入输出操作的流就称为IO流。
# IO流结构
## 流的分类方式
按流向分：从文件/网络/内存等(数据源)到程序是输入流
从程序到文件/网络/内存等(数据源)是输出流
按数据处理单位分
字节流：以字节为单位传输数据的流，以Stream结尾的都是字节流。
字符流：以字符为单位传输数据的流，以Reader结尾的都是输入字符流，以Writer结尾的都是输出字符流。
按功能（层次）分
节点流：用于直接操作目标设备的流
处理流(也叫过滤流)：是对一个已存在的流的连接和封装，通过对数据的处理为程序提供更为强大、灵活的读写功能。
## IO流的结构
如下图所示：
![IO流的结构](/images/2018-8-1/java_io_show.jpg)
注意：
所有的字节输入流类都是InputStream的子类；所有的字节符输入流类都是Reader的子类；所有的字节节输出流类都是OutputStream的子类；所有的字节符输出流类都是Writer的子类，且他们都为抽象类。
## IO流四大抽象类
InputStream的基本方法：

     public abstract int read() throws IOException {}//从输入流中读取数据的下一个字节, 返回读到的字节值.若遇到流的末尾,返回-1
     public int read(byte[] b) throws IOException {}//从输入流中读取 b.length 个字节的数据并存储到缓冲区数组b中.返回的是实际读到的字节总数
     public int read(byte[] b, int off, int len) throws IOException {}//读取 len 个字节的数据,并从数组b的off位置开始写入到这个数组中
     public void close() throws IOException {}//关闭此输入流并释放与此流关联的所有系统资源
     public int available() throws IOException {}//返回此输入流下一个方法调用可以不受阻塞地从此输入流读取（或跳过）的估计字节数
     public long skip(long n) throws IOException {}//跳过和丢弃此输入流中数据的 n 个字节，返回实现路过的字节数。
OutputStream的基本方法：

    public abstract void write(int b) throws IOException {}//将指定的字节写入此输出流。
    public void write(byte[] b) throws IOException {}// 将 b.length 个字节从指定的 byte 数组写入此输出流。
    public void write(byte[] b, int off, int len) throws IOException {}//将指定 byte 数组中从偏移量 off 开始的 len 个字节写入此输出流。
    public void flush() throws IOException {}//刷新此输出流并强制写出所有缓冲的输出字节。
    pulbic void close() throws IOException {}//关闭此输出流并释放与此流有关的所有系统资源。
Reader的基本方法：

    public int read() throws IOException {}//读取单个字符，返回作为整数读取的字符，如果已到达流的末尾返回-1
    public int read(char[] cbuf) throws IOException {}//将字符读入数组，返回读取的字符数
    public abstract int read(char[] cbuf, int off, int len) throws IOException {}//读取 len 个字符的数据，并从数组cbuf的off位置开始写入到这个数组中
    public abstract void close() throws IOException {}//关闭该流并释放与之关联的所有资源
    public long skip(long n) throws IOException {}//跳过n个字符。
    public int available()  //还可以有多少能读到的字节数
Writer的基本方法：

    public void write(int c) throws IOException {} //写入单个字符
    public void write(char[] cbuf) throws IOException {} //写入字符数组
    public abstract void write(char[] cbuf, int off, int len) throws IOException {} //写入字符数组的某一部分
    public void write(String str) throws IOException {} //写入字符串
    public void write(String str, int off, int len) throws IOException {}//写字符串的某一部分
    public abstract void close() throws IOException {}  //关闭此流，但要先刷新它
    public abstract void flush() throws IOException {}  //刷新该流的缓冲，将缓冲的数据全写到目的地

# IO流的具体使用
## FileInputStream  和  FileOutputStream

	public static void fileInputStreamTest() {
		File f1 = new File("D:\\in.txt");
		File f2 = new File("D:\\out.txt");
		try {
			FileInputStream fi = new FileInputStream(f1); 
			FileOutputStream fo = new FileOutputStream(f2);
			byte[] buf = new byte[521];
            int len = 0;
            while((len = fi.read(buf)) != -1){
                fo.write(buf, 0, len);
            }
            fo.flush();
            fo.close();
            fi.close();
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
## PipedOutputStream和PipedintputStream
Java里的管道输入流PipedInputStream与管道输出流PipedOutputStream实现了类似管道的功能，用于不同线程之间的相互通信。
Java的管道输入与输出实际上使用的是一个循环缓冲数组来实现，这个数组默认大小为1024字节。输入流PipedInputStream从这个循环缓冲数组中读数据，输出流PipedOutputStream往这个循环缓冲数组中写入数据。当这个缓冲数组已满的时候，输出流PipedOutputStream所在的线程将阻塞；当这个缓冲数组首次为空的时候，输入流PipedInputStream所在的线程将阻塞。Java在它的jdk文档中提到不要在一个线程中同时使用PipeInpuStream和PipeOutputStream，这会造成死锁。

    public class WriteThread implements Runnable{
        private PipedOutputStream pout;  

        WriteThread(PipedOutputStream pout){  
          this.pout=  pout;  
        }  

        @Override
        public void run() {
            try {  
                System.out.println("W:开始将数据写入:但等个5秒让我们观察...");  
                Thread.sleep(5000);  //释放cpu执行权5秒  
                pout.write("writePiped 数据...".getBytes());  //管道输出流  
                pout.close();  
              } catch(Exception e) {  
                throw new RuntimeException("W:WriteThread写入失败...");  
              }  
        }
    }

    public class ReadThread implements Runnable{
        private PipedInputStream pin;  

        ReadThread(PipedInputStream pin) {  
          this.pin=pin;  
        }  

        @Override
        public void run() {  //由于必须要覆盖run方法,所以这里不能抛,只能try  
          try {  
                System.out.println("R:读取前没有数据,阻塞中...等待数据传过来再输出到控制台...");  
                byte[] buf = new byte[1024];  
                int len = pin.read(buf);  //read阻塞  
                System.out.println("R:读取数据成功,阻塞解除...");  
                String s= new String(buf,0,len);  
                System.out.println(s);  //将读取的数据流用字符串以字符串打印出来  
                pin.close();       
          }  catch(Exception e)  {  
                throw new RuntimeException("R:管道读取流失败!");  
          }     
        }  
    }

    public class Test {
        public static void main(String[] args) throws IOException {
              PipedInputStream pin = new PipedInputStream();  
              PipedOutputStream pout = new PipedOutputStream();  
              pin.connect(pout);  //输入流与输出流连接  

              ReadThread readTh   = new ReadThread(pin);  
              WriteThread writeTh = new WriteThread(pout);  
              new Thread(readTh).start();  
              new Thread(writeTh).start();  
        }
    }
## BufferedInputStream和BufferedOutputStream
FileInputStream和FileOutputStream 在使用时，我们介绍了可以用byte数组作为数据读入的缓存区，以读文件为列，读取硬盘的速度远远低于读取内存的数据，为了减少对硬盘的读取，通常从文件中一次读取一定长度的数据，把数据存入缓存中，在写入的时候也是一次写入一定长度的数据，这样可以增加文件的读取效率。我们在使用FileInputStream的时候是用byte数组来做了缓存，而BufferedInputStream  and BufferedOutputStream已经为我们增加了这个缓存功能。

	public static void fileBufferTest() {
		File f1 = new File("D:\\in.txt");
		File f2 = new File("D:\\out.txt");
		try {
            BufferedInputStream bufferedInputStream = new BufferedInputStream(new FileInputStream(f1));   
            BufferedOutputStream bufferedOutputStream = new BufferedOutputStream(new FileOutputStream(f2));   
            byte[] data = new byte[1];
            while(bufferedInputStream.read(data)!=-1) {   
                bufferedOutputStream.write(data);   
            }   
            //将缓冲区中的数据全部写出   
            bufferedOutputStream.flush();   
            bufferedInputStream.close();   
            bufferedOutputStream.close();   
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
## 读写对象ObjectInputStream和ObjectOutputStream

    public class ObjectStream {
        public static void main(String[] args) {
            ObjectStream.objectStreamTest();
        }

        public static void objectStreamTest() {
            Demo newObject;
            try {
                ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(new File("D:/123.obj")));
                oos.writeObject(new Demo());
                oos.flush();
                ObjectInputStream ois = new ObjectInputStream(new FileInputStream(new File("D:/123.obj")));
                newObject = (Demo)ois.readObject();
                System.out.println(newObject.num);
                ois.close();
                oos.close();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }

    class Demo implements Serializable{
        private static final long serialVersionUID = 1L;
        int num = 30;
    }
## SequenceInputStream
 合并流，将与之相连接的流集组合成一个输入流并从第一个输入流开始读取， 直到到达文件末尾，接着从第二个输入流读取，依次类推，直到到达包含的最后一个输入流的文件末尾为止。 合并流的作用是将多个源合并合一个源。可接收枚举类所封闭的多个字节流对象。

    public class Test {
          public static void main(String[] args) {  
                 doSequence();  
          }  

          private static void doSequence() {  
             SequenceInputStream sis = null;  // 创建一个合并流的对象  
             BufferedOutputStream bos = null;   // 创建输出流。  
             try {  
                // 构建流集合 
                Vector<InputStream> vector = new Vector<InputStream>();  
                vector.addElement(new FileInputStream("D:/in.txt"));  
                vector.addElement(new FileInputStream("D:/out.txt"));  
                Enumeration<InputStream> e = vector.elements();  
                sis = new SequenceInputStream(e);  
                bos = new BufferedOutputStream(new FileOutputStream("/Users/zhengchao/cctv/File_OUT.txt"));  
                // 读写数据
                byte[] buf = new byte[1024];  
                int len = 0;  
                while ((len = sis.read(buf)) != -1) {  
                   bos.write(buf, 0, len);  
                   bos.flush();  
                }  
             } catch (Exception e1) {  
                e1.printStackTrace();  
             } finally {  
                try {  
                   if (sis != null)  
                      sis.close();  
                } catch (IOException e) {  
                   e.printStackTrace();  
                }  
                try {  
                   if (bos != null)  
                      bos.close();  
                } catch (IOException e) {  
                   e.printStackTrace();  
                }  
             }  
          }  
    }
## FileReader与FileWriter

    public static void fileReaderTest() {
		File f1 = new File("D:\\in.txt");
		File f2 = new File("D:\\out.txt");
		try {
			FileReader fr = new FileReader(f1);
			FileWriter fw = new FileWriter(f2);
			char[] ch = new char[512];
            int len = 0;
            while((len = fr.read(ch)) != -1){
            	fw.write(ch, 0, ch.length);
            }
            fw.flush();
            fw.close();
            fr.close();
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
## BufferedReader和BufferedWriter

	public static void fileBufferReaderWriteTest() {
		File f1 = new File("D:\\in.txt");
		File f2 = new File("D:\\out.txt");
		try {
	        BufferedReader br=new BufferedReader(new FileReader(f1));
	        BufferedWriter bw=new BufferedWriter(new FileWriter(f2));

	        String s=br.readLine();
            while(null!=s) {
                bw.write(s);
                //由于BufferedReader的readLine()是不读入换行符的，所以写入换行时须用newLine()方法
                bw.newLine();
                s=br.readLine();
            }
            br.close();
            bw.close();
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
注：在使用过程中应注意将编码格式设置为UTF-8，否则文件写入会乱码。

以上只是IO流的部分类，用于对IO流相关知识进行查阅，后面再实际的工作中也会根据实际需求不断添加。
参考自：
https://blog.csdn.net/zhengchao1991/article/details/53033137
https://blog.csdn.net/SilenceOO/article/details/50995062
https://blog.csdn.net/Yue_Chen/article/details/72772445