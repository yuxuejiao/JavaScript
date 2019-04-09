* eg1: 简单的函数调用   
```
function foo() {
    console.log("bar");
    console.log(this);
}
foo(); 

```
* eg2: 函数作为对象的属性被调用
```
var myObj = {key: "Obj"};
myObj.logThis = function () {
    console.log(this);
}
myObj.logThis(); 
```
* eg3: 当函数被重新赋值到一个新的变量

```
var myVar = myObj.thisMethod;
myVar();
```

* eg4: 事件监听
```
btn.addEventListener('click' ,function handler(){
  console.log(this);
})
```
* eg5: new 关键字
```
function Person (name) {
    this.name = name;
    this.sayHello = function () {
        console.log ("Hello", this);
    }
}

var awal = new Person("Awal");
awal.sayHello();
```
详解：    



词法作用域： 写代码或者定义时确定的   
动态作用域： 运行时确定的   

js并不具有动态作用域，只有词法作用域
但this关注函数如何调用，像动态作用域

this是在运行时绑定的，并不是在编写时绑定的。它的上下文取决于函数调用时的各种条件。
this的绑定和函数声明的位置没有任何关系，只取决于函数的调用方式。

this 既不指向函数自身也不指向函数的词法作用域

this 实际上是在函数被调用时发生的绑定，它指向什么完全取决于函数在哪里被调用。

foo.call(obj) 调用foo时会把this绑定到obj对象上

箭头函数继承外层函数调用的this绑定

如果要判断一个运行中函数的 this 绑定，就需要找到这个函数的直接调用位置。
找到之后 就可以顺序应用下面这四条规则来判断 this 的绑定对象。
1. 由 new 调用？绑定到新创建的对象。 
2. 由 call 或者 apply（或者 bind）调用？绑定到指定的对象。 
3. 由上下文对象调用？绑定到那个上下文对象。 
4. 默认：在严格模式下绑定到 undefined，否则绑定到全局对象。


PS: [https://stackoverflow.com/questions/3127429/how-does-the-this-keyword-work] 《你不知道的JavaScript》