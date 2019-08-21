

https://github.com/nnngu/LearningNotes/blob/master/Java%20Basis/007%20Java%E7%9A%84%E5%8F%82%E6%95%B0%E4%BC%A0%E9%80%92%E6%98%AF%E5%80%BC%E4%BC%A0%E9%80%92%E8%BF%98%E6%98%AF%E5%BC%95%E7%94%A8%E4%BC%A0%E9%80%92.md

## Java引用传递和值传递

- Java没有引用传递
- 值传递，对于数值类型，是拷贝一个数值供传递；对于引用类型，是拷贝一个引用传递，这个引用也是指向相同堆的位置，所以能够修改对象。所以如果将这个引用赋给其他堆地址（创新对象），那么，原来的对象没有任何改变，只是这个参数引用指向了一个新对象，看下面例子
- 引用传递，真正的引用传递，就真的把这个引用真真实实传给方法，修改引用，指向新对象，那么原来的对象就失去这个引用了。



```java

/*
	基本类型传递，数值互不相干， num还是1，x=2
*/
public class TransferTest {
    public static void main(String[] args) {
        int num = 1;
        System.out.println("changeNum()方法调用之前：num = " + num);
        changeNum(num);
        System.out.println("changeNum()方法调用之后：num = " + num);
    }

    public static void changeNum(int x) {
        x = 2;
    }
}

/*
	这也是值传递，
	方法调用之后，产生了一个新对象，
	原本 person = new Person()
	和新的 p = new Person()
	p只不过该开始和 person执行同一个地方，这是值传递。
	引用传递，则 p就是 person，p指别的，就是person指别的。
*/

public class TransferTest2 {
      public static void main(String[] args) {
          Person person = new Person();
          System.out.println(person);
          change(person);
          System.out.println(person);
      }
  
      public static void change(Person p) {
         p = new Person();
     }
 }
 
 /**
  * Person类
  */
 class Person {
 
 }




```

