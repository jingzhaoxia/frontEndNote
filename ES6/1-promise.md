# Promise的好处
Promsie 可以解决的问题
1. 把你从回调地狱中解救出来
1. 让你优雅的捕获错误
1. 为你分担异步并发的难题

```javascript
// 此处使用node举例，不会不要紧，先混个脸熟。再见就不陌生了呀
let fs = require('fs');
// 异步读取文件
fs.readFile('./name', function (err, data){
    if(err){}
    fs.readFile(data, function (err, address){
        if(err){}
        fs.readFile(address, function (err, product){
            // 1）深陷在回调地域中不能抽身
            if(err){
                // 2）捕获错误。OMG，我哪里错了？！？！告诉我，肯定改~
            }
        });
    });
});

// 3） 你的名字、你的地址。都告诉我，惊喜才会送到你面前呀~
fs.readFile('./name', function (err, data){});
fs.readFile('./address', function (err, data){});
```

# Promise 使用的例子
## 状态变化
- Promise 有3种状态
>- pending 等待态
>- fulfilled 成功态
>- rejected 失败态

【等待态 -> 成功态】 or 【等待态 -> 失败态】二选一，你来定。

![](https://github.com/jingzhaoxia/frontEndCode/blob/master/zhaoxiajingjing/20190928/tu/tu.png)

- Promsie 是个类，接收一个函数参数 executor 执行器，一上来就执行了。这里是同步的。
- 每一个Promsie的实例上都有一个then方法。是基于回调实现的。

```javascript
console.log('一封情书');

let p = new Promise((resolve, reject)=>{
    console.log('executor请说出你的选择：');
    resolve('你中意我~(*^▽^*)');
    reject('你发了好人卡(╥﹏╥)o');
});

p.then((value)=>{
    console.log('成功态', value);
}, (reason) => {
    console.log('失败态', reason);
});

console.log('纸短情长');
```


## 链式调用

```javascript
console.log('-----一封情书-----');

let p = new Promise((resolve, reject) => {
    console.log('executor 请说出你的选择：');
    resolve('你中意我~(*^▽^*)');
    reject('你发了好人卡(╥﹏╥)o');
});

p.then((value) => {
    console.log('成功态---', value);
}, (reason) => {
    console.log('失败态---', reason);
}).then((value) => {
    console.log('---爱你一万年~(*^▽^*)');
}, (reason) => { 
    console.log('---伤心总是难免的o(╥﹏╥)o');
});
console.log('~~~纸短情长~~~');
```


### 异步请求
- name.txt
```javascript
zhaoxiajingjing
```

- 3.js
```javascript
let fs = require('fs');
let p = new Promise((resolve, reject)=>{
    fs.readFile('./name.txt', 'utf8', function (err, data){
        if(err){
            return reject(err);
        }
        resolve(data);
    });
});
p.then((value)=>{
    console.log('成功了', value);
}, (reason)=>{
    console.log('失败了', reason);
});
```

## 复杂的使用 Promise

下面的例子输出什么呢？

```javascript
let fs = require('fs');

function read(filePath) {
    return new Promise((resolve, reject) => {
        fs.readFile(filePath, 'utf8',function(err, data){
            if(err) {
                return reject(err);
            }
            resolve(data);
        });
    });
}

read('./name.txt')
// then-1
.then(function (data) {
    console.log('data①', data);
    return new Promise((resolve, reject) => {
        reject('错误了');
    });
})
// then-2
.then((data) => {
    console.log('data②', data);
}, err => {
    console.log('err②', err);
})
// then-3
.then((data) => {
    console.log('data③', data);
}, (err) => {
    console.log('err③', err);
});
```


** OK，提炼一下重点： **
1. 有三个状态。怎么变化的？
1. executor。怎么执行的？
1. then方法，成功态和失败态的回调。
1. then方法的链式调用。
1. Promise 处理异步。

# 实现以上内容
## Promise 基本实现
- new Promise 会传一个函数作为参数。这个函数有两个参数：resolve 成功 reject 失败，都是用于改变状态的。都在实例上，**一个 Promise 一生只改变一次状态**

- 每个实例上都有一个 then 方法，异步的。对于成功态和失败态的回调：onFulfilled onRejected

```javascript
const PENDING = 'pending';
const SUCCESS = 'fulfilled';
const FAIL = 'rejected';

class Promise{
    constructor(execuotr){
        const status = PENDING; // 我等着你给答案~
        this.value;
        this.reason;

        let resolve = (value)=>{
            if(status === PENDING) {
                this.status = SUCCESS; // 你中意我~
                this.value = value;
            }
        };

        let reject = (reason)=>{
            if(status === PENDING) {
                this.status = FAIL; // 你发了好人卡
                this.reason = reason;
            }
        };

        // 是同步的哦~
        try {
            executor(resolve, reject);
        } catch (e) {
            reject(e);
        }
    }
    then(onFulfilled, onRejected){
        if(this.status === SUCCESS) {
            onFulfilled(this.value);  // 爱你一万年~
        }
        if(this.status === FAIL) {
            onRejected(this.reason); // 伤心总是难免的
        }
    }
}
```

## Pomise 解决异步问题
Promise 是个容器，里面可以放一些异步的请求，请求成功了走成功态，请求失败了走失败态。当然，你要反过来走也可以哒~

```javascript
const PENDING = 'pending';
const SUCCESS = 'fulfilled';
const FAIL = 'rejected';

class Promise {
    constructor(executor){
        this.status = PENDING;
        this.value;
        this.reason;

        // 用来存储 订阅的内容的
        this.onSuccessCallbacks = [];
        this.onFailCallbacks = [];

        let resolve = (value)=>{
            if(this.status === PENDING) {
                this.status = SUCCESS;
                this.value = value;
                this.onSuccessCallbacks.forEach(fn => fn());
            }
        };
        let reject = (reason)=>{
            if(this.status === PENDING) {
                this.status = FAIL;
                this.reason = reason;
                this.onFailCallbacks.forEach(fn => fn());
            }
        };

        try {
            executor(resolve, reject);
        } catch (e) {
            reject(e);
        }
    }
    then(onFulfilled, onRejected){
        if(this.status === SUCCESS){
            onFulfilled(this.value);
        }
        if(this.status === FAIL){
            onRejected(this.reason);
        }
        // 当Promise里面有异步请求控制状态改变时，会先走到then方法里面
        if(this.status === PENDING) {
            this.onSuccessCallbacks.push(()=>{
                onFulfilled(this.value);
            });
            this.onFailCallbacks.push(()=>{
                onRejected(this.reason);
            });
        }
    }
}

```

Promise 里面有异步请求时候，会先走到 then方法里面了。此时，需要把成功态回调和失败态回调先存储起来，等到异步请求回来以后变更了状态，再触发执行。

## Promise 的链式
### then 方法 返回一个新的 Promise
Promise 一生只能改变一次状态。那么，Promise 的链式调用then方法，说明每次都会返回一个**新的Promise**。

```javascript
const SUCCESS = 'fulfilled';
const FAIL = 'rejected';
const PENDING = 'pending';

class Promise {
    constructor(executor) {
       // ... executor 代码
    }
    then(onFulfilled, onRejected) {
        let promise2;

        promise2 = new Promise((resolve, reject)=>{
            if (this.status === SUCCESS) {
                try { // 用 try catch 捕获同步的报错
                    // 成功态的回调的返回值 x
                    // 【问题】 如果 x 是一个promsie，那么需要取得x执行后的结果
                    let x = onFulfilled(this.value);
                    resolve(x);

                } catch (e) {
                    reject(e);
                }
            }
            if (this.status === FAIL) {
                try {
                    let x = onRejected(this.reason);
                    resolve(x);
                } catch (e) {
                    reject(e);
                }
            }
            if (this.status === PENDING) {
                this.onSuccessCallbacks.push(()=>{
                    try {
                        let x = onFulfilled(this.value);
                        resolve(x);
                    } catch (e) {
                        reject(e);
                    }
                });
                this.onFailCallbacks.push(()=>{
                    try {
                        let x = onRejected(this.reason);
                        resolve(x);
                    } catch (e) {
                        reject(e);
                    }
                });
            }
        });

        return promise2;
    }
}
```

### 解析 x
如何判断 x 是Promsie，还是一个普通值？【参考规范 https://promisesaplus.com "Promsie A+"  2.2.7】

```javascript
const SUCCESS = 'fulfilled';
const FAIL = 'rejected';
const PENDING = 'pending';

function resolvePromise(promise2, x, resolve, reject) {
    // 死循环了，容错
    if(promise2 === x) {
        return reject('TypeError: Chaining cycle detected for promise~~~~');
    }
    // 判断 x 类型
    if(typeof x === 'function' || (typeof x === 'object' && x != null)) {
        // 这个才有可能是 promsie
        try {
            let then = x.then;
            if(typeof then === 'function') {
                // 此时，认为就是个promise
                // 如果promsie是成功的，那么结果向下传递，如果失败了就传到下一个失败里面去
                then.call(x, y=>{
                    resolvePromise(promise2, y, resolve, reject);
                }, r => {
                    reject(r);
                });
            } else {
                resolve(x);
            }
        } catch (e) {
            reject(e);
        }
    } else {
        // x 肯定不是一个promsie
        resolve(x);
    }
}

class Promise {
    constructor(executor) {
        // ... executor 代码
    }
    then(onFulfilled, onRejected) {
        let promise2;

        promise2 = new Promise((resolve, reject) => {
            if (this.status === SUCCESS) {
                // 用定时器模拟此时promise2已经能获取到了
                setTimeout(() => {
                    try {
                        let x = onFulfilled(this.value);
                        resolvePromise(promise2, x, resolve, reject);
                    } catch (e) {
                        reject(e);
                    }
                });
            }
            // 其他情况同理，先以一个为例说明
        });
        
        return promise2;
    }
}
```

### 严谨度
**Promsie 给你的承诺，一句情话：一个 Promise 一生只改变一次状态**

```javascript
const SUCCESS = 'fulfilled';
const FAIL = 'rejected';
const PENDING = 'pending';

function resolvePromise(promise2, x, resolve, reject) {
    // 死循环了，容错
    if(promise2 === x) {
        return reject('TypeError: Chaining cycle detected for promise~~~~');
    }
    let called;
    if(typeof x === 'function' || (typeof x === 'object' && x != null)) {
        try {
            let then = x.then;
            if(typeof then === 'function') {
                then.call(x, y=>{
                    if(called) return;
                    called = true;
                    resolvePromise(promise2, y, resolve, reject);
                }, r => {
                    if(called) return;
                    called = true;
                    reject(r);
                });
            } else {
                resolve(x);
            }
        } catch (e) {
            if(called) return;
            called = true;
            reject(e);
        }
    } else {
        resolve(x);
    }
}

class Promise {
    constructor(executor) {
        // .... executor 的代码
    }
    then(onFulfilled, onRejected) {
        let promise2;

        promise2 = new Promise((resolve, reject) => {
            if (this.status === SUCCESS) {
                // 用定时器模拟此时promise2已经能获取到了
                setTimeout(() => {
                    try {
                        let x = onFulfilled(this.value);
                        resolvePromise(promise2, x, resolve, reject);
                    } catch (e) {
                        reject(e);
                    }
                });
            }
            // 其他情况同理，先以一个为例说明
        });

        return promsie2;
    }
}
```

### 值的穿透

```javascript
let p = new Promise((resolve, reject)=>{
    resolve(1000);
});
p.then().then().then(data => {
    console.log(data);
});
```
值可以传过去
```javascript
const SUCCESS = 'fulfilled';
const FAIL = 'rejected';
const PENDING = 'pending';

function resolvePromise(promise2, x, resolve, reject) {
    // ... 判断 x 的值
}

class Promise {
    constructor(executor) {
        // ... executor 代码
    }
    then(onFulfilled, onRejected) {
        // 值穿透
        onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : val => val;
        onRejected = typeof onRejected === 'function' ? onRejected : err => {throw err};
        // ... promsie2 的判断
    }
}

```

## 测试

- 测试这个库是否符合我们的promise A+ 规范
- `promises-aplus-tests` 用来测试当前的库是否符合规范
- `npm i promises-aplus-tests -g`
- `promises-aplus-tests 文件名`


## Promise A+ 规范
Promsie 是一个构造函数，是个类。默认高版本浏览器，node都自带了。不用考虑兼容性，放心大胆的使用吧！如果真不兼容，那就用es6-promsie包自己是一套吧~

Promsie A+

https://promisesaplus.com


## 容错
上面的内容，还需要一部分容错。就是当executor 里面的有一个promsie的时候，执行的结果。

```javascript
let Promise = require('./promise.js');
let p = new Promise((resolve, reject)=>{
    resolve(new Promise((resolve, reject)=>{
        reject(404);
    }));
});
p.then(value => console.log(1, value), reason => console.log(2, reason));

// 输出：
// 2 404
```

在同步执行时，resolve 的value是一个Promsie，那么需要等它的结果。

```javascript
const SUCCESS = 'fulfilled';
const FAIL = 'rejected';
const PENDING = 'pending';

function resolvePromise(promise2, x, resolve, reject) {
    // ... 校验x
}

class Promise {
    constructor(executor) {
        // ... code

        let resolve = (value) => {
            if (value instanceof Promise) {
                return value.then(resolve, reject);
            }

            if (this.status === PENDING) {
                this.status = SUCCESS;
                this.value = value;
                this.onSuccessCallbacks.forEach(fn => fn());
            }
        };
        // ... code
    }
    then(onFulfilled, onRejected) {
        // ... then 方法
    }
}

module.exports = Promise;
```



