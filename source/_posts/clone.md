---
title: JS深浅拷贝
date: 2020-09-15 19:29:19
tags:
- JavaScript
---
## JavaScript深浅拷贝

JavaScript在对于引用数据类型进行赋值时，如果直接进行赋值，两者实际访问的是同一个内存空间中的值，如下：

```js
let arr = [1,2,3]
let newArr = arr
newArr.push(5)
console.log(arr, newArr) // 输出均为[1,2,3,5]
```

因此，我们需要对数据进行拷贝来实现复制。

JavaScript分为浅拷贝和深拷贝

所谓浅拷贝，是将基本类型元素拷贝，互不影响，而引用类型元素只拷贝其引用，在进行修改时，两者还是都会改变

而深拷贝则是完全的克隆一个新的对象

### 浅拷贝的实现

如果是数组，可以利用数组的一些方法，slice、concat返回一个新数组来实现拷贝

自己实现一个浅拷贝方法

~~~js
function shallowCopy(obj) {
    // 判断obj是否为一个对象
    if(typeof obj !== 'object') return 'obj is not an object!'

    // 判断传入的obj是数组还是对象
    let newObj = Array.isArray(obj) ? [] : {}

    // 将obj中的每项数据复制
    for(let key in obj) {
        if(obj.hasOwnProperty(key)) {
            newObj[key] = obj[key]
        }
    }
    return newObj
}
~~~

### 深拷贝的实现

利用`JSON.parse(JSON.stringfy(arr))`可以进行深拷贝，但是不能拷贝函数

自己实现一个深拷贝方法

在拷贝的时候判断一下数据类型，如果是对象，递归调用拷贝函数

```js
function deepCopy(obj) {
    // 判断obj是否为一个对象
    if(typeof obj !== 'object') return 'obj is not an object!'

    // 判断传入的obj是数组还是对象
    let newObj = Array.isArray(obj) ? [] : {}

    // 将obj中的每项数据复制
    for(let key in obj) {
        if(obj.hasOwnProperty(key)) {
            newObj[key] = typeof obj[key] === 'object' ? deepCopy(obj[key]) : obj[key]
        }
    }
    return newObj
}
```

