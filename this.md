每个函数的this是在调用时被绑定的，完全取决于函数的调用位置。（this既不指向函数本身，也不指向函数的词法作用域）      
### 绑定规则（默认绑定、隐式绑定、显式绑定、new绑定）
1、 默认绑定
```
function foo(){
    console.log(this.a)
}
var a = 2;
foo(); // 2
```
```
function foo(){
    "use strict";
    console.log(this.a)
}
var a = 2;
foo(); // TypeError: this is undefined
```
foo是直接调用，因此使用默认绑定。   
* foo运行在非严格模式下，this指向全局对象window。
* foo运行在严格模式下，this指向undefined。

2、隐式绑定
```
function foo() {      
    console.log( this.a );
} 
 
var obj = {      
    a: 2,
    foo: foo  
}; 
 
obj.foo(); // 2
```
考虑规则是调用位置是否有上下文对象，或者说是否被某个对象拥有或者包含。   
在obj对象内部，包含了一个指向函数foo的属性，并通过这个属性间接引用函数，从而把this间接（隐式）绑定到obj对象上。

* 隐式丢失
被隐式绑定的函数会丢失绑定对象，从而会应用默认绑定，绑定到全局对象或者undefined。   
```
function foo() {      
    console.log( this.a ); 
} 
 
var obj = { 
    a: 2,
    foo: foo  
}; 
 
var bar = obj.foo; // 函数别名！  
var a = "oops, global"; // a 是全局对象的属性 
 
bar(); // "oops, global"
```
虽然 bar 是 obj.foo 的一个引用，但是实际上，它引用的是 foo 函数本身，因此此时的 bar() 其实是一个不带任何修饰的函数调用，因此应用了默认绑定。   
参数传递也会发生隐式丢失现象。   
```
function foo() {      
    console.log( this.a ); 
} 
 
function doFoo(fn) {     
    // fn 其实引用的是 foo 
 
    fn(); // <-- 调用位置！ 
} 
 
var obj = {     
     a: 2,     
     foo: foo  
}; 
 
var a = "oops, global"; // a 是全局对象的属性 
 
doFoo( obj.foo ); // "oops, global"

```
3、显示绑定（call、apply、bind）
```
function foo() {      
    console.log( this.a ); 
} 
 
var obj = { a:2 }; 
 
foo.call( obj ); // 2

```
通过 foo.call(..)，我们可以在调用 foo 时强制把它的 this 绑定到 obj 上。

4、new绑定
```
function foo(a) {      
    this.a = a; 
}  
 
var bar = new foo(2); 
console.log( bar.a ); // 2
```
使用 new 来调用 foo(..) 时，我们会构造一个新对象并把它绑定到 foo(..) 调用中的 this 上。new 是最后一种可以影响函数调用时 this 绑定行为的方法，我们称之为 new 绑定。

### 箭头函数(打破了以上this的绑定规则)
箭头函数继承外层函数调用的this绑定
```
function foo() {     
    // 返回一个箭头函数     
     return (a) => {         
         //this 继承自 foo()         
         console.log( this.a );      
     }; 
} 
 
var obj1 = {     
    a:2 
}; 
 
var obj2 = {      
    a:3 
}; 
 
var bar = foo.call( obj1 ); 
bar.call( obj2 ); // 2, 不是 3 ！

```
foo() 内部创建的箭头函数会捕获调用时foo()的 this。由于foo()的thi绑定到 obj1， bar（引用箭头函数）的this也会绑定到 obj1，箭头函数的绑定无法被修改。（new也不行！）。    
箭头函数常用于回调函数中，例如事件处理器或者定时器。
```
function foo() {      
    setTimeout(() => {         
        // 这里的 this 在此法上继承自 foo()         
        console.log( this.a );      
    },100); 
} 
 
var obj = { a:2 }; 
 
foo.call( obj ); // 2

```
箭头函数可以像 bind(..) 一样确保函数的 this 被绑定到指定对象，此外，其重要性还体现在 ***它用更常见的词法作用域取代了传统的 this 机制***。实际上，在ES6之前我们就已经在使用一种几乎和箭头函数完全一样的模式。   
```
function foo() {     
    var self = this; // lexical capture of this     
     setTimeout( function(){         
         console.log( self.a );     
     }, 100 ); 
}  
 
var obj = { a: 2 }; 
 
foo.call( obj ); // 2

```
bind模式：      
```
function foo() {     
    setTimeout( function(){         
        console.log( this.a );     
    }.bind(this), 100 ); 
}  
 
var obj = { a: 2 }; 
 
foo.call( obj ); // 2
```
虽然 self = this 和箭头函数看起来都可以取代 bind(..)，但是从本质上来说，它们想替代的是this机制。   
如果你经常编写 this 风格的代码，但是绝大部分时候都会使用 self = this 或者箭头函数来否定this机制，那你或许应当：   
* 只使用词法作用域并完全抛弃错误 this 风格的代码； 
* 完全采用 this 风格，在必要时使用 bind(..)，尽量避免使用 self = this 和箭头函数。

### 事件监听
```
btn.addEventListener('click' ,function handler(e){
  console.log(this);
})
```
this指向 e.currentTarget   

### 小结：   
如果要判断一个运行中函数的 this 绑定，就需要找到这个函数的直接调用位置。   
找到之后就可以顺序应用下面这四条规则来判断 this 的绑定对象。
1. 由 new 调用？绑定到新创建的对象。 
2. 由 call 或者 apply（或者 bind）调用？绑定到指定的对象。 
3. 由上下文对象调用？绑定到那个上下文对象。 
4. 默认：在严格模式下绑定到 undefined，否则绑定到全局对象。

PS: [https://stackoverflow.com/questions/3127429/how-does-the-this-keyword-work] 《你不知道的JavaScript》