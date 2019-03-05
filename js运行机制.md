### js的运行机制
宏任务：宿主发起的任务，整体代码script，setTimeout，setInterval    
微任务：js引擎发起的任务，promise中的异步代码，process.nextTick（node）    
注：宏任务和微任务与同步任务和异步任务是两个维度的概念。   

当执行一段js代码(script)，相当于执行一个宏任务（也可以说是一次事件循环）：   
-  遇到同步任务，进入主线程，当主线程中的任务执行完毕，会去读取任务队列（Event Queue）中的结果，进入主线程执行      
-  遇到异步任务，进入Event Table并注册函数，当指定的事情完成时，将回调函数移到Event Queue。   

确定执行顺序的思路：
- 1、先将整个script作为第一个宏任务进入主线程，遇到同步代码，则执行输出   
- 2、然后将其中的宏任务按照注入到Event Queue中的顺序（不是Event Table，不能只是简单只看在代码中出现的顺序）进行排序，
放入宏任务队列。微任务同理按照顺序放入微任务队列。   
- 3、当主线程中的同步代码执行完毕，按照微任务队列优先于宏任务执行的原则，先执行微任务队列中的代码，执行顺序先进先出。   
- 4、微任务队列中的内容执行完毕后，执行宏任务队列中的内容。总的顺序也是先进先出。当宏任务中包含了微任务，如setTimeout
中包含了promise的异步，先执行该宏任务中的代码，再执行其中的异步代码。然后再执行下一个顺序的宏任务。

eg:     
```
	console.log('1');

	setTimeout(function() {
		console.log('2');
		process.nextTick(function() {
			console.log('3');
		})
		new Promise(function(resolve) {
			console.log('4');
			resolve();
		}).then(function() {
			console.log('5')
		})
	})
	process.nextTick(function() {
		console.log('6');
	})
	new Promise(function(resolve) {
		console.log('7');
		resolve();
	}).then(function() {
		console.log('8')
	})

	setTimeout(function() {
		console.log('9');
		process.nextTick(function() {
			console.log('10');
		})
		new Promise(function(resolve) {
			console.log('11');
			resolve();
		}).then(function() {
			console.log('12')
		})
	})
```

输出结果： 1，7，6，8，2，4，3，5，9，11，10，12

[参照文章]  https://segmentfault.com/a/1190000018227028 

