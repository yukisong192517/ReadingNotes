# 混合对象类

## Class Theory

* 什么是类：
    * OO （class oriented programming） ：数据本质上是相互关联的，对于这些有共同处的数据，应该处以打包封装。
    * 类与数据：
        * 字符，字符串就是数据
        * 但是真正关心的是可对数据做什么： String类（计算长度，添加数据，搜索等等） 
    * 实例化：
        * 对于String类来说，所有的字符串都是一个实例。类就是个包裹，包含的是数据和我们能够操作他的函数
    * 继承:
        * 汽车是交通工具的特例
        * Vehicle类：推进器，载人能力（对于所有的交通工具重复定义载人能力是没有意义的）
          Car类： 声明他继承自Vehicle的这个基类。
        * Vehicle和Car会定义相同的方法，但是实例中的数据可能不同。
    * 多态：
        * 父类的通用行为可以被子类更特殊的行为重写

### 类设计模式 （"Class" Design Pattern）

* 不同的语言对于类有不同的限制，java中类是必选的，c++/php 是可以过程化，类二选一。
* js中对于类的看法：
    * 以前提供近似类的语法元素，es6中提供了`Class`关键字
    * js 的类设计机制和真正的类不同。可以叫做“类类”


## 类的机制（Class Mechanics）

*  蓝图（类） -----复制(实例化)------>建筑（实例）
*  构造函数： 类实例由特殊的类方法构造，这个类方法与类同名，任务就是初始化实例所需要的所有信息
    *  构造函数名与类名相同
    *  使用`new`关键字来调用
    *  用于进行实例初始化

## 类的继承（Inheritance）

* Vehicle类和Car类，SpeedBoat类
  
  ```
  
    class Vehicle {
            engines = 1

            ignition() {
                    output( "Turning on my engine." )
            }

            drive() {
                    ignition()
                    output( "Steering and moving forward!" )
            }
    }
    class Car inherits Vehicle {
            wheels = 4

            drive() {
                    inherited:drive()
                    output( "Rolling on all ", wheels, " wheels!" )
            }
    }

    class SpeedBoat inherits Vehicle {
            engines = 2

            ignition() {
                    output( "Turning on my ", engines, " engines." )
            }

            pilot() {
                    inherited:drive()
                    output( "Speeding through the water with ease!" )
            }
    }
  ```
  
![8976e648bdf936e8da7c1fe057f10bec.png](evernotecid://867AB45F-3149-41C8-B4BB-D927BC11A980/appyinxiangcom/3127539/ENResource/p1321)


### 多态（Polymorphism）

* 相对多态：
    * 上面的例子中`car`类引用了继承来的drive，这种技术就是相对多态。
    * 多态在继承链中一个方法名字可以被定义多次，调用时会选取相应的方法。
    * 其他语言中，子类构造函数是可以相对引用父类的构造函数的，但是js中，es6之前，他们的构造函数是没有存在直接联系，无法简单的实现相对引用。
    * 下面的例子中可以看出引用父类的drive中引用了ignition方法，但是speedboat中也有自己的ignition方法。在调用时，引擎会自动使用speedboat实例的ignition
    * 方法的多态性取决于在哪个类的实例中调用
    ```
    //vehicle中
     drive() {
        ignition()
        output( "Steering and moving forward!" )
    }
    // ... 
    //speedboat 中
    pilot() {
        inherited:drive()
        output( "Speeding through the water with ease!" )
    }
    ```
    
  * super： 子类相对引用继承的父类
    * 子类继承父类方法的副本
    * 改写之后，子父类不会互相影响
    * super的本质是复制。
    
### 多重继承

* 其他语言中提供多重继承这种机制：
    * 问题1： 两个父类中同时定义了相同的方法，这个时候子类引用的哪一个呢？
    * 问题2： 孙子类D 继承自两个父类 b 和 c，b,c 同时改写了来自爷爷类a的方法，这个时候d应该引用哪一个方法呢。
* js中：
    * 并不提供多重继承。
    * 但是官方不提供，开发者野蛮生长了。
    
    
   
## 混入（Mixins）

js中的类（本质是对象），不会复制到其他对象，只会被关联起来。js使用混入模拟类的复制行为。

### 显式混入

* 在其他语言中成为extend，但方便理解我们使用mixin

```
function mixin(source,target){
	for(var key in source){
  	if(!(key in target)){
    	target[key] = source[key];
    }
  }
  return target;
}

...

var Car = mixin(Vehicle, {
    wheels: 4,
    drive: function(){
        Vehicle.drive.call(this);
        console.log(this.wheels);
    }
})
```
* mixin 其实复制的是函数引用。
* car本身有`drive（）`所以并没有被重写。因此实现了子类对父类的重写。
* 显式多态： `Vehicle.drive.call(this)`, 使用硬绑定是为了保证this的执行环境为car。为了指明调用对象，必须使用绝对引用，通过名称显式指定Vehicle并且调用drive
* 显式混入的问题：
    * js会在素有使用多态引用的地方创建关联增加了维护成本
    * 显式多态还会模拟多重继承，更加增加了复杂度，难以阅读难以维护。
    * 复制之后如果引用的相同的对象，还是会影响对方

### 寄生继承

```

```

### 隐式继承

```

```