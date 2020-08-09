## 反射基础
1. Java反射机制是在运行状态中，对任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意方法和属性；这种动态获取信息以及动态调用对象方法的功能成为java的反射机制。
2. 要理解反射，首先要理解Class类；
	- Class类是一个用来描述类的类；
	- Class类不能通过new关键字来创建对象，因为其构造器是私有的；
	- 只有java虚拟机能够创建class对象（类加载时，在堆中创建一个Class对象，作为访问方法区运行时数据结构的入口）；
3. 得到Class类对象的方法 
	- 通过Class.forName("全限定类名")方法得到Class类的一个对象；
		- `Class<?> cls = Class.forName("java.util.Date");`
	- 通过一个实例对象的 getClass() 方法 
		- `Date date = new Date(); Class<?> cls = date.getClass();`
	- 通过指定类型的 .class 属性
		- Class<?> cls = Date.class;
4. 动态加载类/静态加载类 
	- 动态加载类：在运行时加载的类
	- 静态加载类：在编译时加载的类
5. 反射的用法
	- 反射实例化对象：有了它就可以不通过new关键字获得对象了，而new是造成耦合的最大元凶(newInstance)！
	- 使用反射调用构造,进而根据具体构造函数得到相应对象(Constructor);
	- 反射实现任意类的指定方法调用(Method, invoke);
	- 反射调用类中任意成员；
## 代码
```java
import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;
import java.util.Date;

public class Reflection {
    class Book{
        String name;

        public Book(String name) {
            this.name = name;
        }

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }
    }
    public static void main(String[] args) throws ClassNotFoundException,
            IllegalAccessException, InstantiationException, NoSuchMethodException,
            InvocationTargetException, NoSuchFieldException {
        System.out.println("先有类后有对象！");
        Date date = new Date();
        System.out.println(date);
        System.out.println("===============");

        System.out.println("通过反射得到Class对象的三种方式");
        System.out.println("1、调用Object类的getClass方法");
        Class<?> cls = date.getClass();
        System.out.println(cls);

        System.out.println("2、使用类.class获得");
        cls = Date.class;
        System.out.println(cls);

        System.out.println("3、通过实例化Class对象，即通过类的全限定类名调用forName，" +
                "此时可以不使用import语句导入一个明确的类，而是通过字符串的形式描述。");
        cls = Class.forName("java.util.Date");
        System.out.println(cls);
        System.out.println("=================");

        System.out.println("有哪些用处？");
        System.out.println("1、反射实例化对象：有了它就可以不通过new关键字获得对象了，而new是造成耦合的最大元凶！");
        System.out.println("比如一个工厂类，如果用new来创建对象的话，那" +
                "么每添加一个类实现，就要改变工厂类（调整if else），但是用反射来实现就不需要调整");
        date = (Date) cls.getConstructor().newInstance();
        System.out.println(date);

        System.out.println("2、使用反射调用构造,进而根据具体构造函数得到相应对象");
        Constructor<?> con = cls.getConstructor(long.class);
        System.out.println(con.newInstance(21234566L));

        System.out.println("3、反射实现任意类的指定方法调用");
        Method getM = cls.getMethod("getTime");
        Method setM = cls.getMethod("setTime", long.class);
        setM.invoke(date, 1234857L);
        System.out.println(getM.invoke(date));

        System.out.println("4. 反射获得某个对象实现的所有接口所对应的class类");
        Class<?>[] interfaces = cls.getInterfaces();
        cls.getMethods();
        for (int i = 0; i < interfaces.length; i++) {
            System.out.println(interfaces[i].getMethods());
        }

        System.out.println("5. 通过反射得到对象的类加载器");
        System.out.println(cls.getClassLoader());

        System.out.println("6、反射调用成员");
        Class<?> clsBook = Class.forName("Reflection.Book");
        Object obj = clsBook.getConstructor().newInstance();//必须实例化对象！
        Field[] fields = clsBook.getDeclaredFields();
        for (int i = 0; i < fields.length; i++) {
            fields[i].setAccessible(true);
            fields[i].set(obj, "Java反射");
            System.out.println(fields[i].get(obj));
        }
    }
}
```

