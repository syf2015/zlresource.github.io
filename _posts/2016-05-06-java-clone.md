---
layout: post
title:  "java clone(java深浅克隆)分析"
date:  2016-05-06
categories: JAVA
---

java clone(java深浅克隆)分析

---

- 目录
  {:toc}

#### 引用变量与普通变量的区别

Java语言的一个优点就是取消了指针的概念。但是，复制时，存在引用变量与普通变量的区别。
其中引用变量，容易出现深复制，与浅复制的问题。
1> 浅复制：是指复制只复制变量的引用，即clone的两个变量指向同一个区域。
2> 深复制：指的是复制时，将原来的变量所指的区域复制，即clone出来的变量，与原来各自指向了不同内存区。
由于浅复制，新变量与原变量指向同一个内存区，修改其中一个变量内存区域的值，会影响另一个。而，深复制，新旧变量完全相互独立。

#### 一、例外：

String 为引用变量,而String没有实现cloneable接口
因为String是在内存中不可以被改变的对象。修改时，会新分配一块内存。就和原来的那块分离了。所以深浅clone不需要考虑String

#### 二、克隆实现：

1 > 完全自己编写克隆方法：直接new 对象，效率不高！！！（因为是native方法，贴近硬件，可能丧失可移植性）
​      
2> 用java为我们准备的：
 0) Object类中： jdk中为我们准备了用native修饰的clone方法

    protected native Object clone() throws CloneNotSupportedException;

为啥是protect类型？：
1. 保护机制：只有自己的子类才可使用clone()--必须继承Object类。
2. 防止子类重载clone()方法后，变成public。
   如下： Student调用Student的，Pet调用Pet的,各调各的clone方法


              Student stu = (Student) super.clone();
          		stu.pet = (Pet) stu.pet.clone();	 

1)  Cloneable 标记接口  java.lang.Cloneable
​           
2) 若clone的类未实现标记接口Cloneable 会抛异常 java.lang.CloneNotSupportedException
​    

#### 三、深克隆例子：

① 继承Cloneable  -->  ② 重写clone()方法

    class Pet implements Cloneable{
    //变量 get set.... 
            	
      @Override 
      protected Object clone() throws CloneNotSupportedException {		
        return super.clone();    //调用 父类Object中的clone()方法
      }
    }
    class Student implements Cloneable{
       //引用变量Person,存在深浅复制问题
      private Pet p;  
      ...get set .....
            
      @Override
      protected Object clone() throws CloneNotSupportedException {
        Student s = (Student) super.clone();
        s.p = (Pet) s.p.clone();		//深复制，需要调用该引用变量自己的clone()
        return s;
      }
    }
