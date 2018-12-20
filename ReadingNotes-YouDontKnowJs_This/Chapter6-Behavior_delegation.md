## 面向委托的设计

* [[prototype]]机制其实根本上就是对象中的一个内部链接引用了另外一个对象
* 原型链： 一个对象中没有找到需要的属性或者方法引用，则会在[[prototype]]上进行查找。同理，如果后者也没找到，则会在后者的[[prototype]]上继续查找。

### 类理论

举例：

```
class Task {
        id;

        // constructor `Task()`
        Task(ID) { id = ID; }
        outputTask() { output( id ); }
}

class XYZ inherits Task {
        label;

        // constructor `XYZ()`
        XYZ(ID,Label) { super( ID ); label = Label; }
        outputTask() { super(); output( label ); }
}

class ABC inherits Task {
        // ...
}
```

图示：

![c62a7ed1b34eb26a144a8d66e3445c10.png](evernotecid://867AB45F-3149-41C8-B4BB-D927BC11A980/appyinxiangcom/3127539/ENResource/p1324)


### Delegation Theory

* 被委托的一定是个对象，不是类也不是函数。 本例子中是Task。
* 对于子任务XYZ,和ABC，需要两个兄弟对象（XYZ和ABC)均要使用对象来存储对应的数据和行为。

```
var Task = {
        setID: function(ID) { this.id = ID; },
        outputID: function() { console.log( this.id ); }
};

// make `XYZ` delegate to `Task`var XYZ = Object.create( Task );

XYZ.prepareTask = function(ID,Label) {
        this.setID( ID );
        this.label = Label;
};

XYZ.outputTaskDetails = function() {
        this.outputID();
        console.log( this.label );
};

// ABC = Object.create( Task );
// ABC ... = ...
```

![e1bffb05522b4eb45fcd4f75d66a8e3f.png](evernotecid://867AB45F-3149-41C8-B4BB-D927BC11A980/appyinxiangcom/3127539/ENResource/p1325)


* OLOO风格的代码：
    * 如果在XYZ上面调用setId的话id和label就是属于XYZ，状态是保存在委托者上的不是委托目标上的。
    * 委托中尽量避免使用同名的函数。否则就需要像硬绑定这种手法来消除歧义。
    * 委托行为会在对象找不到属性或者方法引用的时候，将请求委托给另一对象。
    * 禁止互相委托：因为如果被委托双方都没有这个属性的话就是一个无线递归的循环。
    * 不同调试工具对于委托结果的输出可能不太一致：
        * 对于下面的代码Chrome和Firefox的结果可能不太一样
        
        ```
        function Foo(){};
        
        var a1 = new Foo();
        
        a1;//Foo{} in Chrome ,Object in Firefox
        ```
       * 实际上Chrome并不是单纯的追踪了constuctor.name, 因为如果显式声明了Foo 的 prototype的constructor，a1的输出值并不受到影响。实际上使用的是类似于下面代码的构造。
       ```
       var Foo = {};
       var a1 = Object.create(Foo);
       a1;//Object
       Object.defineProperty(Foo,"constructor", {
        enumerable:false,
        value: function Gotcha(){}
       })
       a1//Gotcha
       ```
     * 如果不是使用构造函数来生成对象的话，chrome是无法追踪对象内部的构造函数名称。

## 6.2 控件类

![789b40710ac1d044f14c0a5c633c2ac4.png](evernotecid://867AB45F-3149-41C8-B4BB-D927BC11A980/appyinxiangcom/3127539/ENResource/p1326)


```
function Widget(width,height) {
        this.width = width || 50;
        this.height = height || 50;
        this.$elem = null;
}


Widget.prototype.render = function($where){
        if (this.$elem) {
           this.$elem.css = function($where){
            console.log('this.width',this.width);
            console.log('Widget ele',this);
            this.$elem.setCss = 'css'+'widget;'
           }
        }
};

```

ES6 的 Class语法糖：
 * `super(width,height)` 代替了原本的`Widget.call(this,width,height)`
 * `super.render($where)` 代替了原本`Widget.prototype.render.call(this,$where)`
 * 使用了super之后很多不讨喜的语法都看不见了，但其实内部还是存在的
 
 
 ### 委托控件对象
 
* 使用具名的方法名，不再是`render`而是`insert`和`build`.
* 避免使用显示伪多态(`Widget.call`和`Widget.prorotype.render.call`)，使用`this.init()`和`this.insert（）`
* 构造和初始化的拆分使代码更灵活。
    
    ```
        var btn1 = new Button(...);
        
        //变成为
        
        var btn1 = Object.create(Button)
        btn1.setup(...);
    ```

* 使用对象关联可以更好的关注分离（separation of concerns），即创建和初始化的分离

6.3 登录实例： 更简单的设计

```
function Controller(){
  this.errors = []
}

Controller.prototype.showDialog= function(title,msg) {
  console.log(`title:${title},message:${msg}`);
}

Controller.prototype.success = function(msg){
  this.showDialog('Succes',msg);
}

Controller.prototype.failure = function(err){
  this.showDialog('Failure',err);
  this.errors.push(err);
}

// 子类 LoginController

function LoginController(){
//   Controller.call(this);
}

LoginController.prototype = Object.create(Controller.prototype);

LoginController.prototype.getUser = function(){
  return 'default user'
}

LoginController.prototype.getPassWord = function(){
  return 'pass'
}

LoginController.prototype.validateEntry = function(user,pw){
  user = user || this.getUser();
  pw = pw || this.getPassWord();
  
  if(!(user && pw)){
    return this.failure('please enter name and password')
  }
  else if(pw.length < 5){
    return this.failure('less than 5')
  }
  return true
}

LoginController.prototype.failure = function(err){
   Controller.prototype.failure.call(this,'login failure,'+ err)
}

// 子类AuthController

function AuthController(login){
//   Controller.call(this);
    this.login = login;
}

AuthController.prototype = Object.create(Controller.prototype);

AuthController.prototype.server = function(url,data) {
  return new Promise((resolve,reject)=>{
    setTimeout(()=>{
    console.log('moc request:'+url+',data:',data)
    reject();
  },5000)
  })
};

AuthController.prototype.checkAuth = function (){
  let user = this.login.getUser();
  let pw = this.login.getPassWord();
  
  if(this.login.validateEntry(user,pw)){
    this.server('/check-auth',{user,pw}).then(this.success.bind(this)).catch(this.failure())
  }
  
}
AuthController.prototype.success = function (){
  Controller.prototype.success.call(this,'Auth success')
}

AuthController.prototype.failure = function (){
  Controller.prototype.failure.call(this,'Auth failure')
}


var login = new LoginController();
var auth = new AuthController(login);
auth.checkAuth();


```
### 反类


```
var LoginController = {
    errors :[],
    getUser: function(){
        return 'default user name'
    },
    getPassword: function() {
        return 'defaultpw'
    },
    validateEntry: function(user,pw){
        user = user || this.getUser();
        pw = pw || this.getPassWord();
        if (!(user && pw))
        {
            return this.failure( "Please enter a username & password!" );
        }
        else if (pw.length < 5)
        {
            return this.failure( "Password must be 5+ characters!" );
        }
        return true
    },
    showDialog: function(title,msg){
        console.log(`title:${title},message:${msg}`);
    },
    failure: function(err) {
        this.errors.push( err );
        this.showDialog( "Error", "Login invalid: " + err );
    }
};

var AuthController = Object.create(LoginController);

AuthController.checkAuth = function() {
    var user = this.getUser();
    var pw = this.getPassword();
    console.log(this.accepted.bind(this))
    if (this.validateEntry( user, pw )) {
        this.server( "/check-auth",{
            user: user,
            pw: pw
        } ).then(this.accepted.bind(this)).catch(this.failure.bind(this,'server-rejected'))
    }
};
AuthController.server = function(url,data) {
    return new Promise((resolve,reject)=>{
        setTimeout(()=>{
            console.log('moc request:'+url+',data:',data)
            resolve();
        },5000)
    })
};
AuthController.accepted = function() {
    this.showDialog( "Success", "Authenticated!" )
};
AuthController.rejected = function(err) {
    this.failure( "Auth Failed: " + err );
};

AuthController.checkAuth();
```

此处遗留疑问： 
promise 中使用bind的原因，以及为何使用箭头函数this的绑定就ok，但是使用普通函数this的绑定为全局变量

使用了委托之后我们完全就只有两个对象`loginController`和`AuthController`。

委托机制完全避免掉了基类的这种机制。因为真实的JavaScript中是没有类的概念的，只有对象本身。而且我们完全避免了多态（一个名字爷爷用，爸爸用，儿子用）。通过细化了名字--比如`AuthController` 中的`Accepted`,`rejected`.

* 这就是另一种设计模式： 行为委托。

6.4 Class

```
Class Foo(){
    methodName(){..}
}
```

* 对象委托机制看似使用了function违背了初衷，而Class看似没有function很完美，但实际上并非表面如此。
* ES6 中，对象中的函数可以被简写，这是语法糖。
* ES6中，也可以使用setPrototypeOf来代替以前的`Object.create(..)`

```
Object.setPrototypeOf(AuthController,LoginController)
```

### 反词法

* 简洁表达式虽然很好，但是有一个缺点

```
var Foo = {
    bar(){....};//与下面的写法相同,下面是去掉语法糖之后的效果
    bar: function(){.... }
    baz: function baz(){ ... }
}
```
缩写虽然很好用，但因为实际上是一个匿名函数赋值给bar属性。具名函数表达式会额外给.baz属性附加一个词法名称标识符baz。
（匿名函数的方式会导致: 1. 调用栈难追踪  2. 自我引用更难 3. 更难理解，具名函数在第一点和第三点上面占据优势。）


匿名函数虽然没有name标识符，但是，简洁方法会在函数对象内部设置一个name属性。（追踪的实现是不同的，无法确保一定能使用）

* 使用简洁方式的时候一定要避免自我引用，普通情况下可以通过`Foo.bar（x）`这种方式，但是当多个对象通过代理共享函数，或者有this绑定的时候，就会出现错误
* 对于自我引用函数最佳的方式是使用传统的具名函数`baz： function baz（）`而不是使用简洁的方法。

## 内省


* 自省： 检查实例的类型，主要目的是通过创建的方式来判断对象的结构和功能。

```
function Foo() {
    // ... 
}
Foo.prototype.something = function （）{
    // ...
}

var a1 = new Foo();

if(a1 instanceof Foo){
    a1.something();
}
```

* instanceof 是检查a1和 Foo之间关系，但其实表达是a1 和 Foo.prototype是相互关联的。

```
function Foo(){
...
}
Foo.prototype...
function Bar(){
...
}
Bar.prototype....
Bar.prototype = Object.create(Foo.prototype);

var b1 = new Bar();


Bar.prototype instanceof Foo;//true
Object.getPrototypeOf(Bar.prototype) === Foo.prototype; //true
Foo.prototype.isPrototypeOf(Bar.prototype)// true

b1 instanceof Foo;//true
b1 instanceof Bar;//true
Object.getPrototypeOf( b1 ) === Bar.prototype //true
Object.getPrototypeOf( b1) === Foo.prototype; // true
Foo.prototype.isPrototypeOf(b1);//true
Bar.prototyoe.isPrototypeOf(b1);//true
```

* 内省的几种方式：
    * instanceof： 左边为对象右边为函数。可以沿着原型链进行检查
    * `Objec.getPrototypeOf`:获得对象的直接原型， 不存在沿着原型链找
    * `isPrototypeOf` 判断一个对象是不是后边对象的原型。可以沿着原型链往下找 
    
* 鸭子类型
    * `if(a1.something){...a1.something()`
    * 举例子：
       Promise就是典型的鸭子类型，判断一个对象是否是Promise，通过检查then方法，但是如果有个对象有自己写的then就会被误判。
 * 委托机制下的内省
 ```
    var Foo = {...];
    var Bar = Object.create(Foo);
    var b1 = Object.create(Bar);
    
    Foo.isProtoypeOf(Bar);
    Object.getPrototypeOf(Bar) === Foo
    
    Foo.isPrototypeOf(b1);
    Bar.isPrototypeOf(b1);
    Object.getPrototypeOf(b1) === Bar
 ```
 
 显然，更加简洁。
 
 ## 总结
 
 行为委托认为对象之间都是兄弟关系。而不是父类和子类的关系。
 只用对象来设计代码，不仅可以让语法更加简洁，而且可以让代码结构更加细腻
对象关联基于`[[protptype]]`的行为委托可以非常自然的实现。