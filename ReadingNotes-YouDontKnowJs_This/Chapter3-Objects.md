# Chapter 3 Objects


## 语法

* 两种构造方式： 声明（文字）和构造
* 文字方式： 
    ```
    var myObj = {
        key : value
    }
    ```
    构造方式：
    ```
    var myObj = new Object();'
    myObj.key = value;
    ```
   区别： 构造形式中必须逐个添加属性，文字声明可以添加多个键值对。构造形式很少见，内部也是使用文字声明。

## Type

* 6种语言类型（primary types）： `string`,`number`,`boolean`,`null`,`undefined`,`object`
* simple primitives 简单基本类型：
    * string，boolean,number,null,undefined,都不是对象
    * null是简单基本类型，但是执行`typeof null`会返回object是一个bug
    (WHY? 底层二进制，如果二进制前三位为0会被认为是对象，null全都是0 所以前三位为0，因此会返回Object)
* complex primitives复杂基本类型
    * function
    * object
    * array
* 对象的子类型：内置对象
    * String,Number,Boolean,Object,Function,Array,Date,RegExp,Error
    * 上面的这些其实是一些内置函数，可以当做构造函数使用，使用new产生的函数调用，构造一个对应子类型的新对象
    
    ```
    var strPrimitive =  “I am a String”;
    typeof strPrimitive; //string
    strPrimitive instanceof String;//false
    
    var strObject = new String('I am a string');
    typeof strObject; //"object"
    strObject instanceof String //true
    
    Object.prototype.toString.call(strObject);//[object String]
    ```
    
    * 对于`Object`,`Function`,`Array`,`RegExp`使用构造形式会有一些额外的选项，所以当有需要的时候可以使用构造形式，但普通的时候都建议使用文字形式。

| 类型 | 形式  |
| --- | --- |
| String | 字面量会自动转为对象 |
| Number | 数值字面量会自动被转化成`new Number(42)`  |
| null,undefined | 没有构造形式，只有文字形式 |
| Date | 只有构造形式，没有文字形式 |
| Object,Array,Function,RegExp | 无论使用文字形式还是构造形式都是对象 |
| Error | 抛出异常的时候自动创建 |

## Contents 内容

* 对象内部其实是属性的名称，真正的存储位置并不在对象内部，名称类似于指针指向真正的位置
* 两种访问方式
    * `.`property access 属性访问
    * `[]` key access 键访问
    * 区别： `.`要求标识符满足命名规范，'[]'则允许任何URF-8 的字符串作为属性名。如果key 是变量，使用key access
    * property的名字必须为string，其他类型会被转化为字符创。
   
   ### 可计算属性名
   
   对于文字形式声明对象时是无法通过`[]`的形式进行声明的。ES6 中增加了computed property Name使用`[]`包裹一个表达式来当做属性名。
   
```
    var prefix = "foo";

    var myObject = {
            [prefix + "bar"]: "hello",
            [prefix + "baz"]: "world"
    };

    myObject["foobar"]; // hello
    myObject["foobaz"]; // world
```
    
### 属性与方法
 
 * 对于其他语言，当function 属于某个对象（类）的时候会被叫做Method
 * 对于js，函数永远也不会属于某个对象的method，他们只是对于相同函数的一个引用。只不过区别在this的隐式绑定，但这种绑定是动态的。

### 数组

* 数组也是对象，每个下标为整数，但是仍然可以添加属性，但是不会影响length的值。
* 虽然理论上可以将数组完全当做对象来用，但是js对于数组和对象分别根据用途和行为做了相对应的优化，所以最好还是区分使用。
* 如果属性名看起来像一个数字，则会被当做数值下标。

```
    var myarray = ["foo",42, "bar"];
    myarray.baz = "baz";
    myarray.length; // 3
    myarray.baz; // "baz"
    
    //如果属性名看起来像一个数字，则会被当做数值下标
    var myArray = [ "foo", 42, "bar" ];
    myArray["3"] = "baz";
    myArray.length;  // 4
    myArray[3];          // "baz"
```

### 对象的复制

* 无法选择一个默认的复制方法

```
function anotherFunction() { /*..*/ }

var anotherObject = {
        c: true
};

var anotherArray = [];

var myObject = {
        a: 2,
        b: anotherObject,    // reference, not a copy!
        c: anotherArray,     // another reference!
        d: anotherFunction
};

anotherArray.push( anotherObject, myObject );
```


    
深复制`myObject`的时候，会复制`anotherArray`，`anotherFunction`,和`anotherObject`，如果复制 `anotherObject`又需要复制myObject，形成了一个无限的循环引用。这个时候会有一个问题，我们如何处理这种循环引用，是应该深层的复制不复制还是弹出报错？

* 函数的处理： 复制一个函数以为这什么。有的会通过toString将源代码序列化。
* 解决方法1：对于JSON安全的对象,但只解决部分情况。
`JSON.parse(JSON.stringify(obj))`
* 浅复制： 
    * `Object.assign({} , myPbject)` 
    * assign 的工作原理： 遍历一个或者多个源对象的所有**可枚举属性（enumerable）**，**owned keys（自有键）**，**并使用等号复制运算赋值给目标对象**。源对象属性的一些特性（writable）不会被复制到目标对象

    ```
    var newObj = Object.assign( {}, myObject );

    newObj.a; // 2
    newObj.b === anotherObject;          // true
    newObj.c === anotherArray;           // true
    newObj.d === anotherFunction;        // true
    ```
    
### 属性描述符

```
var myObject = {
        a: 2
};

Object.getOwnPropertyDescriptor( myObject, "a" );
// {
//    value: 2,
//    writable: true,
//    enumerable: true,
//    configurable: true
// }
```
* 属性描述符（数据描述符）：
    * value
    * writable： 是否可以修改属性的值
        * 非严格模式 如果writable是false的话，不能修改属性
        * 严格模式下，会直接报错
    * enumerable：
        * 控制属性是否出现在对象的属性枚举中。（for..in）中不会遍历该属性为false的属性
    * configurable
      * 如果属性是可配置的则可以使用`definePropery`进行修改属性符。
      * 如果不可配置，无论是否是严格模式，都会报错typeerror。
      * configurable 为false的时候，是允许writable 由 true 改为false，但是无法由false改为true。
      * configurable 为false的时候还会禁止删除改属性
      
```
var myObject = {
        a: 2
};

myObject.a = 3;
myObject.a; // 3

Object.defineProperty( myObject, "a", {
        value: 4,
        writable: true,
        configurable: false,      // not configurable!
        enumerable: true
} );

myObject.a;// 4
myObject.a = 5;
myObject.a; // 5

Object.defineProperty( myObject, "a", {
        value: 6,
        writable: true,
        configurable: true,
        enumerable: true
} ); // TypeError


delete myObject.a;
myObject.a;                             // 2
```
* 使用`Object.defineProperty`添加新的属性，或如果`configurable`是true的话可以修改已有的属性。
* 例子： 
    ```
    var myObject = {};
    Object.defineProperty(myObject,'a',{
    value : 2,
    writable: true,
    configurable: true,
    enumerable: true,
    });
    
    myObject.a // 2
    ```
 使用defineProperty 显式指定的属性，但是一般不会这样指定，除非像修改属性描述符。
 
### 不变性

* 对象常量： `writable`为false，`configurable`为false

    ```
    var myObject = {};

    Object.defineProperty( myObject, "FAVORITE_NUMBER", {
            value: 42,
            writable: false,
            configurable: false
    } );
    ```
    
* 禁止扩展（`preventExtensions`）
    * 禁止一个对象添加新属性并且保留已有属性`Object.preventExtensions`.在非严格模式下，静默失败。严格模式下抛出typeError的错误 
    
    ```
    var myObject = {
            a: 2
    };

    Object.preventExtensions( myObject );

    myObject.b = 3;
    myObject.b; // undefined
    ```
    
  * seal （密封）
    * seal 会创建一个密封的对象，本质是在现有对象上调用`preventExtensions`和`configurable:false`.这个就限定了既不能重新配置也不能删除任何已存的属性。但是你可以修改这些属性的值
   * freeze （冻结）
        * `Object.freeze`会创建一个冻结对象，本质上是调用了`seal`并且将`writable`设置为false。因此值是不允许改变的。
        * freeze的不可变行是最高的
        * 这个冻结是浅冻结，深冻结是可以实现的，但是注意不要冻结到其他共享对象。
        
        
### [[get]]

```
var myObject ={
    a:2
}
myObject.a //2
```

`myObject.a`看起来像是一次属性访问，但其实不是指在对象内部查找a，实际上是实现了一次[[get]]操作：
    * 在对象内部查找是否有名称相同的属性，找到就返回这个值
    * 没有找到的话，[[get]]算法的定义会执行遍历[[prorotype]]原型链。
    * 无论如何都没有找到相同的属性，则会返回undefined。与变量不同，如果当前的词法作用域中无法找到变量，会抛出referenceError异常。
    
* 一个特殊的例子

    ```
    var myObject = {
            a: undefined
    };
    myObject.a; // undefined

    myObject.b; // undefined
    ```
    
    无法仅仅通过值的结果判断属性的存在。
    
### [[put]]

* [[put]]被触发的时候，会先判断是否对象中已经存在这个属性。
    * 存在：
        * 属性是否是访问描述符？是，存在setter则调用setter
        * 属性的writable 是否是false，否的话静默失败或者抛出异常
        * 不是访问描述符，writable也是true的话，设置属性的值
        
### getters and setters

* getter 和 setter 部分改写默认操作，只能用在单个属性中，无法用在整个对象上。
* 对象属性同时具备getter和setter，这个属性会被定义为访问描述符
* 同时具备getter 和setter 忽略value和writable 特性。这个时候getter和setter，configurable，enumerable占据主导
* 定义get的两种方式

```
// 第一种方式
var myObject = {
        // define a getter for `a`
        get a() {
                return 2;
        }
}
// 第二种： 显示定义
Object.defineProperty(myObject, "b",{
    get: function(){ retrun this.a *2},
    enumerable : true
)
```

* 当只有定义getter而没有定义setter的时候，任何赋值操作并不会抛出异常，而且即使有setter因为默认返回的2，所有set是没有意义的
* 通常setter 和 getter是成对出现的

```
var myObject = {
get a(){
    return this._a_;
}

set a(val){
    return this._a_ = val*2;
}
};

myObject.a = 2;
myObject.a // 4
```

### 存在性


* 检查存在性的2种方式
    * `in` 操纵符,会遍历原型链
    * `hasOwnProperty`只会在当前对象中寻找
    
* 普通对象都可以通过Object.prototype 的委托来访问hasOwnProperty 但是如果创建方式是通过`Object.creater(null)`来创建的`hasOwnProperty`则会失败。
* `in`操作符检查的是某个属性名是否存在，不是去检查值。


```
var myObject = {
    a: 4
};

("a" in myObject) //true
( "b" in myObject) // false

myObject.hasOwnProperty("a");
myObject.hasOwnProperty("b");

```

### 可枚举性

* 可枚举性就是是否可以出现在对象属性的遍历中
* 检查属性是否可枚举还有另外一种方式： `myObject.propertyIsEnumerable `,这种方式是检查属性是否直接存在于对象中（非原型链）并且enumerable 为 true

```
Object.defineProperty(myObject,"a",{
enumerable: true,
value:2
});
Object.defineProperty(myObject,"b",{
enumerable: false,
value:3
});

myObject.b;//3
'b' in myObject; //true

for(var k in myObject){
console.log(k)
}
//a

```

* 两种方式获得所有的属性：
    * Object.keys()返回所有的可枚举属性
    * Object.getOwnPropertyNames() 返回所有属性，无论是否可以枚举
    * 这两种方式都只会查找对象直接包含的属性
    
    
    
    
## 遍历

* for..in 并不是遍历所有属性的值而是遍历的可枚举属性
* ES6 中 for..of遍历的是属性的值
* 原理：
    * 先请求的是迭代器对象，通过调用迭代器的next()方法遍历所有的返回值
    * `Symbol.iterator`是用来获取对象的@@iterator内部属性，@@iterator不是一个迭代器对象，而是一个返回迭代器对象的函数
    * 数组内部是有迭代器函数，但是对象没有，是无法通过自身完成`for...of`遍历的    
    
 ```
 var myArr = [1,2,3];
 
 var it = myArr[Symbol.iterator]();
it.next(); // { value:1, done:false }
it.next(); // { value:2, done:false }
it.next(); // { value:3, done:false }
it.next(); // { done:true }
 ```
 
虽然没有内置的轮子，但是对于想遍历的对象可以定义`@@iterator`。

```
var myObject = {
    a: 2,
    b: 3
}

Object.defineProperty(myObject,Symbol.iterator, {
    enumerable: false,
    writable: false,
    configurable: true,
    value: function(){
        var o = this;
        var idx  = 0;
        var ks = Object.keys(o);
        return {
            next: function() {
                return {
                    value: o[ks[idx++]],
                    done： idx> ks.length
                }
            }
        }
    }

});


var it = myObject[Symbol.iterator]();
it.next(); // { value:2, done:false }
it.next(); // { value:3, done:false }
it.next(); // { value:undefined, done:true }

// iterate `myObject` with `for..of`for (var v of myObject) {
        console.log( v );
}
// 2
// 3
```