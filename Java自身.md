# 接口

## 1、 可拥有的字段

* 默认都是静态常量修饰  public static final 

## 2、可拥有的方法

### （1）默认方法

* default 修饰，可重写，可重载
* 可以在不破坏原有的实现的情况下扩展功能
* 如果两个接口中定义了一模一样的默认方法，并且一个实现类同时实现了这两个接口，那么必须在实现类中重写默认方法，否则编译失败。
* 通过实现类调用接口的默认方法

### (2) 静态方法

* 接口的静态方法必须提供实现

### （3）抽象方法

* 不提供修饰符的都是抽象方法，不提供实现，不需要显示提供修饰符



# 抽象类

## 1、可拥有的方法

### （1） 抽象方法

* 和接口不同的是，抽象方法的需要显示表明abstract

### （2）具体方法



