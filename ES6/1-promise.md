# Promise�ĺô�
Promsie ���Խ��������
1. ����ӻص������н�ȳ���
1. �������ŵĲ������
1. Ϊ��ֵ��첽����������

```javascript
// �˴�ʹ��node���������᲻Ҫ�����Ȼ�����졣�ټ��Ͳ�İ����ѽ
let fs = require('fs');
// �첽��ȡ�ļ�
fs.readFile('./name', function (err, data){
    if(err){}
    fs.readFile(data, function (err, address){
        if(err){}
        fs.readFile(address, function (err, product){
            // 1�������ڻص������в��ܳ���
            if(err){
                // 2���������OMG����������ˣ������������ң��϶���~
            }
        });
    });
});

// 3�� ������֡���ĵ�ַ���������ң���ϲ�Ż��͵�����ǰѽ~
fs.readFile('./name', function (err, data){});
fs.readFile('./address', function (err, data){});
```

# Promise ʹ�õ�����
## ״̬�仯
- Promise ��3��״̬
>- pending �ȴ�̬
>- fulfilled �ɹ�̬
>- rejected ʧ��̬

���ȴ�̬ -> �ɹ�̬�� or ���ȴ�̬ -> ʧ��̬����ѡһ����������


- Promsie �Ǹ��࣬����һ���������� executor ִ������һ������ִ���ˡ�������ͬ���ġ�
- ÿһ��Promsie��ʵ���϶���һ��then�������ǻ��ڻص�ʵ�ֵġ�

```javascript
console.log('һ������');

let p = new Promise((resolve, reject)=>{
    console.log('executor��˵�����ѡ��');
    resolve('��������~(*^��^*)');
    reject('�㷢�˺��˿�(�i�n�i)o');
});

p.then((value)=>{
    console.log('�ɹ�̬', value);
}, (reason) => {
    console.log('ʧ��̬', reason);
});

console.log('ֽ���鳤');
```


## ��ʽ����

```javascript
console.log('-----һ������-----');

let p = new Promise((resolve, reject) => {
    console.log('executor ��˵�����ѡ��');
    resolve('��������~(*^��^*)');
    reject('�㷢�˺��˿�(�i�n�i)o');
});

p.then((value) => {
    console.log('�ɹ�̬---', value);
}, (reason) => {
    console.log('ʧ��̬---', reason);
}).then((value) => {
    console.log('---����һ����~(*^��^*)');
}, (reason) => { 
    console.log('---�������������o(�i�n�i)o');
});
console.log('~~~ֽ���鳤~~~');
```


### �첽����
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
    console.log('�ɹ���', value);
}, (reason)=>{
    console.log('ʧ����', reason);
});
```

## ���ӵ�ʹ�� Promise

������������ʲô�أ�

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
    console.log('data��', data);
    return new Promise((resolve, reject) => {
        reject('������');
    });
})
// then-2
.then((data) => {
    console.log('data��', data);
}, err => {
    console.log('err��', err);
})
// then-3
.then((data) => {
    console.log('data��', data);
}, (err) => {
    console.log('err��', err);
});
```


** OK������һ���ص㣺 **
1. ������״̬����ô�仯�ģ�
1. executor����ôִ�еģ�
1. then�������ɹ�̬��ʧ��̬�Ļص���
1. then��������ʽ���á�
1. Promise �����첽��

# ʵ����������
## Promise ����ʵ��
- new Promise �ᴫһ��������Ϊ�������������������������resolve �ɹ� reject ʧ�ܣ��������ڸı�״̬�ġ�����ʵ���ϣ�**һ�� Promise һ��ֻ�ı�һ��״̬**

- ÿ��ʵ���϶���һ�� then �������첽�ġ����ڳɹ�̬��ʧ��̬�Ļص���onFulfilled onRejected

```javascript
const PENDING = 'pending';
const SUCCESS = 'fulfilled';
const FAIL = 'rejected';

class Promise{
    constructor(execuotr){
        const status = PENDING; // �ҵ��������~
        this.value;
        this.reason;

        let resolve = (value)=>{
            if(status === PENDING) {
                this.status = SUCCESS; // ��������~
                this.value = value;
            }
        };

        let reject = (reason)=>{
            if(status === PENDING) {
                this.status = FAIL; // �㷢�˺��˿�
                this.reason = reason;
            }
        };

        // ��ͬ����Ŷ~
        try {
            executor(resolve, reject);
        } catch (e) {
            reject(e);
        }
    }
    then(onFulfilled, onRejected){
        if(this.status === SUCCESS) {
            onFulfilled(this.value);  // ����һ����~
        }
        if(this.status === FAIL) {
            onRejected(this.reason); // �������������
        }
    }
}
```

## Pomise ����첽����
Promise �Ǹ�������������Է�һЩ�첽����������ɹ����߳ɹ�̬������ʧ������ʧ��̬����Ȼ����Ҫ��������Ҳ������~

```javascript
const PENDING = 'pending';
const SUCCESS = 'fulfilled';
const FAIL = 'rejected';

class Promise {
    constructor(executor){
        this.status = PENDING;
        this.value;
        this.reason;

        // �����洢 ���ĵ����ݵ�
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
        // ��Promise�������첽�������״̬�ı�ʱ�������ߵ�then��������
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

Promise �������첽����ʱ�򣬻����ߵ� then���������ˡ���ʱ����Ҫ�ѳɹ�̬�ص���ʧ��̬�ص��ȴ洢�������ȵ��첽��������Ժ�����״̬���ٴ���ִ�С�

## Promise ����ʽ
### then ���� ����һ���µ� Promise
Promise һ��ֻ�ܸı�һ��״̬����ô��Promise ����ʽ����then������˵��ÿ�ζ��᷵��һ��**�µ�Promise**��

```javascript
const SUCCESS = 'fulfilled';
const FAIL = 'rejected';
const PENDING = 'pending';

class Promise {
    constructor(executor) {
       // ... executor ����
    }
    then(onFulfilled, onRejected) {
        let promise2;

        promise2 = new Promise((resolve, reject)=>{
            if (this.status === SUCCESS) {
                try { // �� try catch ����ͬ���ı���
                    // �ɹ�̬�Ļص��ķ���ֵ x
                    // �����⡿ ��� x ��һ��promsie����ô��Ҫȡ��xִ�к�Ľ��
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

### ���� x
����ж� x ��Promsie������һ����ֵͨ�����ο��淶 https://promisesaplus.com "Promsie A+"  2.2.7��

```javascript
const SUCCESS = 'fulfilled';
const FAIL = 'rejected';
const PENDING = 'pending';

function resolvePromise(promise2, x, resolve, reject) {
    // ��ѭ���ˣ��ݴ�
    if(promise2 === x) {
        return reject('TypeError: Chaining cycle detected for promise~~~~');
    }
    // �ж� x ����
    if(typeof x === 'function' || (typeof x === 'object' && x != null)) {
        // ������п����� promsie
        try {
            let then = x.then;
            if(typeof then === 'function') {
                // ��ʱ����Ϊ���Ǹ�promise
                // ���promsie�ǳɹ��ģ���ô������´��ݣ����ʧ���˾ʹ�����һ��ʧ������ȥ
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
        // x �϶�����һ��promsie
        resolve(x);
    }
}

class Promise {
    constructor(executor) {
        // ... executor ����
    }
    then(onFulfilled, onRejected) {
        let promise2;

        promise2 = new Promise((resolve, reject) => {
            if (this.status === SUCCESS) {
                // �ö�ʱ��ģ���ʱpromise2�Ѿ��ܻ�ȡ����
                setTimeout(() => {
                    try {
                        let x = onFulfilled(this.value);
                        resolvePromise(promise2, x, resolve, reject);
                    } catch (e) {
                        reject(e);
                    }
                });
            }
            // �������ͬ������һ��Ϊ��˵��
        });
        
        return promise2;
    }
}
```

### �Ͻ���
**Promsie ����ĳ�ŵ��һ���黰��һ�� Promise һ��ֻ�ı�һ��״̬**

```javascript
const SUCCESS = 'fulfilled';
const FAIL = 'rejected';
const PENDING = 'pending';

function resolvePromise(promise2, x, resolve, reject) {
    // ��ѭ���ˣ��ݴ�
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
        // .... executor �Ĵ���
    }
    then(onFulfilled, onRejected) {
        let promise2;

        promise2 = new Promise((resolve, reject) => {
            if (this.status === SUCCESS) {
                // �ö�ʱ��ģ���ʱpromise2�Ѿ��ܻ�ȡ����
                setTimeout(() => {
                    try {
                        let x = onFulfilled(this.value);
                        resolvePromise(promise2, x, resolve, reject);
                    } catch (e) {
                        reject(e);
                    }
                });
            }
            // �������ͬ������һ��Ϊ��˵��
        });

        return promsie2;
    }
}
```

### ֵ�Ĵ�͸

```javascript
let p = new Promise((resolve, reject)=>{
    resolve(1000);
});
p.then().then().then(data => {
    console.log(data);
});
```
ֵ���Դ���ȥ
```javascript
const SUCCESS = 'fulfilled';
const FAIL = 'rejected';
const PENDING = 'pending';

function resolvePromise(promise2, x, resolve, reject) {
    // ... �ж� x ��ֵ
}

class Promise {
    constructor(executor) {
        // ... executor ����
    }
    then(onFulfilled, onRejected) {
        // ֵ��͸
        onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : val => val;
        onRejected = typeof onRejected === 'function' ? onRejected : err => {throw err};
        // ... promsie2 ���ж�
    }
}

```

## ����

- ����������Ƿ�������ǵ�promise A+ �淶
- `promises-aplus-tests` �������Ե�ǰ�Ŀ��Ƿ���Ϲ淶
- `npm i promises-aplus-tests -g`
- `promises-aplus-tests �ļ���`


## Promise A+ �淶
Promsie ��һ�����캯�����Ǹ��ࡣĬ�ϸ߰汾�������node���Դ��ˡ����ÿ��Ǽ����ԣ����Ĵ󵨵�ʹ�ðɣ�����治���ݣ��Ǿ���es6-promsie���Լ���һ�װ�~

Promsie A+

https://promisesaplus.com


## �ݴ�
��������ݣ�����Ҫһ�����ݴ����ǵ�executor �������һ��promsie��ʱ��ִ�еĽ����

```javascript
let Promise = require('./promise.js');
let p = new Promise((resolve, reject)=>{
    resolve(new Promise((resolve, reject)=>{
        reject(404);
    }));
});
p.then(value => console.log(1, value), reason => console.log(2, reason));

// �����
// 2 404
```

��ͬ��ִ��ʱ��resolve ��value��һ��Promsie����ô��Ҫ�����Ľ����

```javascript
const SUCCESS = 'fulfilled';
const FAIL = 'rejected';
const PENDING = 'pending';

function resolvePromise(promise2, x, resolve, reject) {
    // ... У��x
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
        // ... then ����
    }
}

module.exports = Promise;
```



