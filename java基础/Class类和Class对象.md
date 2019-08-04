https://www.cnblogs.com/lzq198754/p/5780331.html

##Class类和Class对象

###1、RTTI 

​	Run-Time Type Identification，运行时类型识别。

​	这就提到Java的类的类型：编译时类型 和 运行时类型

###2、Class类和 Class对象

```java
- 存于java.lang包中。
- Class类用来表示运行时类型信息，Class类创建出Class对象，一个类对应一个Class对象，
- 为什么需要class对象：
	 new一个对象或引用静态成员变量时，JVM的类加载器子系统就加载对应Class对象,根据其提	  	   供的类型信息创建对象或静态变量引用值。
- Class对象存于对应类的.class文件中。
- 每个类对应只有一个Class对象，Class类全是私有的构造函数，所以需要JVM加载。
- 反射机制就是建立于Class对象能在运行时提供类型信息的基础
```

###3、Class对象的加载

​	Class对象的加载，可以理解为类的加载初始化时机，后者是将类的字节码文件加载到虚拟机中，而Class对象即在字节码文件中，所以，这里可以参看类的初始化时机：

```java
（1）new一个对象；

	读取或设置一个类的静态字段（除了final修饰、编译期把结果放入常量池的静态字段外）；

	读取或设置一个类的静态方法 ；

（2）通过java.lang.reflect包的方法对类进行反射调用，若类无初始化，则先触发初始化；

（3）初始化一个类的时候，若父类没有初始化，则先初始化；

（4）虚拟机启动时，用户需要指定一个要执行的主类，虚拟机回先初始化这个主类。

（5）当使用JDK 1.7的动态语言支持时，如果一个java.lang.invoke.MethodHandle实例最后的	解析结果REF_getStatic、REF_putStatic、REF_invokeStatic的方法句柄，则需要先触发	这个方法句柄所对应的类的初始化。
```



### 4、Class对象获取方式

#### ① Class.forName()

​	- Class.forName()方法的调用将会返回一个对应类的Class对象。

​	- 这样的好处是不需通过该类的实例对象引用去获取Class对象。但是**会初始化类**。

​	- 可能抛异常，因为在编译期不能确定类是否存在。

#### ② obj.getClass() 

​	- 通过对象获取，自然是要初始化类。

#### ③ class字面量

 - 在编译期就会受到检查，所以不同抛异常。

 - 这种方法不论是类、接口、数组、**基本数据类型**都可以。

   其中，基本数据类型：

   ```java
   boolean.class = Boolean.TYPE;
   char.class = Character.TYPE;
   byte.class = Byte.TYPE;
   short.class = Short.TYPE;
   int.class = Integer.TYPE;
   long.class = Long.TYPE;
   float.class = Float.TYPE;
   double.class = Double.TYPE;
   void.class = Void.TYPE;
   ```

- 这种方式不会导致类的初始化，因为在类的初始化过程的最开始：加载阶段，就可以获取class字面量了。

### 5、理解泛化的Class对象引用



## 反射

### 1、概念

​	在运行状态中，对于任意一个类，能够知道该类的所有属性和方法；对于任一个对象，都能调用它的任意一个属性和方法。这种动态获取信息和动态调用对象的方法就是Java的反射机制。



#### Constructor类

​	Constructor类存在于反射包(java.lang.reflect)中，反映的是Class 对象所表示的**类的构造方法**。获取Constructor对象是通过Class类中的方法获取的，Class类与Constructor相关的主要方法如下：

| 方法返回值         | 方法名称                                             | 方法说明                                                  |
| ------------------ | ---------------------------------------------------- | --------------------------------------------------------- |
| `static Class<?>`  | forName(String className)                            | 返回与带有给定字符串名的类或接口相关联的 Class 对象。     |
| `Constructor<T>`   | `getConstructor(Class<?>... parameterTypes)`         | 返回指定参数类型、具有public访问权限的构造函数对象        |
| `Constructor<?>[]` | getConstructors()                                    | 返回所有具有public访问权限的构造函数的Constructor对象数组 |
| `Constructor<T>`   | `getDeclaredConstructor(Class<?>... parameterTypes)` | 返回指定参数类型、所有声明的（包括private）构造函数对象   |
| `Constructor<?>[]` | `getDeclaredConstructor()`                           | 返回所有声明的（包括private）构造函数对象                 |
| T                  | newInstance()                                        | 创建此 Class 对象所表示的类的一个新实例。                 |





#### Field类

​	Field提供有关类或接口的单个字段的信息，以及对他的动态访问权限。反射的字段可能是一个类（静态）字段或实例字段。同样的道理，我们可以通过Class类的提供的方法来获取代表字段信息的Field对象，Class类与Field对象相关方法如下：

| 方法返回值 | 方法名称                        | 方法说明                                                     |
| ---------- | ------------------------------- | ------------------------------------------------------------ |
| `Field`    | `getDeclaredField(String name)` | 获取指定name名称的(包含private修饰的)字段，不包括继承的字段  |
| `Field[]`  | `getDeclaredField()`            | 获取Class对象所表示的类或接口的所有(包含private修饰的)字段,不包括继承的字段 |
| `Field`    | `getField(String name)`         | 获取指定name名称、具有public修饰的字段，**包含继承字段**     |
| `Field[]`  | `getField()`                    | 获取修饰符为public的字段，包含继承字段                       |





#### Method类

​	Method 提供关于类或接口上单独某个方法（以及如何访问该方法）的信息，所反映的方法可能是类方法或实例方法（包括抽象方法）。下面是Class类获取Method对象相关的方法：

| 方法返回值 | 方法名称                                                     | 方法说明                                                     |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `Method`   | `getDeclaredMethod(String name, Class<?>... parameterTypes)` | 返回一个指定参数的Method对象，该对象反映此 Class 对象所表示的类或接口的指定已声明方法。 |
| `Method[]` | `getDeclaredMethod()`                                        | 返回 Method 对象的一个数组，这些对象反映此 Class 对象表示的类或接口声明的所有方法，包括公共、保护、默认（包）访问和私有方法，但不包括继承的方法。 |
| `Method`   | `getMethod(String name, Class<?>... parameterTypes)`         | 返回一个 Method 对象，它反映此 Class 对象所表示的类或接口的指定公共成员方法。 |
| `Method[]` | `getMethods()`                                               | 返回一个包含某些 Method 对象的数组，这些对象反映此 Class 对象所表示的类或接口（包括那些由该类或接口声明的以及从**超类**和超接口继承的那些的类或接口）的公共 member 方法。 |





#### 反射包中的Array类

​	在Java的java.lang.reflect包中存在着一个可以动态操作数组的类，Array，它提供了动态创建和访问 Java 数组的方法。Array 允许在执行 get 或 set 操作进行取值和赋值。在Class类中与数组关联的方法是：

| 方法返回值 | 方法名称             | 方法说明                                   |
| ---------- | -------------------- | ------------------------------------------ |
| `Class<?>` | `getComponentType()` | 返回表示数组元素类型的 Class，即数组的类型 |
| `boolean`  | `isArray()`          | 判定此 Class 对象是否表示一个数组类。      |

| 方法返回值      | 方法名称                                                 | 方法说明                                       |
| --------------- | -------------------------------------------------------- | ---------------------------------------------- |
| `static Object` | set(Object array, int index)                             | 返回指定数组对象中索引组件的值。               |
| `static int`    | `getLength(Object array)`                                | 以 int 形式返回指定数组对象的长度              |
| `static object` | `newInstance(Class<?> componentType, int... dimensions)` | 创建一个具有指定类型和维度的新数组。           |
| `static Object` | `newInstance(Class<?> componentType, int length)`        | 创建一个具有指定的组件类型和长度的新数组。     |
| `static void`   | `set(Object array, int index, Object value)`             | 将指定数组对象中索引组件的值设置为指定的新值。 |