## 序列化与反序列化
对象序列化的目的是将对象保存到磁盘中，或允许在网络中直接传输对象。  
对象序列化机制允许把内存中的Java对象转换成平台无关的二进制流，从而允许把这种二进制流持久的保存在磁盘上，通过网络将这种二进制流传输到另一个网络节点，
这样可以使得对象可以脱离程序的运行而独立的存在（在Java中，我们创建出的对象是运行在JVM的堆内存中，只有JVM处于运行状态时候，这些对象才可能存在。
一旦JVM停止运行，这些对象的状态也就随之丢失）。
对象的反序列化是指将这种二进制流恢复成Java对象。
## 相关接口和类
Java提供了一套接口来支持序列化和反序列化
```Java
java.io.Serializable
java.io.Externalizable
ObjectOutput
ObjectInput
ObjectOutputStream
ObjectInputStream
```
### Serializable
类通过实现java.io.Serializable接口来实现序列化功能。该接口是一个标记接口，实现该接口无需实现任何方法，它只是表明该类的实例是可序列化的。  
一旦某个类实现了Serializable接口，可以通过如下两个步骤来序列化该对象:  
1.创建一个ObjectOutputStream，这个输出流是一个处理流，所以必须建立在其他节点流的基础之上。
```Java
ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("tmpFile"));
```
2.调用ObjectOutputStream对象的writeObject()方法输出可序列化对象。
```Java
oos.writeObject(user);
```
希望从一个二进制流中恢复Java对象，需要反序列化，反序列化步骤如下：  
1.创建一个ObjectInputStream输入流，这个输入流是一个处理流，所以必须建立在其他节点流的基础之上。
```Java
ObjectInputStream ois = new ObjectInputStream(new FileInputStream("tmpFile"));
```
2.调用ObjectInputStream对象的readObject()方法读取流中的对象，该方法返回一个Object类型的Java对象。可以使用强制类型转化将其转化为真实类型。
```Java
User readUser = (User) ois.readObject();
```
## 参考资料
《疯狂Java讲义第四版》  
[Java对象的序列化与反序列化](https://www.hollischuang.com/archives/1150)  
[深入分析Java的序列化与反序列化](https://www.hollischuang.com/archives/1140)  
[Java 序列化的高级认识](https://www.ibm.com/developerworks/cn/java/j-lo-serial/)
