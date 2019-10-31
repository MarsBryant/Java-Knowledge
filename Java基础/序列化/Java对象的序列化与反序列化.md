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
```
### Serializable
类通过实现java.io.Serializable接口来实现序列化功能。该接口是一个标记接口，实现该接口无需实现任何方法，它只是表明该类的实例是可序列化的。  
一旦某个类实现了Serializable接口，可以通过如下两个步骤来序列化该对象:  
1.创建一个ObjectOutputStream，这个输出流是一个处理流，所以必须建立在其他节点流的基础之上。
```Java
ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("user.txt"));
```
2.调用ObjectOutputStream对象的writeObject()方法输出可序列化对象。
```Java
oos.writeObject(user);
```
希望从一个二进制流中恢复Java对象，需要反序列化，反序列化步骤如下：  
1.创建一个ObjectInputStream输入流，这个输入流是一个处理流，所以必须建立在其他节点流的基础之上。
```Java
ObjectInputStream ois = new ObjectInputStream(new FileInputStream("user.txt"));
```
2.调用ObjectInputStream对象的readObject()方法读取流中的对象，该方法返回一个Object类型的Java对象。可以使用强制类型转化将其转化为真实类型。
```Java
User readUser = (User) ois.readObject();
```
程序如下所示，定义一个User类，实现了Serializable接口
```Java
import java.io.Serializable;

public class User implements Serializable {

    private String name;
    private static int age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        User.age = age;
    }

    @Override
    public String toString() {
        return "User{"
                + "name='" + name + '\''
                + ", age=" + age
                + '}';
    }
}
```
通过下面的代码进行序列化和反序列化
```Java
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;

public class SerializableDemo {

    public static void main(String[] args) {
        User user = new User("mars", 20);
        System.out.println(user); // output: User{name='mars', age=20}
        ObjectOutputStream oos = null;
        try {
            oos = new ObjectOutputStream(new FileOutputStream("user.txt"));
            oos.writeObject(user);
            oos.close();
        } catch (IOException e) {
            e.printStackTrace();
        }

        ObjectInputStream ois = null;
        try {
            ois = new ObjectInputStream(new FileInputStream("user.txt"));
            User readUser = (User) ois.readObject();
            System.out.println(readUser); // output: User{name='mars', age=20}
            ois.close();
        } catch (IOException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}
```
### Externalizable
先修改上述代码，来了解下Serializable和Externalizable的区别。  
```Java
import java.io.Externalizable;
import java.io.IOException;
import java.io.ObjectInput;
import java.io.ObjectOutput;

public class User1 implements Externalizable {

    private String name;
    private int age;

    public User1(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public User1() {
        System.out.println("no param construcror.");
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public void writeExternal(ObjectOutput out) throws IOException {

    }

    @Override
    public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException {

    }

    @Override
    public String toString() {
        return "User{"
                + "name='" + name + '\''
                + ", age=" + age
                + '}';
    }
}
```

```Java
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;

public class ExternalizableDemo {

    public static void main(String[] args) {
        User1 user = new User1("mars", 20);
        System.out.println(user); // output: User{name='mars', age=20}
        ObjectOutputStream oos = null;
        try {
            oos = new ObjectOutputStream(new FileOutputStream("user.txt"));
            oos.writeObject(user);
            oos.close();
        } catch (IOException e) {
            e.printStackTrace();
        }

        ObjectInputStream ois = null;
        try {
            ois = new ObjectInputStream(new FileInputStream("user.txt"));
            User1 readUser = (User1) ois.readObject();
            System.out.println(readUser); // User{name='null', age=0}
            ois.close();
        } catch (IOException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}
//output: 
//User{name='mars', age=20}
//no param construcror.
//User{name='null', age=0}
```
通过如上结果发现，对User1进行序列化和反序列化后得到对象的属性都为默认值，对象并没有被持久保留下来。因为Externalizable接口强制自定义序列化，使用该接口来序列化和反序列化对象时，需要开发人员重写writeExternal()与readExternal()方法。  
修改代码，重写writeExternal()与readExternal()方法。
```Java
import java.io.Externalizable;
import java.io.IOException;
import java.io.ObjectInput;
import java.io.ObjectOutput;

public class User1 implements Externalizable {

    private String name;
    private int age;

    public User1(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public User1() {
        System.out.println("no param construcror.");
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public void writeExternal(ObjectOutput out) throws IOException {
        out.writeObject(this.name);
        out.writeInt(this.age);
    }

    @Override
    public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException {
        this.name = (String) in.readObject();
        this.age = in.readInt();
    }

    @Override
    public String toString() {
        return "User{"
                + "name='" + name + '\''
                + ", age=" + age
                + '}';
    }
}
```

```Java
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;

public class ExternalizableDemo {

    public static void main(String[] args) {
        User1 user = new User1("mars", 20);
        System.out.println(user); // output: User{name='mars', age=20}
        ObjectOutputStream oos = null;
        try {
            oos = new ObjectOutputStream(new FileOutputStream("user.txt"));
            oos.writeObject(user);
            oos.close();
        } catch (IOException e) {
            e.printStackTrace();
        }

        ObjectInputStream ois = null;
        try {
            ois = new ObjectInputStream(new FileInputStream("user.txt"));
            User1 readUser = (User1) ois.readObject();
            System.out.println(readUser); // User{name='mars', age=20}
            ois.close();
        } catch (IOException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}
//output:
//User{name='mars', age=20}
//no param construcror.
//User{name='mars', age=20}
```
执行代码时候发现，会调用User1的无参构造器。当使用Externalizable机制反序列化的时候，会先使用public的无参构造器创建实例，然后才调用readExternal()方法进行反序列化。因此实现Externalizable的类必须提供public的无参构造器，否则会报no valid constructor错误。
## Transient关键字
通过在实例变量前面使用transient关键字修饰，可以指定Java序列化时无需理会该实例变量。  
## 版本
java序列化机制允许为序列化类提供一个序列化ID(private static final long serialVersionUID)，该变量的值用来标识该Java类的序列化版本。如果一个类升级后，只要它的serialVersionUID类变量的值没有改变，序列化机制会把它们当成同一个序列化版本。  
序列化ID有两种生成策略，一个是固定的1L，一个是随机生成一个不重复的long类型数据(由JVM根据类的相关信息计算得出)。没有特殊要求，默认1L即可。
## 参考资料
《疯狂Java讲义第四版》  
[Java对象的序列化与反序列化](https://www.hollischuang.com/archives/1150)  
[深入分析Java的序列化与反序列化](https://www.hollischuang.com/archives/1140)  
[Java 序列化的高级认识](https://www.ibm.com/developerworks/cn/java/j-lo-serial/)
