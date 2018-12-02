# this全面解析

## 温故

* `this` binding made for each function invocation( 是在每个函数调用的时候进行绑定的)，
* `this` based entirely on its call-site（this完全取决于函数的调用位置）

## 调用位置

* 调用位置就是函数在代码中被调用的位置。
* 分析调用栈（为了到达当前执行位置所调用的所有函数），函数的调用位置就是在当前执行的函数的前一个调用中。
* 如何从调用栈中分析出调用位置，这决定了this的绑定

```
function baz(){ ... };//当前调用位置就是全局作用域
baz();// <-- 调用了baz
```

```
function baz(){
    //当前的调用栈是baz ，调用位置是全局作用域
    console.log('baz');
    bar();//bar 的调用位置
}

function bar(){
    //当前调用栈 baz -> bar
    //调用位置: baz
    console.log('bar');
    foo(); // foo 的调用位置
}

function foo(){
    //当前调用栈是baz->bar-> foo
   //当前的调用位置在bar中
   console.log('foo');
}


baz();
```

## 绑定规则

* 四条绑定规则
    * 默认绑定
    * 隐式绑定
    * 显式绑定
    * new 绑定
### 默认绑定

* 最常用的函数调用类型：独立函数调用（standalone function invocation）
* 在无法应用其他规则时，此条是默认规则

```
function foo() {
        console.log( this.a );
}

var a = 2;

foo(); // 2   
```

* 上面的例子函数调用时候应用了this的默认绑定: this指向全局对象
* 在直接使用不带任何修饰参数的函数引用进行调用的时候，使用默认绑定。
* 严格模式中，不能将全局对象用于默认绑定，this会绑定undefined；（注意：函数内部是严格模式）
* 在调用位置是严格模式不影响默认绑定（第三方库的严格模式不同会导致这类情况）

```

function foo() {
        "use strict"; //函数内部是严格模式
        console.log( this.a );
}

var a = 2;
// --------------------------------------
// 严格模式不影响this的绑定，虽然不会有人将严格非严格混用


function foo() {
        console.log( this.a );
}

var a = 2;

(function(){
        "use strict";
        foo(); // 2
})();

```

### Implicit Binding （隐式绑定）

* 上下文对象，是否被某个对象拥有或者包含

```
function foo(){
    console.log(this.a);
}

var obj = {
    a:2,
    foo: foo
}

obj.foo();
```

`obj.foo()` : 调用栈使用`obj`作为上下文来引用函数，因此此时`obj`包含函数的引用。
* 当函数引用有上下文对象的时候，隐式绑定会把this绑定到这个上下文对象。
* 对象的属性联众只有最后一层的调用位置起作用。

```
function foo() {
//调用栈： obj1-》obj2
        console.log( this.a );//this是obj2
}

var obj2 = {
        a: 42,
        foo: foo
};

var obj1 = {
        a: 2,
        obj2: obj2
};

obj1.obj2.foo(); // 42
```

* 隐式丢失：
   严格模式影响隐式绑定的规则

```
function foo() {
        console.log( this.a );
}

var obj = {
        a: 2,
        foo: foo
};

var bar = obj.foo; // function reference/alias!

var a = "oops, global"; // `a` also property on global object

bar(); // "oops, global"
```

注：`bar`虽然引用的是obj.foo 但是实际引用的是函数本身，因此在执行的时候`bar()`其实是一个不带任何修饰的函数调用。所以是默认绑定

* 参数传递是一种隐式传递

```
function foo() {
        console.log( this.a );
}

function doFoo(fn) {
        // `fn` is just another reference to `foo`

        fn(); // <-- call-site!
}

var obj = {
        a: 2,
        foo: foo
};

var a = "oops, global"; // `a` also property on global object

doFoo( obj.foo ); // "oops, global"
```

```
//setTimeout 函数的内部实现
function setTimeout(fn,delay){
fn();//也是作为参数传进来的情况，造成this指向的是全局
}

function foo() {
        console.log( this.a );
}

var obj = {
        a: 2,
        foo: foo
};

var a = "oops, global"; // `a` also property on global object

setTimeout( obj.foo, 100 ); // "oops, global"
```

### Explicit Binding(显式绑定)

* js提供了绝大多数的函数可以使用的方法来进行this绑定：
`call` 和 `apply`
* 第一个参数是对象，给this准备的，调用函数的时候将this绑定到这个对象上。这种绑定方式就是显式绑定。

```
function foo() {
        console.log( this.a );
}

var obj = {
        a: 2
};

foo.call( obj ); // 2
```

* 使用call方法强制将this绑定到obj。
* 如果传入参数是一个原始值，原始值会转换成对象的形式。（这就是装箱boxing）
* 显示绑定也无法解决丢失绑定问题

```
function foo(){
console.log(this.a);
}

var obj = {
a:2,
}

var bar = function(){
foo.call(obj);
//强制把foo的this绑定到obj，无论如何调用bar，都是在obj上调用foo
//显式的强制绑定。
}

bar();
setTimeout(bar,100);
bar.call( window ); // 2
//无论你怎么操作bar的this ，foo的this永远是obj
```

应用场景： 创建一个包裹函数，负责接收函数并返回值

```
function foo(something){
    console.log(this.a , something);
    return this.a + something;
}

var obj = {
    a:2
}

var bar = function(){
    return foo.apply(obj,arguments)
}

var b = bar (3); // 2 3
console.log(b);//5
```

```
function foo(something){
...
}

function bind(fn,obj){
    return function(){
        return fn.apply(obj,arguments)
    }
}
```

* 硬绑定的内置实现: bind函数，用途：将this绑定到参数的对象中，返回一个函数。硬编码的新函数，会把指定的参数设置为this的上下文，并调用原始函数。
* 在es6中，硬绑定的函数有一个从原函数中派生出的name属性。比如说`bar= foo.bind(..)`有一个`bar.name = bound foo`,这个属性与在堆栈追踪中显示的一样

```
function foo(something){
    console.log(this.a, something);
    return this.a + something;
}


var obj = {
        a: 2
};
foo.bind( obj );

var b = bar( 3 ); // 2 3
console.log( b ); // 5
```

* api调用的context
    可选参数`context`用于指定回调函数的this
    
    ```
    array.forEach(callback(currentValue, index, array){
        //do something
    }, this) // <---- 这里
    
    function foo(el) {
            console.log( el, this.id );
    }

    var obj = {
            id: "awesome"
    };

    // use `obj` as `this` for `foo(..)` calls
    [1, 2, 3].forEach( foo, obj ); // 1 awesome  2 awesome  3 awesome
    ```
    
### `new`绑定

* 有关于构造函数:
    * 构造函数只是使用new操作符时被调用的函数。
    * 不属于某个类，也不会实例化一个类。
    * 只是被new操作符操纵的普通函数
    
* 包括内置对象的所有函数都可以用new来调用。这种调用就是构造函数调用。
* 真相：并不存在什么构造函数，只有对于函数的构造调用

* 执行步骤：
    1. 创建一个崭新的对象。
    2. 执行[[prototype]]连接
    3. 新对象绑定到函数调用的this
    4. 如果函数没有其他的返回对象，默认返回这个新对象

```
function foo(a){
this.a = a;
}

var bar = new foo(2);

console.log(bar); //{a:2}
```

## 优先级

1. 隐式 vs 显式 
下面的例子可以看出显示的优先级更高，所以应当考虑是否存在显式绑定

```
function foo(something){
    this.a = something ;
}

var obj1 = {
foo:foo
};

var obj2 = {};
obj1.foo(2);
console.log(obj1.a);

obj1.foo.call(obj2);
obj2.foo.call(obj1);
```

2. new vs 隐式
new绑定优先级更高。

```
    function foo(sth){
        this.a = something;
    }
    var obj1 = {
        foo: foo
    }
    var obj2 = {};
    obj1.foo(2);
    console.log(obj1.a);
    
    obj1.foo.call(obj2, 3);
    console.log(obj2.a);
    
    var bar = new obj1.foo( 4 );
    console.log( obj1.a ); // 2
    console.log( bar.a ); // 4
```

3. new vs 显式

* 无法通过new foo.call(obj1) 来直接创建，可以使用硬绑定来测试优先级
    * 猜想：
    如果new的优先级高，obj的a为2 ，baz的a为3
   如果bar(3)的优先级高，obj的a 为3    
            
```
function foo(something) {
        this.a = something;
}

var obj1 = {};

var bar = foo.bind( obj1 );
bar( 2 );
console.log( obj1.a ); // 2

var baz = new bar( 3 );
console.log( obj1.a ); // 2
console.log( baz.a ); // 3
```

* 因为polyfill的兼容代码与es5内置的bind函数不相同，所以无法创建一个不包含.prototype的函数。所以若果new中使用硬绑定函数并依赖polyfill的话要小心
* 函数内会判断硬绑定函数是否被new 调用，如果是，则新创建的this会替代硬绑定的this

## 判断this
1. 是否存在new绑定，有的话，this一定绑定的是新创建的对象
2. 是否存在call，apply，bind这种显式绑定？ 有的话就是绑定指定的特定对象
3. 是否存在隐式绑定？ 比如 `obj1.foo()`这种情况
4. 其余情况则是默认绑定，严格模式绑定undefined，非严格模式为全局对象。`var bar = foo();`

## this 的绑定例外

### null作为apply，call，bind的参数的时候，其实应用的是默认的绑定规则

```
function foo(){
console.log(this.a);
}

var a = 2;
foo.call(null);//2
```

使用场景：
1. 展开数组，并且当做参数传入函数
2. 对参数进行科里化

```
function foo(a,b){
console.log(a,b)
}
// 展开数组，并且当做参数传入函数
foo.apply(null, [2,3]);
// 对参数进行科里化
var bar = foo.bind(null,2);
bar(3);
```

* NOTE:
    * 因为这两个函数都要求传入一个进行this绑定的对象，因为场景并不关心this的指向，所以需要传入null进行占位
    * ES6中可以使用扩展运算符来进行参数的的展开，但没有合适的方式进行科里化，因此还要继续使用bind

* 问题
    * 忽略this的绑定如果函数中真有this相关的绑定，则会把this绑定到全局对象，导致不可预计到后果


* 更安全的this
    创建一个DMZ(demilitarized zone)非军事区对象，把this绑定到这个空的非委托对象。使用`Object.create(null)`来创建。
（WHY：`Object.create(null)`不会创建Object.prototype这个委托，比{}更空）
使用φ： 更安全，更有可读性

```
function foo(a,b) {
        console.log( "a:" + a + ", b:" + b );
}

// our DMZ empty object
var ø = Object.create( null );

// spreading out array as parameters
foo.apply( ø, [2, 3] ); // a:2, b:3

// currying with `bind(..)`
var bar = foo.bind( ø, 2 );
bar( 3 ); // a:2, b:3
```

### 间接引用Indirection

* 对于间接引用的函数，会使用默认绑定规则。
    * 对于函数的引用赋值会导致this丢失
    * 如果执行的是`p.foo()`则`this.a`的值为4，但是在赋值之后，返回的是函数本身`foo`此时的函数是暴露在全局作用域中的，所以输出结果为2.应用的是默认绑定
    * 调用位置是否为严格模式不会影响this绑定的是全局还是`undefined`，函数体内为严格模式则绑定`undefined`

```
function foo(){
    console.log(this.a);
}

var a =2;
var o = { a: 3, foo: foo};
var p = {a: 4};

o.foo();
(p.foo = o.foo)();
```


### 软绑定

* 硬绑定： this强制绑定到指定的对象，只有通过更高优先级的new才能修改this指向
问题：灵活性被降低，无法通过显式或者隐式来修改this的指向。

```
function foo() {
   console.log("name: " + this.name);
}

var obj = { name: "obj" },
    obj2 = { name: "obj2" },
    obj3 = { name: "obj3" };

var fooOBJ = foo.softBind( obj );

fooOBJ(); // name: obj

obj2.foo = foo.softBind(obj);
obj2.foo(); // name: obj2   <---- look!!!

fooOBJ.call( obj3 ); // name: obj3   <---- look!

setTimeout( obj2.foo, 10 ); // name: obj   <---- falls back to soft-binding
```

原理： 检查是否this是undefined或者全局对象，如果是，this绑定到指定的默认对象，不是则this该是什么就是什么。


## this词法

* 如果使用的是箭头函数，this不符合四种规则
* 箭头函数中的this是根据外层作用域来决定this
* 箭头函数中的this一旦被指定，则无法更改，new也无法更改

```
function foo() {
        // return an arrow function
        return (a) => {
                // `this` here is lexically adopted from `foo()`
                console.log( this.a );
        };
}

var obj1 = {
        a: 2
};

var obj2 = {
        a: 3
};

var bar = foo.call( obj1 );
bar.call( obj2 ); // 2, not 3!

function foo(){
setTimeout(()=>{console.log(this.a)},5000)
}

var obj = {a:2};
foo.call(obj);//2
```

* 箭头函数确保了this被绑定到指定的对象，取代了传统的this机制。
