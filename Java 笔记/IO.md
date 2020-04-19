处理数据，有字节流，为了人们观看方便，有了字符流

从流向分输入流，输出流

如果操作的节点，比如文件，字节数组：节点流

节点之上，装饰流，处理流（节点流性能提升）



重点掌握文件拷贝

创建源

选择流

操作流

释放资源

如果是字节，借助byte容器，循环读取。用read来读，写出则用write和flush强制刷行，最后close

字符流，直接用bufferedReader，直接readLine。
write newLine append

data流： 操作字符串 writeUtf readUtf

操作对象 Object。（inputSteam和OutputSteam）可以序列化和反序列化。
反序列化由于有字符转换，用instanceOf

一般IO就是文件上传和下载