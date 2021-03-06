##  C++的精髓——虚函数 ##

虚函数为了重载和多态的需要，在基类中是由定义的，即便定义是空，所以子类中可以重写也可以不写基类中的函数！

在某基类中声明为 virtual 并在一个或多个派生类中被重新定义的成员函数，用法格式为：virtual 函数返回类型 函数名（参数表） {函数体}；实现多态性，通过指向派生类的基类指针或引用，访问派生类中同名覆盖成员函数

纯虚函数在基类中是没有定义的，必须在子类中加以实现，很像java中的接口函数！

### 虚函数 ###

引入原因：为了方便使用多态特性，我们常常需要在基类中定义虚函数。

```cpp
class Cman

{

public:

virtual void Eat(){……};

void Move();

private:

};

class CChild : public CMan

{

public:

virtual void Eat(){……};

private:

};

CMan m_man;

CChild m_child;
//这才是使用的精髓，如果不定义基类的指针去使用，没有太大的意义
CMan *p ;

p = &m_man ;

p->Eat(); //始终调用CMan的Eat成员函数，不会调用 CChild 的

p = &m_child;

p->Eat(); //如果子类实现(覆盖)了该方法，则始终调用CChild的Eat函数

//不会调用CMan 的 Eat 方法；如果子类没有实现该函数，则调用CMan的Eat函数

p->Move(); //子类中没有该成员函数，所以调用的是基类中的

```

### 纯虚函数 ###

纯虚函数是一种特殊的虚函数，它的一般格式如下：
```cpp
class <类名>
{
virtual <类型><函数名>(<参数表>)=0;
…
};
```
引入原因：

1、同“虚函数”；

2、在很多情况下，基类本身生成对象是不合情理的。例如，动物作为一个基类可以派生出老虎、孔雀等子类，但动物本身生成对象明显不合常理。

//纯虚函数就是基类只定义了函数体，没有实现过程定义方法如下

// virtual void Eat() = 0; 直接=0 不要 在cpp中定义就可以了

//纯虚函数相当于接口，不能直接实例话，需要派生类来实现函数定义

//有的人可能在想，定义这些有什么用啊 ，我觉得很有用

//比如你想描述一些事物的属性给别人，而自己不想去实现，就可以定

//义为纯虚函数。说的再透彻一些。比如盖楼房，你是老板，你给建筑公司

//描述清楚你的楼房的特性，多少层，楼顶要有个花园什么的

//建筑公司就可以按照你的方法去实现了，如果你不说清楚这些，可能建筑

//公司不太了解你需要楼房的特性。用纯需函数就可以很好的分工合作了

### 虚函数和纯虚函数区别 ###
#### 观点一： ####

类里声明为虚函数的话,这个函数是实现的，哪怕是空实现，它的作用就是为了能让这个函数在它的子类里面可以被重载，这样的话，这样编译器就可以使用后期绑定来达到多态了

纯虚函数只是一个接口，是个函数的声明而已，它要留到子类里去实现。
```cpp
class A{

protected:

void foo();//普通类函数

virtual void foo1();//虚函数

virtual void foo2() = 0;//纯虚函数

}
```

#### 观点二：####

虚函数在子类里面也可以不重载的；但纯虚必须在子类去实现，这就像Java的接口一样。通常我们把很多函数加上virtual，是一个好的习惯，虽然牺牲了一些性能，但是增加了面向对象的多态性，因为你很难预料到父类里面的这个函数不在子类里面不去修改它的实现

#### 观点三：####

虚函数的类用于“实作继承”，继承接口的同时也继承了父类的实现。当然我们也可以完成自己的实现。纯虚函数的类用于“介面继承”，主要用于通信协议方面。关注的是接口的统一性，实现由子类完成。一般来说，介面类中只有纯虚函数的。

#### 观点四：####

**错误：**带纯虚函数的类叫虚基类，这种基类不能直接生成对象，而只有被继承，并重写其虚函数后，才能使用。这样的类也叫抽象类。

虚函数是为了继承接口和默认行为

纯虚函数只是继承接口，行为必须重新定义
////////////////////////////////////////////////////////////////////////////////////
虚基类的初始化　　虚基类的初始化与一般多继承的初始化在语法上是一样的,但构造函数的调用次序不同.
　　派生类构造函数的调用次序有三个原则:
　　(1)虚基类的构造函数在非虚基类之前调用;
　　(2)若同一层次中包含多个虚基类,这些虚基类的构造函  虚基类和非虚基类的区别虚基类和非虚基类的区别数按它们说明的次序调用;
　　(3)若虚基类由非虚基类派生而来,则仍先调用基类构造函数,再调用派生类的构造函数.
编辑本段
C++的虚基类　　在派生类继承基类时，加上一个virtual关键词则为虚拟基类继承，如：
```cpp
class derive:virtual public base
{
};
```
　　虚基类主要解决在多重继承时，基类可能被多次继承，虚基类主要提供一个基类给派生类，如：
```cpp
  class B
  {
  };
  class D1:public B
  {
  };
  class D2:public B
  {
  };
  class C:public D1,public D2
  {
  };
```
　　这里C在D1,D2上继承，但有两个基类，造成混乱。因而使用虚基类，即：
```cpp
  classB
  {
  };
  class D1:virtual public B
  {
  };
  class D2:virtual publicB
  {
  };
  class C:public D1,public D2
```

在使用虚基类时要注意：　　

* (1) 一个类可以在一个类族中既被用作虚基类，也被用作非虚基类。
* (2) 在派生类的对象中，同名的虚基类只产生一个虚基类子对象，而某个非虚基类产生各自的子对象。
* (3) 虚基类子对象是由最远派生类的构造函数通过调用虚基类的构造函数进行初始化的。\
* (4) 最远派生类是指在继承结构中建立对象时所指定的类。
* (5) 派生类的构造函数的成员初始化列表中必须列出对虚基类构造函数的调用；如果未列出，则表示使用该虚基类的缺省构造函数。
* (6) 从虚基类直接或间接派生的派生类中的构造函数的成员初始化列表中都要列出对虚基类构造函数的调用。但仅仅用建立对象的最远派生类的构造函数调用虚基类的构造函数，而该派生类的所有基类中列出的对虚基类的构造函数的调用在执行中被忽略，从而保证对虚基类子对象只初始化一次。
* (7) 在一个成员初始化列表中同时出现对虚基类和非虚基类构造函数的调用时，虚基类的构造函数先于非虚基类的构造函数执行。
  静态联编：在程序链接阶段就可以确定的调用。
  动态联编：在程序执行时才能确定的调用。