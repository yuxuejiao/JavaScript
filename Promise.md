## Promise
Promise对象用于表示一个异步操作的最终状态（完成或失败），以及该异步操作的结果值。   
它有以下三种状态：
* pending: 初始值，不是fulfilled，也不是rejected
* fulfilled: 操作成功
* rejected: 操作失败   
promise可以从pending->fulfilled，也可以从pending->rejected。一旦状态改变，就不再改变。当状态发生变化，then绑定的函数就会被调用。
### 构建promise对象
```
var p = new Promise(function(resolve, reject){
    // 异步操作
    if(异步操作成功){
        resolve(data); //改变promise对象的状态，从而调用then中绑定的处理方法 这里的resolve并不是异步成功的调用函数
    }else{
        reject(error); //该表promise对象的状态，从而调用then中绑定的处理方法
    }
} /* executor函数 */)

使用new + Promise构造函数来创建对象，传参为executor函数（处理器函数）。Promise构造函数执行时，立即调用executor函数。
executor函数有两个传参resolve函数和reject函数。resolve和reject函数被调用时，分别将promise的状态从pending改为fulfilled或rejected。
executor 内部通常会执行一些异步操作，一旦异步操作执行完毕(可能成功/失败)，要么调用resolve函数来将promise状态改成fulfilled，要么调用reject函数将promise的状态改为rejected。如果在executor函数中抛出一个错误，那么该promise 状态为rejected。executor函数的返回值被忽略。
```
### .then方法
```
p.then(function(data){
    // onFulfilled函数
}, function(error){
    // onRejected函数
})

当在executor函数中执行resolve或reject函数，改变了promise对象的状态。此时会调用then中绑定的相应处理方法。
then方法为当前的promise对象绑定了resolve和reject函数，返回一个新的promise，将以回调的返回值来resolve。
```
### 链式调用中then方法回调函数的返回值
* 如果then中的回调函数返回一个值，那么then返回的Promise将会成为接受状态，并且将返回的值作为接受状态的回调函数的参数值。
* 如果then中的回调函数没有返回值，那么then返回的Promise将会成为接受状态，并且该接受状态的回调函数的参数值为 undefined。
* 如果then中的回调函数抛出一个错误，那么then返回的Promise将会成为拒绝状态，并且将抛出的错误作为拒绝状态的回调函数的参数值。
* 如果then中的回调函数返回一个已经是接受状态的Promise，那么then返回的Promise也会成为接受状态，并且将那个Promise的接受状态的回调函数的参数值作为该被返回的      Promise的接受状态回调函数的参数值。
* 如果then中的回调函数返回一个已经是拒绝状态的Promise，那么then返回的Promise也会成为拒绝状态，并且将那个Promise的拒绝状态的回调函数的参数值作为该被返回的      Promise的拒绝状态回调函数的参数值。
* 如果then中的回调函数返回一个未定状态（pending）的Promise，那么then返回Promise的状态也是未定的，并且它的终态与那个Promise的终态相同；同时，它变为终态时调用   的回调函数参数与那个Promise变为终态时的回调函数的参数是相同的。

### .resolve方法
```
Promise.resolve(value/promise/thenable);

Promise.resolve('success');
等价于
new Promise(function(resolve){resolve('success')});

Promise.resolve('success').then(function(value){
    console.log(value);
})
console.log('111');

// 先输出 111， 再输出 success

Promise.resolve可以接收除promise对象之外的简单值，带有then方法的对象这些值。返回的是一个新的promise对象，因此可以用.then方法。 
Promise.resolve会立即进入resolved状态，但是then还是异步输出。

```
### .reject方法
```
Promise.reject(reason);

Promise.reject(new Error('error'))
等价于
new Promise(function(resolve,reject){reject('error')})

Promise.reject返回的是一个新的promise对象，会立即进入rejected状态。
```
### .finally方法
返回一个新的promise对象，无论当前promise对象的状态是resolved还是rejected。
### .catch方法
```
promise.then(function(data) {
    console.log('success');
}).catch(function(error) {
    console.log('error', error);
});
等同于
promise.then(function(data) {
    console.log('success');
}).then(undefined, function(error) {
    console.log('error', error);
});

.catch相当于.then(undefined, onRejected)

var promise = new Promise(function (resolve, reject) {
    throw new Error('test');
});
等同于
var promise = new Promise(function (resolve, reject) {
    reject(new Error('test'));
});
//用catch捕获
promise.catch(function (error) {
    console.log(error);
});

reject方法的作用，相当于抛错。
promise对象的错误，会一直向后传递，直到被捕获。即错误总会被下一个catch所捕获。then方法指定的回调函数，若抛出错误，也会被下一个catch捕获。catch中也能抛错，则需要后面的catch来捕获。
如果没有使用catch方法指定处理错误的回调函数，Promise对象抛出的错误不会传递到外层代码，即不会有任何反应（Chrome会抛错)。

```
### reject和catch的区别
* promise.then(onFulfilled, onRejected)
    在onFulfilled中发生异常的话，在onRejected中是捕获不到这个异常的。因为onRejected错误处理函数是为promise对象准备的。
* promise.then(onFulfilled).catch(onRejected)
    .then中产生的异常能在.catch中捕获

### .all方法
var p = Promise.all([p1, p2, p3]);   
将多个promise实例，包装成一个新的promise实例。多个promise并行执行。   
Promise.all方法接受一个数组（或具有Iterator接口）作参数，数组中的对象（p1、p2、p3）均为promise实例（如果不是一个promise，该项会被用Promise.resolve转换为一个promise)。它的状态由这三个promise实例决定。
* 当p1, p2, p3状态都变为fulfilled，p的状态才会变为fulfilled，并将三个promise返回的结果，按参数的顺序（而不是 resolved的顺序）存入数组，传给p的回调函数。
* 当p1, p2, p3其中之一状态变为rejected，p的状态也会变为rejected，并把第一个被reject的promise的返回值，传给p的回调函数。

### .race方法
var p = Promise.race([p1, p2, p3]);      
该方法同样是将多个Promise实例，包装成一个新的Promise实例。      
Promise.race方法同样接受一个数组（或具有Iterator接口）作参数。当p1, p2, p3中有一个实例的状态发生改变（变为fulfilled或rejected），p的状态就跟着改变。并把*第一个*改变状态的promise的返回值，传给p的回调函数。   
在第一个promise对象变为resolve后，并不会取消其他promise对象的执行。  

### then中的抛错，没有对错误进行catch，则会一直保持reject状态
```
function taskA() {
    console.log(x);
    console.log("Task A");
}
function taskB() {
    console.log("Task B");
}
function onRejected(error) {
    console.log("Catch Error: A or B", error);
}
function finalTask() {
    console.log("Final Task");
}
var promise = Promise.resolve();
promise
    .then(taskA)
    .then(taskB)
    .catch(onRejected)
    .then(finalTask);
    
-------output-------
Catch Error: A or B,ReferenceError: x is not defined
Final Task
```
### promise状态从pending变为resove或reject，就不会再改变
```
var promise = new Promise(function(resolve, reject) {
  resolve();
  throw 'error'; // not called
});

promise.catch(function(e) {
   console.log(e);      //not called
});

promise状态一旦改变了，就不会再发生变化。因为promise resolve了，后面再抛错，也不会进入catch方法。
```

### promise vs 回调函数
* 回调层层嵌套，promise链式调用，将异步操作以同步操作流程表示，易于编程和理解。
* promise一定是异步的，无论其状态变更是同步还是异步的，then中绑定的回调一定是在同段代码中的同步代码执行后调用。从而调用的时间比较可控，不存在回调函数中回调时间过早还是过晚的问题。
* promise一定会执行且执行一次。而回调函数不一定会执行，并且有可能会多次执行。
