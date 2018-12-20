## 5.1 [[prototype]]

### 介绍

* 本质是对于其他对象的引用，在试图引用对象属性会触发[[get]]操作，get操作首先检查对象本身有没有这个属性，如果没有，则开始遍历原型链。

```
var anotherobj = {
a:2,
};
var myobj = Object.create(anotherobj);
myobj.a //2
```

* `Object.create(..)`的原理是创建一个对象，并把它关联到指定的对象
* 查找的终止条件是： 
    * 找到了
    * 或者查完了整个原型链，此时返回`undefined`

* 遍历原型链的两种方式：
    * `for...in`遍历整个原型链，并访问`enumerable`为`true`的属性
    * `某个属性 in object` 遍历原型链，无论这个属性是否为enumerable
    
### Object.prototype

* 所有普通的prototype的终点都是`Object.prototype` 
* 对象的`toString`,`valueOf`,`hasOwnProperty`都是因为这个原型链终点。

### 属性设置和屏蔽

对于语句`myObject.foo = bar;`:
* 如果myObject内部有foo的普通访问属性，会直接修改已有的属性
* 如果foo在原型链中，会遍历原型链，如果找不到，会在myObject 内部添加foo属性。如果在上层找得到，会出现三种情况
    *  上层存在名为`foo`的普通数据访问属性，并且`writable:true`,会直接在myObject中添加名为`foo`的新属性
    * 上层存在名为`foo`的普通数据访问属性，并且`writable:false`,如果是严格模式，会弹窗报错，否则会跳过这个语句
    * 如果找的到并且存在setter，会调用这个setter，foo不会被添加到myobject，也不会修改这个foo的setter
* 如果在第二种第三种情况下也希望在`myobject`中增加屏蔽属性，就不能使用`myObject.foo = ' bar '`这种格式，需要`Object.defineProperty()`向myObject添加属性。
* 使用`myObject.a ++`会产生隐式屏蔽
    ``` 
    var anotherObject = {
            a: 2
    };

    var myObject = Object.create( anotherObject );

    anotherObject.a; // 2myObject.a; // 2

    anotherObject.hasOwnProperty( "a" ); // truemyObject.hasOwnProperty( "a" ); // false

    myObject.a++; // oops, implicit shadowing!
    //这句话相当于myObject.a = myObject.a + 1;

    anotherObject.a; // 2myObject.a; // 3

    myObject.hasOwnProperty( "a" ); // true
    ```
## 5.2 类

### 类函数

* 函数会有一个默认的公有且不可枚举的的属性`prototype`被称为foo的原型
* 本质上是所有通过函数创建的实例, 都会关联到这个对象上.
* 实际上并没有进行实例的复制，是创建 了多个对象，只不过`[[prototype]]`关联的是相同的对象

* 原型继承：
    * 本质上并不是继承，因为没有进行复制操作，一个对象可以通过委托访问另一个对象的属性和函数
    * ![5dd7bedf6d9a04f9a60714a0ff1e6826.png](evernotecid://867AB45F-3149-41C8-B4BB-D927BC11A980/appyinxiangcom/3127539/ENResource/p1322)

### 构造函数

```
function Foo() {
        // ...
}

Foo.prototype.constructor === Foo; // true

var a = new Foo();
a.constructor === Foo; // true
```
* `Foo.prototype` 有一个公有，不可枚举属性`.constuctor`,这个属性引用的是对象关联函数
* 通过`new Foo()`创建的函数也有一个`.constructor`属性，指向创建对象的函数。
* 其实Foo函数本身不是构造函数，就是普通的函数，反而，new关键字的调用其实是构造函数调用。`new`关键字会劫持普通函数用构造对象的形式来调用他

### 技术

* 两种面向类的技术：
    * `this.name = name`给每个对象都添加了`.name`属性。
    * `Foo.prototype.myName = ...` 给原型对象添加了属性
    * 在创建过程中，a 和 b的内部[[prototype]] 都会关联到Foo.prototype。当无法找到`myName`,会通过委托在`Foo.prototype`
* `.constructor`只是通过默认的[[prototype]]委托指向Foo,和构造没有关系，一旦修改Foo.prototype 则能证明说法

```
function Foo() { /* .. */ }

Foo.prototype = { /* .. */ }; // create a new prototype object

var a1 = new Foo();
a1.constructor === Foo; // false!
a1.constructor === Object; // true!
```

因为a1没有.constructor 属性，所以委托[[prototype]]上的Foo.prototype.新赋值的对象也没有。因此再往上找，则Object有，所以指向内置的`Object`

* 如何在更改Foo.prototype 还可以找到构造函数。
  
  ```
  function Foo() { ... };
  Foo.prototype = { ... };
  
  Object.defineProperty(Foo.prototype, 'constructor', {
    value: 'Foo',
    writable: true,
    configurable: true,
    enumerable: false,
  })
  ```
* `a1.constructor`是一个非常不安全的引用，尽量避免使用这样的引用。


## 5.3 原型继承

```

function Foo(name) {
        this.name = name;
}

Foo.prototype.myName = function() {
        return this.name;
};

function Bar(name,label) {
        Foo.call( this, name );
        this.label = label;
}

// here, we make a new `Bar.prototype`
// linked to `Foo.prototype`
Bar.prototype = Object.create( Foo.prototype );

// Beware! Now `Bar.prototype.constructor` is gone,
// and might need to be manually "fixed" if you're
// in the habit of relying on such properties!

Bar.prototype.myLabel = function() {
        return this.label;
};

var a = new Bar( "a", "obj a" );

a.myName(); // "a"a.myLabel(); // "obj a"
```

原型继承展示出了`Bar.prototype`和`Foo.prototype`的委托关系。`Bar.prototype = Object.create( Foo.prototype );`会创建一个新的`Bar.prototype`并且把新对象内部的[[prototype]]关联到指定的对象（`Foo.prototype`）;

* `Bar.prototype = Foo.prototype`没有创建出新的对象，只是添加了指向Foo.prototype 的引用。
* `Bar.prototype = new Foo()`虽然实现了关联到吗指定的对象，但同时创建了`.prototype`和 '.constructor'
* ES6之前使用的`._proto_`,ES6之后添加了辅助函数`Object.setPrototypeOf(..)`: `Object.setPrototypeOf(Bar.prototype,Foo.prototype)`

### 检查类之间的关系

* 对象与函数之间的关系： `a instanceof A` 。意思是在a 的整条原型链上面是否有指向Foo.prototype 的对象。

