https://www.cnblogs.com/Qian123/p/5710533.html

## 一、问题引入

```java
对于基本类型 
int a = 10 ; int b = a ; //这是拷贝一份值给b，两者之后互不相干。
对于引用类型
myDemo1 = myDemo2 ;   //这是将复制一个栈上引用给 myDemo1 ，理论上还是引用到堆上的同一个对象。
					// myDemo1 和 myDemo2 的修改是共享的。那么如果要拷贝一份一模一样的对象，
					//可以考虑克隆和序列化。
```



## 二、为什么要克隆

```
答案是：克隆的对象可能包含一些已经修改过的属性，而new出来的对象的属性都还是初始化时候的值，所以当需要一个新的对象来保存当前对象的“状态”就靠clone方法了。那么我把这个对象的临时属性一个一个的赋值给我新new的对象不也行嘛？可以是可以，但是一来麻烦不说，二来，大家通过上面的源码都发现了clone是一个native方法，就是快啊，在底层实现的。
	提个醒，我们常见的Object a=new Object();Object b;b=a;这种形式的代码复制的是引用，即对象在内存中的地址，a和b对象仍然指向了同一个对象。
　　而通过clone方法赋值的对象跟原来的对象时同时独立存在的。
```



## 三、克隆基础

​	一个对象想要实现克隆功能，就需要实现 Cloneable 接口，这是一个和Seriablizable一样的空接口，作为标识用。不实现接口克隆时会报错。

​	**接下来就是覆盖 clone(), 访问权限为protected ， 方法中调 super.clone()方法得到需要的复制对象。（native方法），clone()方法是Object自带的**

​	例子：

```java
/**
 * @author: lingz
 * @Date: 2019/4/28 19:47
 */
public class CloneDemo implements Cloneable {

    private int number ;

    public int getNumber(){
        return number ;
    }

    public void setNumber(int number){
        this.number = number ;
    }

    @Override
    protected Object clone() throws CloneNotSupportedException {

        CloneDemo cloneDemo = null ;
        try{
            cloneDemo = (CloneDemo) super.clone() ;
        }catch (CloneNotSupportedException e){
            e.printStackTrace();
        }
        //其实这一步就会直接返回克隆的结果
        //return super.clone();
        return cloneDemo ;
    }

    public static void main(String[] args) throws CloneNotSupportedException {
        CloneDemo cloneDemo1 = new CloneDemo() ;
        cloneDemo1.setNumber(10);
        CloneDemo cloneDemo2 = (CloneDemo) cloneDemo1.clone() ;
        cloneDemo2.setNumber(20);
        System.out.println("cloneDemo1 ：" + cloneDemo1.getNumber());
        System.out.println("cloneDemo2 ：" + cloneDemo2.getNumber());
    }
}
```



## 四、浅克隆 和 深克隆

### 1、浅克隆

​	上面例子中，成员变量时一个基本类型，所以得到了值。如果是一个引用类型，则只是简单地复制了引用，仍共享一个堆上的变量。这就是浅克隆。

### 2、深克隆

​	深克隆，就是即使成员变量时引用类型，也一起拷贝了。例子：

```java
/**
*	引用类型也实现克隆接口，重写clone方法
*/
class Address implements Cloneable {  
    private String add;  
  
    public String getAdd() {  
        return add;  
    }  
  
    public void setAdd(String add) {  
        this.add = add;  
    }  
      
    @Override  
    public Object clone() {  
        Address addr = null;  
        try{  
            addr = (Address)super.clone();  
        }catch(CloneNotSupportedException e) {  
            e.printStackTrace();  
        }  
        return addr;  
    }  
}  
  
class Student implements Cloneable{  
    private int number;  
  
    private Address addr;  
      
    public Address getAddr() {  
        return addr;  
    }  
  
    public void setAddr(Address addr) {  
        this.addr = addr;  
    }  
  
    public int getNumber() {  
        return number;  
    }  
  
    public void setNumber(int number) {  
        this.number = number;  
    }  
      
    @Override  
    public Object clone() {  
        Student stu = null;  
        try{  
            stu = (Student)super.clone();   //浅复制  
        }catch(CloneNotSupportedException e) {  
            e.printStackTrace();  
        }  
        stu.addr = (Address)addr.clone();   //关键！！！深度复制  
        return stu;  
    }  
}  
public class Test {  
      
    public static void main(String args[]) {  
          
        Address addr = new Address();  
        addr.setAdd("杭州市");  
        Student stu1 = new Student();  
        stu1.setNumber(123);  
        stu1.setAddr(addr);  
          
        Student stu2 = (Student)stu1.clone();  
          
        System.out.println("学生1:" + stu1.getNumber() + ",地址:" + stu1.getAddr().getAdd());  
        System.out.println("学生2:" + stu2.getNumber() + ",地址:" + stu2.getAddr().getAdd());  
          
        addr.setAdd("西湖区");  
          
        System.out.println("学生1:" + stu1.getNumber() + ",地址:" + stu1.getAddr().getAdd());  
        System.out.println("学生2:" + stu2.getNumber() + ",地址:" + stu2.getAddr().getAdd());  
    }  
}
```

​	

## 五、克隆和序列化

​	深克隆能完整克隆一个对象，但是可以看出还是比较繁琐的，需要手动写每个引用实例的克隆。因此，我们可以通过序列化来直接拷贝一个对象。先序列化一个对象，后再反序列化得到一个对象，这个对象的状态和原先的可以按照意愿一模一样，而且简单。

## 六、总结

实现对象克隆有两种方式：

  1). 实现Cloneable接口并重写Object类中的clone()方法；

  2). 实现Serializable接口，通过对象的序列化和反序列化实现克隆，可以实现真正的深度克隆。

**注意：**基于序列化和反序列化实现的克隆不仅仅是深度克隆，更重要的是通过泛型限定，可以检查出要克隆的对象是否支持序列化，这项检查是编译器完成的，不是在运行时抛出异常，这种是方案明显优于使用Object类的clone方法克隆对象。让问题在编译的时候暴露出来总是优于把问题留到运行时。