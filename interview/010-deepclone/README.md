# 深拷贝

<motto></motto>

> 今天，来完成一个自己的深拷贝

``` js
let checkType = (data) => {
    return Object.prototype.toString.call(data).slice(8, -1);
}

let deepClone = (target, cache = new WeakMap()) => {
    // 首先先来判断一下要拷贝的数据是否为基本原始类型
    let targetType = checkType(target);

    let cloneResult;
    if (targetType === 'Object') {
        cloneResult = {}
    } else if (targetType === 'Array') {
        cloneResult = []
    } else {
        return target
    }

    // 避免循环引用
    if (cache.get(target)) {
        return cache.get(target);
    }
    cache.set(target, cloneResult);

    for (const key in target) {
      cloneResult[key] = (typeof target[key]) === 'object' ?
        deepClone(target[key], cache) : target[key]
    }

    return cloneResult;
}
```
