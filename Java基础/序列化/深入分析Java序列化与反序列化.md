## 导读
本文主要涉及以下几个问题：
* 父类的序列化
* 对象引用的序列化
* 静态变量序列化
* 自定义序列化

### 父类的序列化
如果一个子类实现了Serializable接口，要想将它的父类对象也序列化，就需要让父类也实现Serializable接口。如果父类是不可序列化的，只是带有无参数的构造器，则该父类中定义的成员变量值不会序列化到二进制流中。如果父类是不可序列化的，也不带有无参数的构造器，则反序列化时将抛出InvalidClassException异常。
```java
import java.io.Serializable;

public class Parent implements Serializable{
    private String name;
    private int age;
    public String eat;

    public Parent(String eat) {
        this.eat = eat;
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

    public String getEat() {
        return eat;
    }

    public void setEat(String eat) {
        this.eat = eat;
    }

    @Override
    public String toString() {
        return "Parent{"
                + "name='" + name + '\''
                + ", age=" + age
                + ", eat=" + eat
                + '}';
    }
}
```
```Java
import java.io.Serializable;

public class Child extends Parent implements Serializable {
    private String childName;
    private int childAge;

    public Child(String childName, int childAge, String eat) {
        super(eat);
        this.childName = childName;
        this.childAge = childAge;
    }
    public String getName() {
        return childName;
    }

    public void setName(String childName) {
        this.childName = childName;
    }

    public int getAge() {
        return childAge;
    }

    public void setAge(int childAge) {
        this.childAge = childAge;
    }

    @Override
    public String toString() {
        return "Child{"
                + "childName='" + childName + '\''
                + ", childAge=" + childAge
                + ", eat=" + eat
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

public class ParentSerializableDemo {
    public static void main(String[] args) {
        Child child = new Child("mars", 2, "fish");

        ObjectOutputStream oos = null;
        try {
            oos = new ObjectOutputStream(new FileOutputStream("parent.txt"));
            oos.writeObject(child);
            oos.close();
        } catch (IOException e) {
            e.printStackTrace();
        }

        ObjectInputStream ois = null;
        try {
            ois = new ObjectInputStream(new FileInputStream("parent.txt"));
            Child readChild = (Child) ois.readObject();
            ois.close();
            System.out.println(readChild); // output: Child{childName='mars', childAge=2, eat=fish}
        } catch (IOException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }

}
```
### 对象引用的序列化
如果某个类的成员变量的类型是另一个引用类型，那么这个引用类必须是可序列化的，否则拥有该类型成员变量的类也是不可序列化的。
```Java
import java.io.Serializable;

public class User implements Serializable {

    private String name;
    private int age;

    public User(String name, int age) {
        this.name = name;
        this.age = age;
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
    public String toString() {
        return "User{"
                + "name='" + name + '\''
                + ", age=" + age
                + '}';
    }
}
```

```Java
import java.io.Serializable;

public class Product implements Serializable {

    private String name;
    private User user;

    public Product(String name, User user) {
        this.name = name;
        this.user = user;
    }

    public String getName() {
        return this.name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public User getUser() {
        return this.user;
    }

    public void setUser(User user) {
        this.user = user;
    }

    @Override
    public String toString() {
        return "product{"
                + "name='" + name + '\''
                + ", user=" + user
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

public class ProductDemo {
    public static void main(String[] args) {
        Product product = new Product("apple", new User("mars", 20));
        System.out.println(product); // output: product{name='apple', user=User{name='mars', age=20}}
        ObjectOutputStream oos = null;
        try {
            oos = new ObjectOutputStream(new FileOutputStream("product.txt"));
            oos.writeObject(product);
            oos.close();
        } catch (IOException e) {
            e.printStackTrace();
        }

        ObjectInputStream ois = null;
        try {
            ois = new ObjectInputStream(new FileInputStream("product.txt"));
            Product readProduct = (Product) ois.readObject();
            ois.close();
            System.out.println(readProduct); // output: product{name='apple', user=User{name='mars', age=20}}
        } catch (IOException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}
```
假设有如下特殊情况，程序有两个product对象，他们的user实例变量都引用到同一个User对象，而且该User对象还有一个引用变量引用它。
```Java
User user = new User("mars", 20);
Product product1 = new Product("apple", user);
Product product2 = new Product("pen", user);
```
Java序列化机制采用了一种特殊的序列化算法：
* 所有保存到磁盘中的对象都有一个序列化编号
* 当程序试图序列化一个对象时，程序将先检查该对象是否已经被序列化过，只有该对象从未被序列化过，系统才会将该对象转换成字节序列并输出
* 如果这个对象已经序列化过，程序只是直接输出一个序列化编号
```Java
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;

public class ProductDemo1 {
    public static void main(String[] args) {
        User user = new User("mars", 20);
        Product product1 = new Product("apple", user);
        Product product2 = new Product("pen", user);

        ObjectOutputStream oos = null;
        try {
            oos = new ObjectOutputStream(new FileOutputStream("product.txt"));
            oos.writeObject(product1);
            oos.writeObject(product2);
            oos.writeObject(user);
            oos.writeObject(product2);
            oos.close();
        } catch (IOException e) {
            e.printStackTrace();
        }

        ObjectInputStream ois = null;
        try {
            ois = new ObjectInputStream(new FileInputStream("product.txt"));
            Product readProduct1 = (Product) ois.readObject();
            Product readProduct2 = (Product) ois.readObject();
            User readUser = (User) ois.readObject();
            Product readProduct3 = (Product) ois.readObject();
            ois.close();
            System.out.println(readProduct1.getUser() == readUser); // output: true
            System.out.println(readProduct2.getUser() == readUser); // output: true
            System.out.println(readProduct3.getUser() == readUser); // output: true
            System.out.println(readProduct2 == readProduct3); // output: true
        } catch (IOException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}
```
这样会有一个潜在问题，当程序序列化一个可变对象时，只有第一次使用writeObject()方法才会将该对象转换成二进制字节流输出，程序再次调用writeObject()方法时，只是输出前面的序列化编号，即使后面该对象实例变量值已经被改变，改变的变量值也不会输出。
```Java
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;

public class ProductDemo2 {
    public static void main(String[] args) {
        Product product = new Product("apple", new User("mars", 20));
        ObjectOutputStream oos = null;
        try {
            oos = new ObjectOutputStream(new FileOutputStream("product.txt"));
            oos.writeObject(product);
            product.setName("pen");
            System.out.println(product); // output: product{name='pen', user=User{name='mars', age=20}}
            oos.writeObject(product);
            oos.close();
        } catch (IOException e) {
            e.printStackTrace();
        }

        ObjectInputStream ois = null;
        try {
            ois = new ObjectInputStream(new FileInputStream("product.txt"));
            Product readProduct = (Product) ois.readObject();
            Product readProduct1 = (Product) ois.readObject();
            ois.close();
            System.out.println(readProduct); // output: product{name='apple', user=User{name='mars', age=20}}
            System.out.println(readProduct1); // output: product{name='apple', user=User{name='mars', age=20}}
        } catch (IOException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}
```
### 静态变量序列化
如下程序所示，User中有一个静态变量brithday值为1999-02-01，在序列化之后将值改为1991-02-08，反序列化后brithday值为1991-02-08。因为在序列化时，并不保存静态变量，序列化保存的是对象的状态，静态变量属于类的状态，因此序列化并不保存静态变量。
```Java
import java.io.Serializable;

public class User implements Serializable {

    public static String brithday = "1999-02-01";
    public String eat = "fish";

    private String name;
    private int age;

    public User(String name, int age) {
        this.name = name;
        this.age = age;
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
    public String toString() {
        return "User{"
                + "name='" + name + '\''
                + ", age=" + age
                + ", eat=" + eat
                + ", brithday=" + brithday
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

public class SerializableDemo {

    public static void main(String[] args) {
        User user = new User("mars", 20);
        System.out.println(user); // output: User{name='mars', age=20, eat=fish, brithday=1999-02-01}
        ObjectOutputStream oos = null;
        try {
            oos = new ObjectOutputStream(new FileOutputStream("user.txt"));
            oos.writeObject(user);
            user.eat = "meat";
            User.brithday = "1991-02-08";
            oos.close();
        } catch (IOException e) {
            e.printStackTrace();
        }

        ObjectInputStream ois = null;
        try {
            ois = new ObjectInputStream(new FileInputStream("user.txt"));
            User readUser = (User) ois.readObject();
            System.out.println(readUser); // output: User{name='mars', age=20, eat=fish, brithday=1991-02-08}
            ois.close();
        } catch (IOException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}
```
## 参考资料
《疯狂Java讲义第四版》  
[深入分析Java的序列化与反序列化](https://www.hollischuang.com/archives/1140)  
[Java 序列化的高级认识](https://www.ibm.com/developerworks/cn/java/j-lo-serial/)
