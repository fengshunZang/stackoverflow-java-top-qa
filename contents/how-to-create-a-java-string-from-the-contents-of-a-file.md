## 怎样将文件的内容生成java string
### 问题：
我现在使用如下的语句有一段时间了。它看起来是最广泛使用的，最起码在我浏览过的网站来说。
有谁有一个更好/不同的方法来将文件中的内容转换为java中的string吗？

````java
private String readFile(String file) throws IOException {
    BufferedReader reader = new BufferedReader(new FileReader(file));
    String         line = null;
    StringBuilder  stringBuilder = new StringBuilder();
    String         ls = System.getProperty("line.separator");

    try {
        while((line = reader.readLine())!= null) {
            stringBuilder.append(line);
            stringBuilder.append(ls);
        }

        return stringBuilder.toString();
    } finally {
        reader.close();
    }
}

````

### 最佳答案：
#### 从文件中读取全部内容
下面是java7中利用实用的Method实现的一种紧凑、健壮的写法。
````java
static String readFile(String path, Charset encoding) 
  throws IOException 
{
  byte[] encoded = Files.readAllBytes(Paths.get(path));
  return new String(encoded, encoding);
}
````
#### 从文件中读取文件行
java7增加了一种更方便的从文件中[`读取文件行的方法`](http://docs.oracle.com/javase/7/docs/api/java/nio/file/Files.html#readAllLines%28java.nio.file.Path,%20java.nio.charset.Charset%29)，用 List<String>表示。
这种方式是“有损耗的”，因为行分隔符在每行最后被剥夺了。
````java
List<String> lines = Files.readAllLines(Paths.get(path), encoding);
````
#### 内存分析
第一种保留行分隔符的方法能在短时间内需求文件规模几倍的内存，因为短时间内源文件内容（字节数组）和编码的字符（每个都是16位的即使在文件中按8位编译）都曾经在内存中存留。处理那些相对于有限的内存很小的文件是使用这种方法最安全的方式。
第二种读取文件行的方法通常来说更节省内存，因为输入字节的缓冲不必包含整个文件。但是，对于相比内存很大的文件来说，这种方法也同样不合适。
对于读取大型文件来说，你需要设计一个不同的系统。这个系统从流中读取一大块文本，处理，然后移动到下一块，重新使用同样的大小固定的内存模块。这里，“大型”取决于电脑的规格。现在，定义“大型”的门槛可能从许多G的RAM开始。
#### 字符编码
原来的解答中遗漏了关于字符编码的说明。只有一些特殊情况下，平台的默认设定是你需要的，但是这种情况很少，你应该能够根据自己的需求调整。
StanderdCharsets类定义了一些编码中需要的常量。
````java
String content = readFile("test.txt", StandardCharsets.UTF_8);
````
平台的默认设定中也可从charset类中得到。
````java
String content = readFile("test.txt", Charset.defaultCharset());
````

StackOverFlow 链接：http://stackoverflow.com/questions/326390/how-to-create-a-java-string-from-the-contents-of-a-file
