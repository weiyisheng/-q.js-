## 一些链接
翻译自 https://github.com/kriskowal/q/tree/v1/design, 原作者 @kriskowal。

周边：
[Are there still reasons to use promise libraries like Q or BlueBird now that we have ES6 promises? [closed]
](https://stackoverflow.com/questions/34960886/are-there-still-reasons-to-use-promise-libraries-like-q-or-bluebird-now-that-we)


## 异步的问题

在Node.js还没有native support for promises的时候，社区有很多的Promise libraries，[q.js](https://github.com/kriskowal/q)就是其中一个。作者在这篇文章中深入浅出地阐述了q.js的设计思路，希望能给读者一些启发。
<hr />

Promise解决的是js的异步问题，所以我们从最本质、最简单的需求出发：一个function不能立马返回value，最直接的办法就是用一个callback把最总的value传出来：
```javascript
var oneOneSecondLater = function (callback) {
    setTimeout(function () {
        callback(1);
    }, 1000);
};
```
在此基础上，我们可以加入异常处理，让它本身更完善：
```javascript
var maybeOneOneSecondLater = function (callback, errback) {
    setTimeout(function () {
        if (NO ERROR) {
            callback(1);
        } else {
            errback(new Error("SOME ERROR MESSAGE"));
        }
    }, 1000);
};
```
或者也可以把ERROR当作callback的一个参数传出去，总之有很多办法实现这一个简单异步的需求。但是这里有一个很严重的问题，我们要明白exceptions和try/catch的目的是能够推迟异常的处理，换句话说，就是能够将异常一层层的往外抛出，直到被catch住。那我们这里直接throw error 行不行呢？这个问题后面解答。
这里除了抛异常的问题没有得到解决，还有其他的一些问题，比如当异步嵌套很多的时候，就会出现callback中嵌套callbak，不断嵌套的情况等等。

## Promise
另一种设计思路，不再去关注一定要function返回value或抛出异常，而是让function返回一个object，我们可以通过这个object获取到function最终的执行结果，不管成功还是失败。这个object就被称为“promise”，表示它一定会交给你一个结果，成功或失败。
```javascript
var maybeOneOneSecondLater = function () {
    var callback;
    setTimeout(function () {
        callback(1);
    }, 1000);
    return {
        then: function (_callback) {
            callback = _callback;
        }
    };
};

maybeOneOneSecondLater().then(callback);
```
我们可以通过then方法注册callback。现在
