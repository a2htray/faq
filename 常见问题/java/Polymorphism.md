# Polymorphism 多态

`Polymorphism`可以富裕一个对象多种形态。最常见的场景，使用一个父类类型的变量去引用一个子类对象。

任何一个Java对象，进行`IS-A`判断时，如果使其为真的类型很多，则可以认为对象是多态的。又因为Java对象对`IS-A`判断，使用本身的类型和`Object`类型都为真，故Java中的对象都是多态的。

值得注意的是，Java中只有通过引用类型才能访问到对象。一个引用类型只能属于一个类型，一旦声明，引用变量的类型将不能被改变。

只要没有使用`final`关键字修饰，引用变量可以被重新赋值为另一个对象。引用变量的类型将决定对象所能调用的方法。

一个引用变量可以指向声明的类型或其子类型，类型可以是类，也可以是接口。

## 例子1

下面来看看这个例子：

```java
public interface Vegetarian{}
public class Animal{}
public class Deer extends Animal implements Vegetarian{}
```

现在`Deer`类可以认为是多态的，因为他具有多重的实现。下面的陈述都是真的：

* 一个Deer是一个Animal
* 一个Deer是一个Vegetarian
* 一个Deer是一个Deer
* 一个Deer是一个Object

当我们声明一个引用变量，实际上都是指向Deer对象，下面的声明都是合法的：

## 例子2

```java
Deer d = new Deer();
Animal a = d;
Vegetarian v = d;
Object o = d;
```

所有的引用变量`d a v o`都是指向堆中同一个Deer对象。

## 虚方法

下面在Java中使用覆写方法来体现`多态`的好处。

`覆写`是指一个子类可以覆写其父类中的方法。本质上，一个被覆写的方法将藏在父类中，除非在子类覆写方法中使用`super`，才能对其进行调用。

## 例子3

```java
/* File name : Employee.java */
public class Employee {
   private String name;
   private String address;
   private int number;

   public Employee(String name, String address, int number) {
      System.out.println("Constructing an Employee");
      this.name = name;
      this.address = address;
      this.number = number;
   }

   public void mailCheck() {
      System.out.println("Mailing a check to " + this.name + " " + this.address);
   }

   public String toString() {
      return name + " " + address + " " + number;
   }

   public String getName() {
      return name;
   }

   public String getAddress() {
      return address;
   }

   public void setAddress(String newAddress) {
      address = newAddress;
   }

   public int getNumber() {
      return number;
   }
}
```

现在我们扩展这个类：

```java
/* File name : Salary.java */
public class Salary extends Employee {
   private double salary; // Annual salary
   
   public Salary(String name, String address, int number, double salary) {
      super(name, address, number);
      setSalary(salary);
   }
   
   public void mailCheck() {
      System.out.println("Within mailCheck of Salary class ");
      System.out.println("Mailing check to " + getName()
      + " with salary " + salary);
   }
   
   public double getSalary() {
      return salary;
   }
   
   public void setSalary(double newSalary) {
      if(newSalary >= 0.0) {
         salary = newSalary;
      }
   }
   
   public double computePay() {
      System.out.println("Computing salary pay for " + getName());
      return salary/52;
   }
}
```

现在我们来推断下下面程序的输出：

```java
/* File name : VirtualDemo.java */
public class VirtualDemo {

   public static void main(String [] args) {
      Salary s = new Salary("Mohd Mohtashim", "Ambehta, UP", 3, 3600.00);
      Employee e = new Salary("John Adams", "Boston, MA", 2, 2400.00);
      System.out.println("Call mailCheck using Salary reference --");   
      s.mailCheck();
      System.out.println("\n Call mailCheck using Employee reference--");
      e.mailCheck();
   }
}
```

输出如下：

```bash
Constructing an Employee
Constructing an Employee

Call mailCheck using Salary reference --
Within mailCheck of Salary class
ailing check to Mohd Mohtashim with salary 3600.0

Call mailCheck using Employee reference--
Within mailCheck of Salary class
ailing check to John Adams with salary 2400.0
```

这里我们实例化了两个`Salary`对象，一个类型为`Salary`，一个类型为`Employee`。

当我们调用`s.mailCheck()`，在编译时期，编译器在`Salary`类中发现了该方法，然后在运行时期，`JVM`调用该方法。

变量`e`上的`mailCheck()`就有很大的不同，因为`e`的是一个`Employee`引用。编译器在看到`e.mailCheck`时，他会在`Employee`类中看到该方法。

这里，敲黑板了啦！！！

在编译阶段，编译器使用`Employee`中的`mailCheck()`让代码过了语句的验证，但在运行阶段，`JVM`还是选择调用`Salary`类中的`mailCheck()`方法。

不管变量定义时，其类型为什么，在运行阶段中，覆写后的方法将会被调用

## 出处

tutorialspoint Java Ploymorphism [https://www.tutorialspoint.com/java/java_polymorphism.htm](https://www.tutorialspoint.com/java/java_polymorphism.htm)
