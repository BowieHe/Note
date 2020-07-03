根据处理数据类型不同，分为字符流和字节流

根据数据流向不同，分为输入流和输出流

根据流的角色不同，分为节点流和处理流：

- 节点流：直接链接数据源的流，乐意直接向数据源读写数据
- 处理流必须通过构造方法接受一个节点流，对节点流使用装饰模式增加功能。处理流必须依赖一个节点流，只有节点流最终可以将数据流输入到IO设备中

**字符流和字节流**
字符流的由来： 因为数据编码的不同，而有了对字符进行高效操作的流对象。本质其实就是基于字节流读取时，去查了指定的码表。字节流和字符流的区别：

1. 读写单位不同：字节流以字节（8bit）为单位，字符流以字符为单位，根据码表映射字符，一次可能读多个字节。
2. 处理对象不同：字节流能处理所有类型的数据（如图片、avi等），而字符流只能处理字符类型的数据。
3. 字节流在操作的时候本身是不会用到缓冲区的，是文件本身的直接操作的；而字符流在操作的时候下后是会用到缓冲区的，是通过缓冲区来操作文件，我们将在下面验证这一点。

**结论**：优先选用字节流。首先因为硬盘上的所有文件都是以字节的形式进行传输或者保存的，包括图片等内容。但是字符只是在内存中才会形成的，所以在开发中，字节流使用广泛。

| s      | 输入流                      | 输出流                       |
| ------ | --------------------------- | ---------------------------- |
| 字节流 | 字节输入流<br />InputStream | 字节输入流<br />OutputStream |
| 字符流 | 字符输入流<br />Reader      | 字符输出流<br />Writer       |

各自的子类，都以父类为自己的后缀，比如文件的字节输入流为FileOutputStream，文件的字符输入流：FileReader

读进来：使用的是输入流，读是输入流中的一个方法(read)
写出去：使用的是输出流，写是输出流中的一个方法(write)

## 节点流

节点流直接连接数据源的流

节点流可以从一个特定的数据源读写数据（文件，内存。。。）

常见的节点流

| 类型          | 字符流                               | 字节流                                          |
| ------------- | ------------------------------------ | ----------------------------------------------- |
| File          | FileReader<br />FilerWriter          | FileInputStream<br />FileOutputStream           |
| Memory Array  | CharArrayReader<br />CharArrayWriter | ByteArrayInputStream<br />ByteArrayOutputStream |
| Memory String | StringReader<br />StringWriter       | -                                               |
| Pipe          | PipeReader<br />PipeWriter           | PipeInputStream<br />PipeOutputStream           |

- File 文件流。对文件进行读、写操作 ：FileReader、FileWriter、FileInputStream、FileOutputStream。
- 从/向内存数组读写数据: CharArrayReader与 CharArrayWriter、ByteArrayInputStream与ByteArrayOutputStream。
- 从/向内存字符串读写数据 StringReader、StringWriter、StringBufferInputStream。
- Pipe管道流。 实现管道的输入和输出（进程间通信）: PipedReader与PipedWriter、PipedInputStream与PipedOutputStream。

## 处理流

处理流是连接已存放的流智商，通过对数据的处理为程序提供更加强大的读写功能

常见的处理流

| 处理类型                               | 字符流                                    | 字节流                                        |
| -------------------------------------- | ----------------------------------------- | --------------------------------------------- |
| Buffering                              | BufferedReader<br />BufferedWriter        | BufferedInputStream<br />BufferedOutputStream |
| Filtering                              | FilterReader<br />FilterWriter            | FilerInputStream<br />FilterOutputStream      |
| Converting between bytes and character | InputStreamReader<br />OutputStreamWriter |                                               |
| Object Serialization                   | -                                         | ObjectInputStream<br />ObjectOutputStream     |
| Data conversion                        | -                                         | DataInputStream<br />DataOutputStream         |
| counting                               | LineNumberReader                          | LineNumberInputStream                         |
| peeking ahead                          | PushbackReader                            | PushbackInputStream                           |
| printing                               | PrintWriter                               | PrintStream                                   |

- Buffering缓冲流：在读入或写出时，对数据进行缓存，以减少I/O的次数：BufferedReader与BufferedWriter、BufferedInputStream与BufferedOutputStream。

- Filtering 滤流：在数据进行读或写时进行过滤：FilterReader与FilterWriter、FilterInputStream与FilterOutputStream。

- Converting between Bytes and Characters 转换流：按照一定的编码/解码标准将字节流转换为字符流，或进行反向转换（Stream到Reader）：InputStreamReader、OutputStreamWriter。

- Object Serialization 对象流 ：ObjectInputStream、ObjectOutputStream。

- DataConversion数据流： 按基本数据类型读、写（处理的数据是Java的基本类型（如布尔型，字节，整数和浮点数））：DataInputStream、DataOutputStream 。

- Counting计数流： 在读入数据时对行记数 ：LineNumberReader、LineNumberInputStream。

- Peeking Ahead预读流： 通过缓存机制，进行预读 ：PushbackReader、PushbackInputStream。

- Printing打印流： 包含方便的打印方法 ：PrintWriter、PrintStream。

  