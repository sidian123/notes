promise相当于一个代理，表示一个**异步**操作成功或失败的结果，即使不能立马获得promise表示的值，但可以为promise附上成功或失败的回调函数。

# 例子

```javascript
function myAsyncFunction(url) {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest();
    xhr.open("GET", url);
    xhr.onload = () => resolve(xhr.responseText);
    xhr.onerror = () => reject(xhr.statusText);
    xhr.send();
  });
}
```

`myAsyncFunction`函数返回一个promise，它代表xhr异步请求后的结果。在promise对象被创建后，它的`executor`函数（第一个参数）会立马被执行。因为这是异步请求，因此不能立马获得结果。但你可以附上处理函数，一般产生结果或失败时执行，如：

```javascript
myAsyncFunction("localhost/api/get")
.then(function(successData){
    console.log(succssData);
})
.catch(function(failueData){
    console.log(failueData);
});
```

当promise的executor函数中的异步结果出现后，即执行了`resolve(data)`，data为promise的结果，会执行`then`的第一个参数（函数），来处理成功的结果。

当`myAsyncFunction`函数中执行了`reject(data)`或抛出异常时，会执行`then`的第二个参数（函数），来处理异步操作的错误。`.catch(onRejected)`是`then.(undefined,onRejected)`的简写。

# Promise

```javascript
new Promise(executor)
```

当promise被创建时，executor函数会被立刻执行，executor函数将被传入`resolve`和`reject`方法。当executor中的异步操作结束后，需要调用`resolve`或`reject`方法表示成功或失败。一旦被调用，通过`then`方法附着在promise上的处理器(`then`的两个参数之一)会被调用。这两个函数(`resolve`和`reject`)执行时传入的参数会被处理器(同上)接收。

# then

```javascript
p.then(onFulfilled[, onRejected]);

p.then((value) => {
  // fulfillment
}, (reason) => {
  // rejection
});
```

参数：

- `onFulfilled`：promise的executor执行resolve后执行该处理器。方法的参数为resolve执行时传入的参数。
- `onRejected`；promise的executor执行reject后（或抛出异常）执行该处理器。方法的参数为reject执行时传入的参数（或异常）。

返回值：

返回一个promise对象，因此可以**链式**使用`then`方法。但此时先搞懂promise代表什么值，当处理器：

- 返回一个值时，promise表示该值

- 不返回值时，promise表示undefined
- 抛出异常时，promise表示失败了（reject）, 值为`error`的值
- 返回promise，该promise的最终结果，就是`then`返回的promise的最终结果。

> 貌似一个失败了（reject）的promise不被then的`onRejected`处理器处理时~~（即onRejected不存在）~~，会继续传给下一个。



# 参考

- [Using promises](<https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises>)
- [Promise](<https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise>)
- [then](<https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/then>)
- [catch](<https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/catch>)