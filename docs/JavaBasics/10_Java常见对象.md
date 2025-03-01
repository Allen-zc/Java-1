# Java 常见类 I

## Object 

Object类是类层次结构的根类。每个类都使用 Object 作为超类。每个类都直接或者间接的继承自Object类。

Object 中常用方法有：

```java
public int hashCode() //返回该对象的哈希码值。
// 注意：哈希值是根据哈希算法计算出来的一个值，这个值和地址值有关，但是不是实际地址值。

public final Class getClass() //返回此 Object 的运行时类

public String toString() //返回该对象的字符串表示。

protected Object clone() //创建并返回此对象的一个副本。可重写该方法

protected void finalize() 
// 当垃圾回收器确定不存在对该对象的更多引用时，由对象的垃圾回收器调用此方法。用于垃圾回收，但是什么时候回收不确定。
```

### equals() 方法

**1. 等价关系** 

Ⅰ 自反性

```java
x.equals(x); // true
```

Ⅱ 对称性

```java
x.equals(y) == y.equals(x); // true
```

Ⅲ 传递性

```java
if (x.equals(y) && y.equals(z))
    x.equals(z); // true;
```

Ⅳ 一致性

多次调用 equals() 方法结果不变

```java
x.equals(y) == x.equals(y); // true
```

Ⅴ 与 null 的比较

对任何**不是 null 的对象** x 调用 x.equals(null) 结果都为 false

```java
x.equals(null); // false;
```

**2. 等价与相等** 

- 对于基本类型，== 判断两个值是否相等，基本类型没有 equals() 方法。
- 对于引用类型，== 判断两个变量是否引用同一个对象，而 equals() 判断引用的对象是否等价。

```java
Integer x = new Integer(1);
Integer y = new Integer(1);
System.out.println(x.equals(y)); // true
System.out.println(x == y);      // false
```

**3. 实现** 

- 检查是否为同一个对象的引用，如果是直接返回 true；
- 检查是否是同一个类型，如果不是，直接返回 false；
- 将 Object 对象进行转型；
- 判断每个关键域是否相等。

```java
public class EqualExample {

    private int x;
    private int y;
    private int z;

    public EqualExample(int x, int y, int z) {
        this.x = x;
        this.y = y;
        this.z = z;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;  //检查是否为同一个对象的引用，如果是直接返回 true；
        if (o == null || getClass() != o.getClass()){
            //检查是否是同一个类型，如果不是，直接返回 false
            return false;
        }

        // 将 Object 对象进行转型
        EqualExample that = (EqualExample) o;

        // 判断每个关键域是否相等。
        if (x != that.x) return false;
        if (y != that.y) return false;
        return z == that.z;
    }
}
```

### hashCode() 方法

hashCode() 返回散列值，而 equals() 是用来判断两个对象是否等价。
**等价的两个对象散列值一定相同，但是散列值相同的两个对象不一定等价**。

**在覆盖 equals() 方法时应当总是覆盖 hashCode() 方法，保证等价的两个对象散列值也相等**。

下面的代码中，新建了两个等价的对象，并将它们添加到 HashSet 中。
我们希望将这两个对象当成一样的，只在集合中添加一个对象，但是因为 EqualExample 没有实现 hasCode() 方法，
因此这两个对象的散列值是不同的，最终导致集合添加了两个等价的对象。

```java
EqualExample e1 = new EqualExample(1, 1, 1);
EqualExample e2 = new EqualExample(1, 1, 1);
System.out.println(e1.equals(e2)); // true
HashSet<EqualExample> set = new HashSet<>();
set.add(e1);
set.add(e2);
System.out.println(set.size());   // 2
```

理想的散列函数应当具有均匀性，即不相等的对象应当均匀分布到所有可能的散列值上。
这就要求了散列函数要把**所有域的值都考虑进来**。
可以将每个域都当成 R 进制的某一位，然后组成一个 R 进制的整数。
R 一般取 31，因为它是一个奇素数，如果是偶数的话，当出现乘法溢出，信息就会丢失，因为与 2 相乘相当于向左移一位。

一个数与 31 相乘可以转换成移位和减法：`31*x == (x<<5)-x`，编译器会自动进行这个优化。

```java
@Override
public int hashCode() {
    int result = 17;
    result = 31 * result + x;
    result = 31 * result + y;
    result = 31 * result + z;
    return result;
}
```

> 了解：IDEA中 Alt+Insert 快捷键就可以快速生成 hashCode() 和 equals() 方法。

### toString() 方法

默认返回 ToStringExample@4554617c 这种形式，其中 @ 后面的数值为散列码的无符号十六进制表示。

```java
public class ToStringExample {

    private int number;

    public ToStringExample(int number) {
        this.number = number;
    }
}
```

```java
ToStringExample example = new ToStringExample(123);
System.out.println(example.toString());
```

```html
ToStringExample@4554617c
```

### clone() 方法

**1. Cloneable** 

clone() 是 Object 的 **protected 方法**，它不是 public，一个类不显式去重写 clone()，其它类就不能直接去调用该类实例的 clone() 方法。

```java
public class CloneExample {
    private int a;
    private int b;
}
```

```java
CloneExample e1 = new CloneExample();
// CloneExample e2 = e1.clone(); 
// 'clone()' has protected access in 'java.lang.Object'
```

重写 clone() 得到以下实现：

```java
public class CloneExample {
    private int a;
    private int b;

    // CloneExample 默认继承 Object
    @Override
    public CloneExample clone() throws CloneNotSupportedException {
        return (CloneExample)super.clone();
    }
}
```

```java
CloneExample e1 = new CloneExample();
try {
    CloneExample e2 = e1.clone();
} catch (CloneNotSupportedException e) {
    e.printStackTrace();
}
```

```html
java.lang.CloneNotSupportedException: CloneExample
```

以上抛出了 CloneNotSupportedException，这是因为 CloneExample 没有实现 Cloneable 接口。

应该注意的是，**clone() 方法并不是 Cloneable 接口的方法，而是 Object 的一个 protected 方法**。

**Cloneable 接口只是规定，如果一个类没有实现 Cloneable 接口又调用了 clone() 方法，就会抛出 CloneNotSupportedException**。

```java
public class CloneExample implements Cloneable {
    private int a;
    private int b;

    @Override
    public Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}
```

**2. 浅拷贝** 

拷贝对象和原始对象的引用类型引用同一个对象。

```java
public class ShallowCloneExample implements Cloneable {

    private int[] arr;

    public ShallowCloneExample() {
        arr = new int[10];
        for (int i = 0; i < arr.length; i++) {
            arr[i] = i;
        }
    }

    public void set(int index, int value) {
        arr[index] = value;
    }

    public int get(int index) {
        return arr[index];
    }

    @Override
    protected ShallowCloneExample clone() throws CloneNotSupportedException {
        return (ShallowCloneExample) super.clone();
    }
}
```

```java
// 拷贝对象和原始对象的引用类型引用同一个对象。
ShallowCloneExample e1 = new ShallowCloneExample();
ShallowCloneExample e2 = null;
try {
    e2 = e1.clone();
} catch (CloneNotSupportedException e) {
    e.printStackTrace();
}
e1.set(2, 222);
System.out.println(e1.get(2)); // 222
System.out.println(e2.get(2)); // 222
```

**3. 深拷贝** 

拷贝对象和原始对象的引用类型引用不同对象。

```java
public class DeepCloneExample implements Cloneable {

    private int[] arr;

    public DeepCloneExample() {
        arr = new int[10];
        for (int i = 0; i < arr.length; i++) {
            arr[i] = i;
        }
    }

    public void set(int index, int value) {
        arr[index] = value;
    }

    public int get(int index) {
        return arr[index];
    }

    @Override
    protected DeepCloneExample clone() throws CloneNotSupportedException {
        DeepCloneExample result = (DeepCloneExample) super.clone();
        // 创建新对象
        result.arr = new int[arr.length];
        for (int i = 0; i < arr.length; i++) {
            result.arr[i] = arr[i];
        }
        return result;
    }
}
```

```java
DeepCloneExample e1 = new DeepCloneExample();
DeepCloneExample e2 = null;
try {
    e2 = e1.clone();
} catch (CloneNotSupportedException e) {
    e.printStackTrace();
}
e1.set(2, 222);
System.out.println(e1.get(2)); // 222
System.out.println(e2.get(2)); // 2
```

**4. clone() 的替代方案** 

使用 clone() 方法来拷贝一个对象即复杂又有风险，它会抛出异常，并且还需要类型转换。
Effective Java 书上讲到，最好不要去使用 clone()，可以使用**拷贝构造函数或者拷贝工厂来拷贝一个对象**。

```java
public class CloneConstructorExample {

    private int[] arr;

    public CloneConstructorExample() { //构造函数
        arr = new int[10];
        for (int i = 0; i < arr.length; i++) {
            arr[i] = i;
        }
    }

    public CloneConstructorExample(CloneConstructorExample original) { // 拷贝构造函数
        arr = new int[original.arr.length];
        for (int i = 0; i < original.arr.length; i++) {
            arr[i] = original.arr[i];
        }
    }

    public void set(int index, int value) {
        arr[index] = value;
    }

    public int get(int index) {
        return arr[index];
    }
}
```

```java
CloneConstructorExample e1 = new CloneConstructorExample();
CloneConstructorExample e2 = new CloneConstructorExample(e1);
e1.set(2, 222);
System.out.println(e1.get(2)); // 222
System.out.println(e2.get(2)); // 2
```



## String

字符串：就是由多个字符组成的一串数据。也可以看成是一个字符数组。

### 构造方法

```java
public String() //空构造

public String(byte[] bytes) //把字节数组转成字符串

public String(byte[] bytes,int index,int length) //把字节数组的一部分转成字符串

public String(char[] value) //把字符数组转成字符串

public String(char[] value,int index,int count) //把字符数组的一部分转成字符串,第三个参数表示的是数目

public String(String original) //把字符串常量值转成字符串
```

使用示例：

```java
public class StringDemo {
    public static void main(String[] args) {
        //public String():空构造
        String s=new String();
        System.out.println("s:"+s);
        System.out.println("s.length="+s.length());
        System.out.println("-----------------------");

        //public String(byte[] bytes):把字节数组转成字符串
        byte[] bys={97,98,99,100,101};

        String s2=new String(bys);
        System.out.println("s2:"+s2);
        System.out.println("s2.length="+s2.length());
        System.out.println("-----------------------");

        //public String(byte[] bytes,int index,int length):把字节数组的一部分转成字符串
        String s3=new String(bys,0,3); //从0位置开始,3个字符
        System.out.println("s3:"+s3);//s3:abc
        System.out.println("s3.length="+s3.length());//s3.length=3
        System.out.println("-----------------------");


        //public String(char[] value):把字符数组转成字符串
        char[] chs={'a','b','c','d','e'};
        String s4=new String(chs); //从0位置开始,3个字符
        System.out.println("s4:"+s4);
        System.out.println("s4.length="+s4.length());
        System.out.println("-----------------------");

        //public String(char[] value,int index,int count):把字符数组的一部分转成字符串
        String s5=new String(chs,0,3); //从0位置开始,3个字符
        System.out.println("s5:"+s5);
        System.out.println("s5.length="+s5.length());
        System.out.println("-----------------------");

        //public String(String original):把字符串常量值转成字符串
        String s6=new String("abcde");
        System.out.println("s6:"+s6);
        System.out.println("s6.length="+s6.length());
    }
}
```

字符串的特点：一旦被赋值，就不能改变

```java
public static void test() {
    String s = "hello";
    s += "world";
    System.out.println("s:" + s); // helloworld
}
```

小练习：

```java
public static void test2(){
    String s1 = new String("hello");
    String s2 = new String("hello");
    System.out.println(s1 == s2); // false
    System.out.println(s1.equals(s2)); // true

    String s3 = new String("hello");
    String s4 = "hello";
    System.out.println(s3 == s4); // false
    System.out.println(s3.equals(s4)); // true

    String s5 = "hello";
    String s6 = "hello";
    System.out.println(s5 == s6); // true
    System.out.println(s5.equals(s6)); // true
}
```

### 判断功能

```java
boolean equals(Object obj) //比较字符串的内容是否相同,区分大小写

boolean equalsIgnoreCase(String str) //比较字符串的内容是否相同,忽略大小写

boolean contains(String str) //判断大字符串中是否包含小字符串

boolean startsWith(String str) //判断字符串是否以某个指定的字符串开头

boolean endsWith(String str) //判断字符串是否以某个指定的字符串结尾

boolean isEmpty()// 判断字符串是否为空
```

注意：字符串内容为空和字符串对象为空。

```java
String s = "";//字符串内容为空
String s = null;//字符串对象为空
```

使用示例：

```java
public class StringDemo3 {
    public static void main(String[] args) {
        // 创建字符串对象
        String s1 = "helloworld";
        String s2 = "helloworld";
        String s3 = "HelloWorld";

        // boolean equals(Object obj):比较字符串的内容是否相同,区分大小写
        System.out.println("equals:" + s1.equals(s2));//true
        System.out.println("equals:" + s1.equals(s3));//false
        System.out.println("-----------------------");

        // boolean equalsIgnoreCase(String str):比较字符串的内容是否相同,忽略大小写
        System.out.println("equals:" + s1.equalsIgnoreCase(s2));//true
        System.out.println("equals:" + s1.equalsIgnoreCase(s3));//true
        System.out.println("-----------------------");

        // boolean contains(String str):判断大字符串中是否包含小字符串
        System.out.println("contains:" + s1.contains("hello"));//true
        System.out.println("contains:" + s1.contains("hw"));//false
        System.out.println("-----------------------");

        // boolean startsWith(String str):判断字符串是否以某个指定的字符串开头
        System.out.println("startsWith:" + s1.startsWith("h"));//true
        System.out.println("startsWith:" + s1.startsWith("hello"));//true
        System.out.println("startsWith:" + s1.startsWith("world"));//false
        System.out.println("-----------------------");

        //boolean endsWith(String str):判断字符串是否以某个指定的字符串结尾
        System.out.println("startsWith:" + s1.endsWith("d"));//true
        System.out.println("startsWith:" + s1.endsWith("world"));//true
        System.out.println("startsWith:" + s1.endsWith("hello"));//false
        System.out.println("-----------------------");

        // boolean isEmpty():判断字符串是否为空。
        System.out.println("isEmpty:" + s1.isEmpty());//false

        String s4 = ""; //字符串内容为空
        String s5 = null;//字符串对象为空
        System.out.println("isEmpty:" + s4.isEmpty());//true
        // NullPointerException
        // s5对象都不存在，所以不能调用方法，空指针异常
        //System.out.println("isEmpty:" + s5.isEmpty());
    }
}
```

### 获取功能

```java
int length() //获取字符串的长度。

char charAt(int index) //获取指定索引位置的字符

int indexOf(int ch) //返回指定字符在此字符串中第一次出现处的索引。为什么这里参数int类型，而不是char类型?原因是：'a'和97其实都可以代表'a'

int indexOf(String str) //返回指定字符串在此字符串中第一次出现处的索引。

int indexOf(int ch,int fromIndex) //返回指定字符在此字符串中从指定位置后第一次出现处的索引。

int indexOf(String str,int fromIndex) //返回指定字符串在此字符串中从指定位置后第一次出现处的索引。

String substring(int start) //从指定位置开始截取字符串,默认到末尾。

String substring(int start,int end) //从指定位置开始到指定位置结束截取字符串。左闭右开
```

使用示例：

```java
public class StringDemo4 {
    public static void main(String[] args) {
        // 定义一个字符串对象
        String s = "helloworld";

        // int length():获取字符串的长度。
        System.out.println("s.length:" + s.length());//10
        System.out.println("----------------------");

        // char charAt(int index):获取指定索引位置的字符
        System.out.println("charAt:" + s.charAt(7));//r
        System.out.println("----------------------");

        // int indexOf(int ch):返回指定字符在此字符串中第一次出现处的索引。
        System.out.println("indexOf:" + s.indexOf('l'));//2
        System.out.println("----------------------");

        // int indexOf(String str):返回指定字符串在此字符串中第一次出现处的索引。
        System.out.println("indexOf:" + s.indexOf("owo"));//4
        System.out.println("----------------------");

        // int indexOf(int ch,int fromIndex):返回指定字符在此字符串中从指定位置后第一次出现处的索引。
        System.out.println("indexOf:" + s.indexOf('l', 4));//8
        System.out.println("indexOf:" + s.indexOf('k', 4)); // -1
        System.out.println("indexOf:" + s.indexOf('l', 40)); // -1
        System.out.println("----------------------");

        // int indexOf(String str,intfromIndex):返回指定字符串在此字符串中从指定位置后第一次出现处的索引。
        System.out.println("indexOf:" + s.indexOf("owo", 4));//4
        System.out.println("indexOf:" + s.indexOf("ll", 4)); //-1
        System.out.println("indexOf:" + s.indexOf("ld", 40)); // -1
        System.out.println("----------------------");

        // String substring(int start):从指定位置开始截取字符串,默认到末尾。包含start这个索引
        System.out.println("substring:" + s.substring(5));//world
        System.out.println("substring:" + s.substring(0));//helloworld
        System.out.println("----------------------");

        // String substring(int start,int
        // end):从指定位置开始到指定位置结束截取字符串。包括start索引但是不包end索引
        System.out.println("substring:" + s.substring(3, 8));//lowor
        System.out.println("substring:" + s.substring(0, s.length()));//helloworld

        /**
         * 获取 字符串中的每个字符
         */
        for(int i=0;i<s.length();i++){
            System.out.println(s.charAt(i));
        }
    }
}
```

### 转换功能

```java
byte[] getBytes() //字符串转换为字节数组。 

char[] toCharArray() //把字符串转换为字符数组。

static String valueOf(char[] chs) //把字符数组转成字符串。

static String valueOf(int i) //把int类型的数据转成字符串。注意：String类的valueOf方法可以把任意类型的数据转成字符串。

String toLowerCase() //把字符串转成小写。

String toUpperCase() //把字符串转成大写。

String concat(String str) //把字符串拼接。
```

使用示例：

```java
public class StringDemo5 {
    public static void main(String[] args) {
        // 定义一个字符串对象
        String s = "JavaSE";

        // byte[] getBytes():把字符串转换为字节数组。
        byte[] bys = s.getBytes();
        for (int x = 0; x < bys.length; x++) {
            System.out.println(bys[x]);//74 97 118 97 83 69
        }
        System.out.println("----------------");

        // char[] toCharArray():把字符串转换为字符数组。
        char[] chs = s.toCharArray();
        for (int x = 0; x < chs.length; x++) {
            System.out.println(chs[x]);//J a v a S E
        }
        System.out.println("----------------");

        // static String valueOf(char[] chs):把字符数组转成字符串。
        String ss = String.valueOf(chs);
        System.out.println(ss);//JavaSE
        System.out.println("----------------");

        // static String valueOf(int i):把int类型的数据转成字符串。
        int i = 100;
        String sss = String.valueOf(i);
        System.out.println(sss);//100 是字符串
        System.out.println("----------------");

        // String toLowerCase():把字符串转成小写。
        System.out.println("toLowerCase:" + s.toLowerCase());//javase
        System.out.println("s:" + s);
        // System.out.println("----------------");
        // String toUpperCase():把字符串转成大写。
        System.out.println("toUpperCase:" + s.toUpperCase());//JAVASE
        System.out.println("----------------");

        // String concat(String str):把字符串拼接。
        String s1 = "hello";
        String s2 = "world";
        String s3 = s1 + s2;
        String s4 = s1.concat(s2);
        System.out.println("s3:"+s3);//helloworld
        System.out.println("s4:"+s4);//helloworld
    }
}
```

练习：将一个字符串的首字母转成大写，其余为小写。(只考虑英文大小写字母字符)

```java
/**
 * 需求：把一个字符串的首字母转成大写，其余为小写。(只考虑英文大小写字母字符)
 * 举例：
 * 		helloWORLD
 * 结果：
 * 		Helloworld
 *
 * 		A:先获取第一个字符
 * 		B:获取除了第一个字符以外的字符
 * 		C:把A转成大写
 * 		D:把B转成小写
 * 		E:C拼接D
 */
public class StringTest {
    public static void main(String[] args) {
        String str="helloWORLD";
        str=str.substring(0,1).toUpperCase().
                concat(str.substring(1).toLowerCase());
        System.out.println(str);
    }
}
```

### 其他功能

```java
//替换功能：
String replace(char old,char new)

String replace(String old,String new)

//去除字符串两端空格
String trim()

//按字典顺序比较两个字符串
int compareTo(String str)
int compareToIgnoreCase(String str)
```

compareTo() 方法源码解析：

```java
private final char value[]; //字符串会自动转换为一个字符数组。
/**
* 举例
  String s6 = "hello";
  String s9 = "xyz";
  System.out.println(s6.compareTo(s9));// -16
*/
public int compareTo(String anotherString) {
    int len1 = value.length;
    int len2 = anotherString.value.length;

    int lim = Math.min(len1, len2); //lim=3
    char v1[] = value; //v1[h e l l o]
    char v2[] = anotherString.value;//v2[x y z]

    int k = 0;
    while (k < lim) {
        char c1 = v1[k];//h
        char c2 = v2[k];//x
        if (c1 != c2) { //先比较的是对应的字符是否相等
            return c1 - c2; //h=104  x=120   104-120=-16
        }
        k++;
    }
    //如果在指定位置，字符都相同，则长度相减
    return len1 - len2;
}
```

### 练习

练习1：把数组中的数据按照指定个格式拼接成一个字符串。举例：int[] arr = {1,2,3};输出结果：[1, 2, 3]

```java
/**
 *
 * 需求：把数组中的数据按照指定个格式拼接成一个字符串
 * 举例：
 * 		int[] arr = {1,2,3};
 * 输出结果：
 *		"[1, 2, 3]"
 * 分析：
 * 		A:定义一个字符串对象，只不过内容为空
 * 		B:先把字符串拼接一个"["
 * 		C:遍历int数组，得到每一个元素
 * 		D:先判断该元素是否为最后一个
 * 			是：就直接拼接元素和"]"
 * 			不是：就拼接元素和逗号以及空格
 * 		E:输出拼接后的字符串
 */
public class StringTest2 {
    public static void main(String[] args) {
        int[] arr={1,2,3,4};
        System.out.println(arrayToString(arr));
    }

    /**
     * 把数组中的数据按照指定个格式拼接成一个字符串
     */
    public static String arrayToString(int[] arr){
        String str="[";
        for(int i=0;i<arr.length;i++){
            if(i==arr.length-1){
                str+=arr[i];
            }else{
                str+=arr[i]+",";
            }
        }
        str+="]";
        return str;
    }
}
```

练习2：统计大串中小串出现的次数

```java
/**
 * 统计大串中小串出现的次数
 * 举例：
 * 		在字符串"woaijavawozhenaijavawozhendeaijavawozhendehenaijavaxinbuxinwoaijavagun"
 * 结果：
 * 		java出现了5次
 *
 * 分析：
 * 		前提：是已经知道了大串和小串。
 * 		A:定义一个统计变量，初始化值是0
 * 		B:先在大串中查找一次小串第一次出现的位置 ( boolean contains(String str):判断大字符串中是否包含小字符串)
 * 			a:索引是-1，说明不存在了，就返回统计变量
 * 			b:索引不是-1，说明存在，统计变量++
 * 		C:把刚才的索引+小串的长度作为开始位置截取上一次的大串，返回一个新的字符串，并把该字符串的值重新赋值给大串
 * 		D:回到B
 */
public class StringTest3 {
    public static void main(String[] args) {
        String s1="woaijavawozhenaijavawozhendeaijavawozhendehenaijavaxinbuxinwoaijavagun";
        String s2="java";
        System.out.println(smallCount(s2,s1));
    }

    // 统计大串中小串出现的次数
    public static int smallCount(String smallStr,String bigStr) {
        int count=0;

        int index=bigStr.indexOf(smallStr);
        while(index!=-1){
            count++;
            //把刚才的索引+小串的长度作为开始位置截取上一次的大串
            int startIndex=index+smallStr.length();
            bigStr=bigStr.substring(startIndex);
            index=bigStr.indexOf(smallStr);
        }
        return count;
    }
}
```



## StringBuffer

### 常用成员方法

```java
//StirngBuffer的构造方法
public StringBuffer() //无参构造方法

public StringBuffer(int capacity) //指定容量的字符串缓冲区对象

public StringBuffer(String str) //指定字符串内容的字符串缓冲区对象

//StringBuffer的方法
public int capacity() //返回当前容量。	理论值

public int length() //返回长度（字符数）。 实际值
```

使用示例：

```java
public static void main(String[] args) {
    // public StringBuffer():无参构造方法
    StringBuffer sb = new StringBuffer();
    System.out.println("sb:" + sb);
    System.out.println("sb.capacity():" + sb.capacity()); //StringBuffer默认的指定容量是16
    System.out.println("sb.length():" + sb.length());//0
    System.out.println("--------------------------");

    // public StringBuffer(int capacity):指定容量的字符串缓冲区对象
    StringBuffer sb2 = new StringBuffer(50);
    System.out.println("sb2:" + sb2);//""
    System.out.println("sb2.capacity():" + sb2.capacity());//50
    System.out.println("sb2.length():" + sb2.length());//0
    System.out.println("--------------------------");

    // public StringBuffer(String str):指定字符串内容的字符串缓冲区对象
    StringBuffer sb3 = new StringBuffer("hello");
    System.out.println("sb3:" + sb3);//hello
    System.out.println("sb3.capacity():" + sb3.capacity());//16+5=21
    System.out.println("sb3.length():" + sb3.length());//5
}
```

### 添加功能

```java
public StringBuffer append(String str) 
//可以把任意类型数据添加到字符串缓冲区里面,并返回字符串缓冲区本身

public StringBuffer insert(int offset,String str) 
//在指定位置把任意类型的数据插入到字符串缓冲区里面,并返回字符串缓冲区本身
```

使用示例：

```java
public static void main(String[] args) {
    // 创建字符串缓冲区对象
    StringBuffer sb = new StringBuffer();
    
    // 链式编程
    sb.append("hello").append(true).append(12).append(34.56);
    System.out.println("sb:" + sb);

    // public StringBuffer insert(int offset,String
    // str):在指定位置把任意类型的数据插入到字符串缓冲区里面,并返回字符串缓冲区本身
    sb.insert(5, "world");
    System.out.println("sb:" + sb);
}
```

### 删除功能

```java
public StringBuffer deleteCharAt(int index) 
//删除指定位置的字符，并返回本身

public StringBuffer delete(int start,int end) 
//删除从指定位置开始指定位置结束的内容，并返回本身
```

使用示例：

```java
public static void main(String[] args) {
    // 创建对象
    StringBuffer sb = new StringBuffer();

    // 添加功能
    sb.append("hello").append("world").append("java");

    // public StringBuffer deleteCharAt(int index):删除指定位置的字符，并返回本身
    // 需求：我要删除e这个字符，肿么办?
    //sb.deleteCharAt(1);
    //public StringBuffer delete(int start,int end):删除从指定位置开始指定位置结束的内容，并返回本身
    // 需求：我要删除world这个字符串，肿么办?
    //sb.delete(5, 10);
    //需求:删除所有字符
    sb.delete(0,sb.length());
    System.out.println("sb:" + sb);
}
```

### 替换功能

```java
public StringBuffer replace(int start,int end,String str) 
//从start开始到end用str替换
```

使用示例：

```java
public static void main(String[] args) {
    // 创建字符串缓冲区对象
    StringBuffer sb = new StringBuffer();

    // 添加数据
    sb.append("hello");
    sb.append("world");
    sb.append("java");
    System.out.println("sb:" + sb);

    // public StringBuffer replace(int start,int end,String str):从start开始到end用str替换
    // 需求：我要把world这个数据替换为"节日快乐"
    sb.replace(5,10,"节日快乐");
    System.out.println("sb："+sb);
}
```

### 反转功能

```java
public StringBuffer reverse()
```

使用示例：

```java
public static void main(String[] args) {
    // 创建字符串缓冲区对象
    StringBuffer sb = new StringBuffer();

    // 添加数据
    sb.append("霞青林爱我");
    System.out.println("sb:" + sb);//霞青林爱我

    // public StringBuffer reverse()
    sb.reverse();
    System.out.println("sb:" + sb);//我爱林青霞
}
```

### 截取功能

```java
public String substring(int start) // 注意截取返回的是String,而不是StringBuffer了

public String substring(int start,int end)
```

使用示例：

```java
public static void main(String[] args) {
    // 创建字符串缓冲区对象
    StringBuffer sb = new StringBuffer();

    // 添加元素
    sb.append("hello").append("world").append("java");
    System.out.println("sb:" + sb);//sb:helloworldjava

    // 截取功能
    // public String substring(int start)
    String s = sb.substring(5);
    System.out.println("s:" + s);//s:worldjava
    System.out.println("sb:" + sb);//sb:helloworldjava

    // public String substring(int start,int end)
    String ss = sb.substring(5, 10);//左闭右开
    System.out.println("ss:" + ss);//ss:world
    System.out.println("sb:" + sb);//sb:helloworldjava
}
```



## String 和 StringBuffer 的相互转换

```java
public class StringBufferDemo7 {
    public static void main(String[] args) {
        String s="hello";
        System.out.println(stringToStringBuffer(s)); //hello
        System.out.println(stringBufferToString(stringToStringBuffer(s)));//hello
    }

    // String -->StringBuffer
    public static StringBuffer stringToStringBuffer(String s) {
        // 注意：不能把字符串的值直接赋值给StringBuffer
        // StringBuffer sb = "hello";
        // StringBuffer sb = s;

        // 方式1:通过构造方法
      /*  StringBuffer sb = new StringBuffer(s);
        return sb;*/
        // 方式2：通过append()方法
        StringBuffer sb2 = new StringBuffer();
        sb2.append(s);
        return sb2;
    }

    //StringBuffer -->String
    public static String stringBufferToString(StringBuffer buffer){
        // 方式1:通过构造方法
       /* String str = new String(buffer);
        return str;*/
        // 方式2：通过toString()方法
        String str2 = buffer.toString();
        return str2;
    }
}
```

### 练习

练习1：将数组拼接成一个字符串

```java
public class StringBufferTest {
    public static void main(String[] args) {
        int[] arr={1,2,3,4};
        System.out.println(arrToString(arr));//[1,2,3,4]
    }

    public static String arrToString(int[] arr){
        StringBuffer buffer=new StringBuffer();
        buffer.append("[");
        for(int i=0;i<arr.length;i++){
            if(i==arr.length-1){
                buffer.append(arr[i]);
            }else{
                buffer.append(arr[i]).append(",");
            }
        }
        buffer.append("]");
        return buffer.toString();
    }
}
```

练习2：判断一个字符串是否是对称字符串

```java
/**
 *  判断一个字符串是否是对称字符串
 * 例如"abc"不是对称字符串，"aba"、"abba"、"aaa"、"mnanm"是对称字符串
 *
 * 分析：
 * 		判断一个字符串是否是对称的字符串，我只需要把
 * 			第一个和最后一个比较
 * 			第二个和倒数第二个比较
 * 			...
 * 		比较的次数是长度除以2。
 */
public class StringBufferTest2 {
    public static void main(String[] args) {
        System.out.println(isSymmetry("aba")); //true
        System.out.println(isSymmetry2("aabbaa"));//true
    }

    //通过字符数组的方式来比较
    public static boolean isSymmetry(String s){
        boolean flag=true;
        char[] chs=s.toCharArray();
        int len=chs.length;
        for(int start=0,end=chs.length-1;start<=end;start++,end--){
            if(chs[start]!=chs[end]){
                flag=false;
                break;
            }
        }
        return flag;
    }

     //通过StringBuffer的reverse
    public static boolean isSymmetry2(String s){
        //通过StringBuffer的reverse获取反转字符串s2
        String s2=new StringBuffer(s).reverse().toString();
        return s2.equals(s);
    }
}
```

练习3：看程序写结果

```java
public class StringBufferTest3 {
    public static void main(String[] args) {
        String s1 = "hello";
        String s2 = "world";
        System.out.println(s1 + "---" + s2);
        change(s1, s2);
        System.out.println(s1 + "---" + s2);

        StringBuffer sb1 = new StringBuffer("hello");
        StringBuffer sb2 = new StringBuffer("world");
        System.out.println(sb1 + "---" + sb2);
        change(sb1, sb2);
        System.out.println(sb1 + "---" + sb2);
    }

    //StringBuffer作为参数传递
    public static void change(StringBuffer sb1, StringBuffer sb2) {
        sb1 = sb2;
        sb2.append(sb1);
    }

    //String作为参数传递
    public static void change(String s1, String s2) {
        s1 = s2;
        s2 = s1 + s2;
    }
}
```

输出结果：

```html
hello---world
hello---world
hello---world
hello---worldworld
```

String作为参数传递，效果和基本类型作为参数传递是一样的。

