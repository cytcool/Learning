# Java字节流与字符流

- File类不支持文件内容处理，如果要处理文件内容那么必须要通过流的操作模式来完成
- 流：输入流和输出流

## 字符流与字节流

- 本质区别：字节流是原生的操作，字符流是经过处理后的操作
- 网络数据传输和磁盘数据保存所支持的数据类型只有字节
- 所有磁盘中的数据必须先读取到内存后才可以读取，内存里会帮助我们将字节变为字符
- 字符更适合处理中文

### 在java.io包中流分为两种

- 字节流：InputStream、OutputStream
- 字符流：Reader、Writer

### 流的实现操作

- 根据文件的路径创建File类对象
- 根据字节流或字符流的子类实例化父类对象
- 进行数据的读取、写入操作
- ==关闭流==

### 字节输出流 OutputStream

- 要想通过程序进行内容的输出，则可以使用java.io.OutputStream这个类

```
//OutputStream定义
public abstract calss OutputStream
extends Object
implements Closeable,Flushable
```

```
//Closeable
public void close() throws IOException;
```

```
//Flushable
public void flush() throws IOException
```
- 在OutputStream类里面实际上还有其它方法

```
//将给定的字节数组内容全部输出：
public void write(byte[] b) throws IOException;
//将给定的字节数组内容部分输出：
public void write(byte[] b,int off,int len) throws IOException;
//输出单个字节
public abstract void write(int b) throws IOException;
```
- OutputStream是一个抽象类，所以抽象类的规则，要想为父类进行实例化，需要使用子类。
- 因为父类的方法已经定义好，只需要关注子类的构造方法
- 如果要进行文件的操作，可以使用FileOutputStream类

```
//接收File类（覆盖）
public FileOutputStream(File file) thorws FileNotFoundException
//追加方法
public FileOutputStream(File file,bollean append) throws FileNotFoundException
```

- 实例：实现文件的内容输出

```
public class TestDemo{
    public static void main(String[] args) throws Exception{
    //1、通过File类定义文件路径
    File file = new File("d:" + File.separator + "hello.txt");
    //必须保证父目录存在
    if(!file.getParentFile().exists)){
    //创建目录
    file.getParentFile().mkdirs
    }
    //2、OutputStream是一个抽象类，所以需要通过子类进行实例化
    OutputStream output = new FileOutputStream(file);
    //3、进行文件的输出处理操作
    String msg = "www.imooc.com";
    //将内容变为字节数组
    output.write(msg.getBytes());
    //4、关闭输出
    output.close()
}
```
- 在进行文件输出的时候所有的文件会自动帮助用户创建，不再需要编写creatNewFile()方法手工创建
- 上述代码重复执行时，不会出现追加的情况，因为默认的状态就是覆盖的重复输出，如果需要追加内容，需要更换FileOutputStream类的构造方法
**OutputStream output = new FileOutputStream(file,==true==);**

### 自动关闭处理(AutoCloseable)

- JDK1.7之后
- 加入的主要目的是希望可以进行自动的关闭处理
- 使用AutoCloseable需要结合try-catch结构


```
try(OutputStream output = new FileOutputStream(file)){
    String msg = "www.imooc.com";
    output.write(msg.getBytes());
}catch(Exception e){
}
```

### 字节输入流InputStream

- 通过程序读取文件内容，需要使用到InputStream

```
//InputStream类定义
public abstract class InputStream
extends Object
implements Closeable
```

```
//读取数据到字节数组之中
//1、返回数据的读取个数，如果开辟的字节数组的大小大于了读取数据的大小，此时返回的是读取数据的个数
//2、如果现在要读取的数据大于数组的长度，则返回数组的长度
//3、如果没有数据了还要继续读，则返回-1;
public int read(byte[] b) throws IOException;
//读取部分数据到字节数组之中
//如果读取满了，则返回的就是长度
//如果没有读取满，则返回的是读取数据的个数
public int read(byte[] b,int off,int len) throws IOException;
//读取单个字节
public abstract int read();
```
- - IutputStream是一个抽象类，所以抽象类的规则，要想为父类进行实例化，需要使用子类,使用**FileInputStream**类

- 实例：实现文件的内容读取

```
public class TestDemo{
    public static void main(String[] args) throws Exception{
    //1、通过File类定义文件路径
    File file = new File("d:" + File.separator + "hello.txt");
    //必须保证程序存在才可以进行读取处理
    if(file.exists()){
        InputStream input = new FileInputStream(file);
        //每次可以读取的最大数量
        byte[] data=new byte[1024];
        //此时的数据读取到了数组之中
        int len = input.read(data);
        String.out.println("读取的内容为"+new String(data,0,len)); 
        input.close();
   }
}
```

### 字符输出流：Writer

- 定义：

```
public abstract class Writer
extends Object
implements Appendable,Closable,Flushable
```
- Writer类中的方法()

```
//输出内容
public void write(String str) throws IOException
public void write(String str,int off,int len) throws IOException
```

- 如果要操作文件，需要使用FileWriter子类
- 实例：通过Writer实现输出

```
public class TestDemo{
    public static void main(String[] args) throws Exception{
    //1、通过File类定义文件路径
    File file = new File("d:" + File.separator + "hello.txt");
    //必须保证父目录存在
    if(!file.getParentFile().exists)){
    //创建目录
    file.getParentFile().mkdirs
    }
   String msg = "世界和平！";
   Writer out = new FileWriter(file);
   out.writer(msg);
   out.close();
}
```

### 字符输入流：Reader

- Reader是一个抽象类
- 文件读取使用FileReader子类
- 类定义：

```
public abstract class Reader
extends Object
implements Readable,Closeable
```
- Reader类中没有方法可以直接读取数据为字符串String
- 实例：通过Reader读取数据

```
public class TestDemo{
    public static void main(String[] args) throws Exception{
    //1、通过File类定义文件路径
    File file = new File("d:" + File.separator + "hello.txt");
    //必须保证程序存在才可以进行读取处理
    if(file.exists()){
        Reader in = new FileReader(file);
        char data[] = new char[1024];
        int len = in.read(data);
        System.out.println(new String(data,0,len));
        in.close;
   }
}
```

### 字节流与字符流的区别

- 正常情况优先考虑字节流，只有在处理中文字符时使用字符流

