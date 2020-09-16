---
title: 防抖与节流
date: 2020-09-14 16:55:48
tags:
  - JavaScript
comments: true
---

## 防抖

 防抖的原理就是：

​		在事件触发后n秒才执行，在这期间若再次触发该事件，重新刷新时间，以再次触发的时间为基准，再次经过n秒后才执行总之，就是要等触发完事件 n 秒内不再触发事件，才执行 

我们写个 `index.html` 文件：

```html
<!DOCTYPE html>
<html lang="zh-cmn-Hans">

<head>
    <meta charset="utf-8">
    <meta http-equiv="x-ua-compatible" content="IE=edge, chrome=1">
    <title>debounce</title>
    <style>
        #container{
            width: 100%; height: 200px; line-height: 200px; text-align: center; color: #fff; background-color: #444; font-size: 30px;
        }
    </style>
</head>

<body>
    <div id="container"></div>
    <script src="debounce.js"></script>
</body>

</html>
```

`debounce.js` 文件的代码如下：

```js
var count = 1;
var container = document.getElementById('container');

function getUserAction() {
    container.innerHTML = count++;
};

container.onmousemove = getUserAction;
```



### 第一版

利用clearTimeout函数和setTimeout函数实现

```js
var count = 1;
var container = document.getElementById('container');

function getUserAction() {
    container.innerHTML = count++;
};
function debounce(func, wait) {
    var timeout;
    return function () {
        clearTimeout(timeout) // 再次触发时，清除原先的定时器
        timeout = setTimeout(func, wait); // 重置定时器
    }
}
container.onmousemove = debounce(getUserAction, 1000);
```



### 第二版

使this指针指向正确的对象

如果我们在 `getUserAction` 函数中 `console.log(this)`，在不使用 `debounce` 函数的时候，`this` 的值为：

```html
<div id="container"></div>
```

但是如果使用我们的 debounce 函数，this 就会指向 Window 对象！

所以我们需要将 this 指向正确的对象。

```js
function debounce(func, wait) {
    var timeout
    return function() {
        // var context = this
        clearTimeout(timeout)
        timeout = setTimeout(() => {
            func.apply(this)
        }, wait)
    }
}
```



### 第三版

JavaScript 在事件处理函数中会提供事件对象 event，我们修改下 getUserAction 函数：

```
function getUserAction(e) {
    console.log(e);
    container.innerHTML = count++;
};
```

如果我们不使用 debouce 函数，这里会打印 MouseEvent 对象，如图所示：

[![MouseEvent](https://github.com/mqyqingfeng/Blog/raw/master/Images/debounce/event.png)](https://github.com/mqyqingfeng/Blog/raw/master/Images/debounce/event.png)

但是在我们实现的 debounce 函数中，却只会打印 undefined!

```js
function debounce(func, wait) {
    var timeout
    return function() {
        // var context = this
        var args = arguments
        clearTimeout(timeout)
        timeout = setTimeout(() => {
            func.apply(this, args)
        }, wait)
    }
}
```



### 第四版

我们接下来思考一个新的需求。

这个需求就是：

我不希望非要等到事件停止触发后才执行，我希望立刻执行函数，然后等到停止触发 n 秒后，才可以重新触发执行。

~~~js
function debounce(func, wait, immediate) {
    var timeout
    return function() {
        // var context = this
        var args = arguments
        if(timeout) clearTimeout(timeout)
        if(immediate) {
            let callNow = !timeout
            timeout = setTimeout(() => {
                timeout = null
            }, wait)
            if(callNow) func.apply(this, args)
        } else {
            timeout = setTimeout(() => {
                func.apply(this, args)
            }, wait)   
        }
    }
}
~~~

 



## 节流

如果持续触发事件，每隔一段时间，只执行一次事件

根据首次是否执行以及结束后是否执行，效果有所不同，实现的方式也有所不同。

主要有两种主流的实现方式：1. 使用时间戳  2. 设置定时器

### 使用时间戳

当触发事件的时候，取出当前的事件戳，然后减去之前的时间戳（初始值设为0），如果大于设置的时间周期，就执行函数，然后更新时间戳为当前的时间戳，如果小于，就不执行。

```js
function throttle(func, wait) {
    let previous = 0
    let args
    return function() {
        let now = new Date()
        args = arguments
        if(now - previous > wait) {
            func.apply(this, args)
            previous = now
        }
    }
}
```



### 使用定时器

当触发事件的时候，我们设置一个定时器，再触发事件的时候，如果定时器存在，就不执行，直到定时器执行，然后执行函数，清空定时器，这样就可以设置下个定时器。 

```js
function throttle(func, wait) {
    let args, context
    let timeout = null
    return function() {
        args = arguments
        context = this
        if(!timeout) {
            timeout = setTimeout(()=>{
                timeout = null
                func.apply(context, args)
            }, wait)
        } 
    }
}
```



所以比较两个方法：

1. 第一种事件会立刻执行，第二种事件会在 n 秒后第一次执行
2. 第一种事件停止触发后没有办法再执行事件，第二种事件停止触发后依然会再执行一次事件



### 综合

触发时能立即执行，停止触发后还能执行一次

```js
function throttle(func, wait) {
    let timeout, context, args, result
    let previous = 0
    let later = function() {
        previous = +new Date()
        timeout = null
        func.apply(context, args)
    }
    return throttled = function() {
        var now = +new Date();
        //下次触发 func 剩余的时间
        var remaining = wait - (now - previous);
        context = this;
        args = arguments;
         // 如果没有剩余的时间了或者你改了系统时间
        if (remaining <= 0 || remaining > wait) {
            if (timeout) {
                clearTimeout(timeout);
                timeout = null;
            }
            previous = now;
            func.apply(context, args);
        } else if (!timeout) {
            timeout = setTimeout(later, remaining);
        }
    };
}
```



### 优化

那我们设置个 options 作为第三个参数，然后根据传的值判断到底哪种效果，我们约定:

leading: false 表示禁用第一次执行
trailing: false 表示禁用停止触发的回调

我们来改一下代码：

```js
// 第四版
function throttle(func, wait, options) {
    var timeout, context, args, result;
    var previous = 0;
    if (!options) options = {};

    var later = function() {
        previous = options.leading === false ? 0 : new Date().getTime();
        timeout = null;
        func.apply(context, args);
        if (!timeout) context = args = null;
    };

    var throttled = function() {
        var now = new Date().getTime();
        if (!previous && options.leading === false) previous = now;
        var remaining = wait - (now - previous);
        context = this;
        args = arguments;
        if (remaining <= 0 || remaining > wait) {
            if (timeout) {
                clearTimeout(timeout);
                timeout = null;
            }
            previous = now;
            func.apply(context, args);
            if (!timeout) context = args = null;
        } else if (!timeout && options.trailing !== false) {
            timeout = setTimeout(later, remaining);
        }
    };
    return throttled;
}
```


