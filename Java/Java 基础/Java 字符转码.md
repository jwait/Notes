# Java 字符转码

对字符进行转码的流程为：源字符串->解码->编码->目标字符串

Java 中有三种方法对字符进行转码



## 使用 Java.lang.String

### 定义的用于解码的构造器

```
String(byte[] bytes, int offset, int length, String charsetName)   
String(byte[] bytes, String charsetName)  
```



### 定义的用于编码的方法

```
byte[] getBytes(String charsetName)  //使用指定字符集进行编码  
byte[] getBytes() //使用系统默认字符集进行编码  
```



### 示例：

```
byte[] src
byte[] dst
String srcStr = new String(src, "gbk")// 使用gbk解码
byte[] dst = srcStr.getBytes("utf8")// 编码为utf8
```



## 使用Java.io.InputStreamReader/OutputStreamWriter

### InputStreamReader构造器指定编码

```java
public InputStreamReader(InputStream in,
                         String charsetName)
                  throws UnsupportedEncodingException
Creates an InputStreamReader that uses the named charset.
Parameters: 
in - An InputStream 
charsetName - The name of a supported charset 

public InputStreamReader(InputStream in,
                         Charset cs)
Creates an InputStreamReader that uses the given charset.
Parameters: 
in - An InputStream 
cs - A charset 

public InputStreamReader(InputStream in,
                         CharsetDecoder dec)
Creates an InputStreamReader that uses the given charset decoder.
Parameters: 
in - An InputStream 
dec - A charset decoder 


```



### OutputStreamWriter构造器指定编码

```java
public OutputStreamWriter(OutputStream out,
                          String charsetName)
                   throws UnsupportedEncodingException
Creates an OutputStreamWriter that uses the named charset.
Parameters: 
out - An OutputStream 
charsetName - The name of a supported charset 

public OutputStreamWriter(OutputStream out,
                          Charset cs)
Creates an OutputStreamWriter that uses the given charset.
Parameters: 
out - An OutputStream 
cs - A charset 

public OutputStreamWriter(OutputStream out,
                          CharsetEncoder enc)
Creates an OutputStreamWriter that uses the given charset encoder.
Parameters: 
out - An OutputStream 
enc - A charset encoder 

```



### 示例：

```java
is = new FileInputStream("C:/项目进度跟踪.txt");//文件读取  
isr = new InputStreamReader(is, "gbk");//解码  
os = new FileOutputStream("C:/项目进度跟踪_utf-8.txt");//文件输出  
osw = new OutputStreamWriter(os, "utf-8");//开始编码  
```



## 使用 Java.nio.Charset

#### 示例：

```java
public void convertionFile_nio(String src, String dest) throws IOException {  
        fis = new FileInputStream(src);  
        in = fis.getChannel();  
        fos = new FileOutputStream(dest);  
        out = fos.getChannel();  
        inSet = Charset.forName("gbk");  
        outSet = Charset.forName("utf-8");  
        de = inSet.newDecoder();  
        en = outSet.newEncoder();  
        while (fis.available() > 0) {  
            b.clear();// 清除标记  
            in.read(b); // 将文件内容读入到缓冲区内:将标记位置从0-b.capacity(),  
                        // 读取完毕标记在0-b.capacity()之间  
            b.flip();// 调节标记,下次读取从该位置读起  
            convertion = de.decode(b);// 开始编码  
  
            temp.clear();// 清除标记  
            temp = en.encode(convertion);  
            b.flip(); // 将标记移到缓冲区的开始,并保存其中所有的数据:将标记移到开始0  
            out.write(temp); // 将缓冲区内的内容写入文件中:从标记处开始取出数据  
        }  
    }  
```

