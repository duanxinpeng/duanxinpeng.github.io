## java Queue
1. add/offer
	- add：向队列末尾添加一个元素，若队列已满，则抛出异常
	- offer：不会抛异常，返回false。
2. remove/poll
	- remove：移除队列队头元素，若队列为空，则抛出异常
	- poll：不抛出异常，返回null
3. element/peek
	- element：查询队列头部元素，若队列为空，则抛出异常
	- peek：不会抛出异常，返回null
	
## 形参/实参
1. 用来接收调用该方法时传递的参数。只有在被调用的时候才分配内存空间，一旦调用结束，就释放内存空间。因此仅仅在方法内有效。
2. 传递给被调用方法的值，预先创建并赋予确定值。
3. 传值调用：传值调用中传递的参数为基本数据类型，参数视为形参。
4. 传引用调用：传引用调用中，如果传递的参数是引用数据类型，参数视为实参。在调用的过程中，将实参的地址传递给了形参，形参上的改变都发生在实参上。

## 泛型
1. 参数化类型，也就是将类型参数化，可以通过传递参数来设置变量类型。
2. ArrayList 默认是Object类型的，所以只要是Object的子类，什么都可以放
3. 为什么ArrayList的传入参数类型不是Object，而是一个泛型？可以规范传入参数类型，只允许传入某种类型，从而在编译期就发现问题，不要将问题遗留到运行期。
4. 泛型只有在编译期才有效
5. 三种用法
	- 泛型类：在实例化类的时候指明泛型类型
	- 泛型接口
	- 泛型方法：在调用方法的时候指明泛型类型
6. https://www.cnblogs.com/coprince/p/8603492.html
```java
public class GenericTest {
   //这个类是个泛型类，在上面已经介绍过
   public class Generic<T>{     
        private T key;

        public Generic(T key) {
            this.key = key;
        }

        //我想说的其实是这个，虽然在方法中使用了泛型，但是这并不是一个泛型方法。
        //这只是类中一个普通的成员方法，只不过他的返回值是在声明泛型类已经声明过的泛型。
        //所以在这个方法中才可以继续使用 T 这个泛型。
        public T getKey(){
            return key;
        }

        /**
         * 这个方法显然是有问题的，在编译器会给我们提示这样的错误信息"cannot reslove symbol E"
         * 因为在类的声明中并未声明泛型E，所以在使用E做形参和返回值类型时，编译器会无法识别。
        public E setKey(E key){
             this.key = keu
        }
        */
    }

    /** 
     * 这才是一个真正的泛型方法。
     * 首先在public与返回值之间的<T>必不可少，这表明这是一个泛型方法，并且声明了一个泛型T
     * 这个T可以出现在这个泛型方法的任意位置.
     * 泛型的数量也可以为任意多个 
     *    如：public <T,K> K showKeyName(Generic<T> container){
     *        ...
     *        }
     */
    public <T> T showKeyName(Generic<T> container){
        System.out.println("container key :" + container.getKey());
        //当然这个例子举的不太合适，只是为了说明泛型方法的特性。
        T test = container.getKey();
        return test;
    }

    //这也不是一个泛型方法，这就是一个普通的方法，只是使用了Generic<Number>这个泛型类做形参而已。
    public void showKeyValue1(Generic<Number> obj){
        Log.d("泛型测试","key value is " + obj.getKey());
    }

    //这也不是一个泛型方法，这也是一个普通的方法，只不过使用了泛型通配符?
    //同时这也印证了泛型通配符章节所描述的，?是一种类型实参，可以看做为Number等所有类的父类
    public void showKeyValue2(Generic<?> obj){
        Log.d("泛型测试","key value is " + obj.getKey());
    }

     /**
     * 这个方法是有问题的，编译器会为我们提示错误信息："UnKnown class 'E' "
     * 虽然我们声明了<T>,也表明了这是一个可以处理泛型的类型的泛型方法。
     * 但是只声明了泛型类型T，并未声明泛型类型E，因此编译器并不知道该如何处理E这个类型。
    public <T> T showKeyName(Generic<E> container){
        ...
    }  
    */

    /**
     * 这个方法也是有问题的，编译器会为我们提示错误信息："UnKnown class 'T' "
     * 对于编译器来说T这个类型并未项目中声明过，因此编译也不知道该如何编译这个类。
     * 所以这也不是一个正确的泛型方法声明。
    public void showkey(T genericObj){

    }
    */

    public static void main(String[] args) {


    }
}
```

## ConcurrentHashMap 的 get 方法为什么不需要加锁
1. 获取 hash 桶的根节点通过 `tabAt` 来完成，线程安全
2. 遍历的时候用到了节点的 next 属性，被 volative 关键字修饰，线程间可见，线程安全 
3. 获取节点的 value 时，节点的 value 值也是被 volatile 修饰，线程间可见，线程安全。
4. 如果正在扩容中，会根据 ForwardingNode get到新表中的内容。

## ConcurrentHashMap 的 putVal 是如何做的？
1. 根据 hashcode 利用 tabAt 函数得到插入位置的下标
2. 判读数组中此处元素是否为 null，如果为 null，直接采用 CAS 进行插入操作即可。
3. 如果不为空并且发现正在进行 rehash，此线程就会帮忙 rehash 操作。
4. 如果部位空，且没在进行 rehash，会用 synchronized 关键字锁住头节点，进行插入操作（尾插法）

## HashMap的扰动函数的作用？
1. 主要作用就是使哈希更加均匀，避免过多的哈希冲突
2. 由于哈希过程相当于对hashmap的 length 取模（length 必须是2的n次方，这样和 `length-1` 与操作就等同于取模操作了），会使得 hashcode 的高位信息丢失，导致哈希不均匀。
3. 扰动函数就是将 hashcode 的高位与其低位进行异或操作，这样低位就既包含了整个 hashcode 的信息。
4. 对经过扰动函数处理之后的 hashcode 再进行取模，获得的结果就更均匀了。

## HashMap 的 rehash 与 redis 的 rehash 有什么不同？
1. redis 采用的是 渐进式哈希，hashmap 采用的是一次性哈希
2. 渐进式哈希 
	- 保存两个哈希表，一个新，一个旧。
	- 每次进行增删改查时，都会检测是否正在进行 rehash，如果正在进行 rehash 操作，就会将 rehashindex 指向的桶中的所有元素移到新表中。
	- 如果系统比较闲，则会出发定时函数来完成 rehash 操作 
	- 增操作直接在新表上进行
	- 删改查操作先在旧表上操作，旧表上没有再去新表查找。
	
## ConcurrentHashMap（jdk8）的多线程协同 rehash 是怎么回事?
1. 多线程协助扩容，每个线程默认处理16个桶
2. 读操作线程不需要帮忙扩容，写操作线程必须先帮忙扩容之后再进行写操作。
3. 对于每个桶内的元素进行移动时，将其 hahscode 与 新 `length`(这里不是 length-1 !) 做与操作，结果为0 放在低位，结果为1 放在高位
4. 之所以这样做，是因为一个元素在扩容后的数组中的位置要么是在原位置，要么是原位置的基础上加上旧表的length得到新位置。
5. 与旧表的 length = 4（100） ，只是单纯的看最高位是否为 1， 不为 1 放低位，为1 放高位。

## 为什么HashMap 允许 key/value 为 null，但ConcurrentHashMap不允许？
1. ConcurrentHashMap 遇到 key/value 为 null 会报空指针错误

## java类名是否允许相同？比如和jdk中某个类相同？
1. 可以，只要不在同一个 packet 中即可！

## lambda 表达式
1. lambda 表达式中只允许引用声明为常量的局部变量