# 异步编程

## 1. Generator函数

异步编程解决方案
```javascript
    function* hello() {
        yield 'hello';
        yield 'world';
        return 'ending';
    ;}

    var h = hello();
    h.next();
```
* 状态机
* 遍历器对象（返回值，内部指针）
* 与普通函数区别：调用后并不会执行
* .next（）方法（遍历器对象方法）
    - 使指针移向下一个状态
    - 返回值：对象；//{value: 'hello', done: bool }
        + value: 当前内部状态的值 
        + done: 是否遍历结束

### 基本概念

#### 语法
    yield语句
    *号

##### yield语句
    暂停标志

* 运行逻辑
    - 遇到yield语句，暂停并将yield后的表达式的值作为返回对象的value值
        + yield语句本身没有返回值，或者说返回值总是undefined
    - 调用next()方法，直到遇到yield语句，再次暂停，重复上述过程
    - 遇到return语句，返回return语句后的表达式的值作为返回对象的value值
    - 若没有return语句，value值为undefined
* 与return语句的区别与联系
    - 都返回语句后的值
    - yield具有位置记忆功能，return没有
    - yield可以有多个额，return只能有一个
* yield语句后的表达式-惰性求值
```javascript
    function* gen() {
        yield 123 + 456;
    }
```
* 不能在普通函数中使用yield语句
    - 使用function定义的函数
* 在表达式中使用，必须放在圆括号中
* 用作函数参数或用于赋值表达式的右边，可以不加括号
* 不使用yield语句的Generator函数
```javascript
    function* f() {
        console.log('执行了');
    }

    var gen = f();
    
    setTimeout(function(){
        gen.next();
        }, 2000);
```


##### *号
    区别于普通函数

### 与Iterator接口的关系???
    Symbol.iterator

## 2. next方法的参数

####语法意义
    可以在Generator函数运行的不同阶段，从外部向内部注入不同的值，从而调整函数行为


```javascript
    function* f(){
    for(var i=0; true; i++){
            var reset = yield i;
            if(reset) {
                i=-1;
            }
            console.log('reset', reset);
        }
    }
    var g = f();
    g.next();
    g.next();
    g.next(true);
    <!-- 当next方法带有参数true时，当前的变量reset就被重置为这个参数，下一轮循环从-1开始 -->
```
##### Q: 为什么next方法的参数会重置reset？
    表示上一条yield语句的返回值

```javascript
    function* f(x) {
        var y = 2 * (yield (x + 1));
        //console.log('x', x);
        //console.log('y', y);
        var z = yield (y / 3);
        //console.log('z', z);
        return (x + y + z);
    }

    var a = f(5);
    a.next(); //{value: 6, done: false}
    a.next(); //{value: NaN, done: false}
    a.next(); //{value: NaN, done: false}

    var b = f(5);
    b.next(); //{value: 6, done: false}
    b.next(12); //{value: 8, done: false}
    b.next(13); //{value: 42, done: true}
```
* 第一次使用next方法的时候不能有参数
    - V8引擎直接忽略第一次使用next方法时的参数
    - 第一个next方法用来启动遍历器对象
* 如何在第一次使用next方法--在Generator函数外面再包一层

```javascript
    function wrapper(geneFun) {
        return function(...args) {
            let geneObj = geneFun(...args);
            geneObj.next();
            return geneObj;
        }
    }

    const wrapped = wrapper(function* () {
        console.log(`First input: ${yield}`);
        return 'DONE';
    });

    wrapped().next('hello');

    //First input: hello
```

## 3. for...of 循环

* 自动遍历Generator函数，不需要再调用next()方法

```javascript
    function* foo(){
        yield 1;
        yield 2;
        yield 3;
        return 4;
    }

    for(let v of foo()) {
        console.log(v);
    }
    //1 2 3 4 5
    //一旦next方法的返回对象的done属性为true，循环就会终止，故不会输出6
```

```javascript
// for...of循环实现裴波那契数列
function* fibonacci() {
    let [pre, curr] = [0, 1];
    for(;;) {
        [pre, curr] = [curr, pre + curr];
        yield curr;
    }   
}

for(let n of fibonacci()) {
    if(n > 1000) break;
    console.log(n);
}
```

* for...of循环，解构赋值，Array.from()都可以将Generator函数返回的Iterator对象作为参数，进行遍历

```javascript
function* numbers() {
    yield 1;
    yield 2;
    return 3;
    yield 4;
}

[...numbers()];

Array.from(numbers());

let [x, y] = numbers();

for(let n of numbers()) {
    console.log(n);
}

//[1,2]
```

* 利用for...of循环可以写出遍历任意对象的方法。

```javascript 
function* objectEntries(obj) {
    // 返回目标对象 属性名组成的数组
    let propKeys = Reflect.ownKeys(obj);
    console.log(propKeys);

    for(let propKey of propKeys) {
        yield [propKey, obj[propKey]];
    }
}

let jane = {first: 'Jane', last: 'Doe'};

for(let [key, value] of objectEntries(jane)) {
    console.log(`${key}: ${value}`);
}

//first: Jane
//last: Doe
```

```javascript
// 另一种写法，将Generator函数加到对象的Symbol.iterator属性上
function* objectEntries() {
    let propKeys = Object.keys(this);

    for(let propKey of propKeys) {
        yield [propKey, this[propKey]];
    }
}

let jane = {first: 'jane', lase: 'doe'};

jane[Symbol.iterator] = objectEntries;

for(let [key, value] of jane) {
    console.log(`${key}: ${value}`);
}
```

## 4. Generator.prototype.throw()
    在函数体外抛出错误，然后在Generator函数体内捕获

### 补充：
* throw语句
    - 显式地抛出异常
    - JavaScript解释器会立即停止当前正在执行的逻辑，并跳转至就近的异常处理程序
    - 抛出异常通常采用Error类型和其子类型
        + name属性：错误类型
        + message属性：存放传递给构造函数的字符串
* try/catch/finally语句
    - 捕获异常，异常处理机制
    - try：定义了需要处理的异常所在的代码块
    - catch：发生异常时，调用catch内的代码逻辑
        + e: 标识符，类似参数，将捕获的异常赋值给它；具有块级作用域，只在catch语句块中有定义
    - finally：放置清理代码，无论是否发生异常，总会执行
        + 使用return、continue、break或者throw语句使程序发生跳转，或者通过调用了抛出异常的方法改变了程序执行流程，不管这个跳转使程序挂起还是继续执行，解释器都会将其忽略

#### 概念
    所谓异常（exception）是当发生了某种异常情况或错误时产生的一个信号；

    抛出异常：用信号通知发生了错误或异常状况

    捕获异常：指处理这个信号，即采取必要手段从异常中恢复

###throw()方法的使用

```javascript
var g = function* () {
    while(true) {
        try {
            yield;
        } catch(e) {
            if(e != 'a') throw e;
            console.log('内部捕获', e);
        }
    }
};

var i = g();
i.next();

try{
    i.throw('a');
    i.throw('b');
} catch(e) {
    console.log('外部捕获', e);
}

//内部捕获 a
//外部捕获 b

<!-- 注意：上面的错误是用遍历器对象的throw方法抛出的，而不是用throw命令抛出的，后者只能被函数体外的catch语句捕获 -->
```

```javascript
// 如果Generator函数内部没有配置try...catch代码块，那么throw方法抛出的错误将被外部try...catch代码块捕获
var g = function* () {
    while(true) {
        yield;
        console.log('内部捕获', e);
    }
};

var i = g();
i.next();

try{
    i.throw('a');
    i.throw('b');
} catch(e) {
    console.log('外部捕获', e);
}

//外部捕获 a
```

如果Generator函数内部部署了try...catch代码块，那么遍历器的throw方法抛出的错误不能影响下一次遍历，否则遍历直接终止
```javascript
//部署了
var gen = function* () {
    try{
        yield console.log('hello');
        yield console.log('world1');
    } catch (e) {
        console.log(e);
    }
    yield console.log('world2');
}

var g = gen();
g.next();

g.throw('err');
g.next();
//hello
//err
//world2

//未部署
var gen = function* () {
    yield console.log('hello');
    yield console.log('world1');
}

var g = gen();
g.next();

try{
  g.throw('err');
 } catch(e) {
  g.next();
 }
//hello
```
但是如果使用throw命令抛出错误就不会影响遍历器状态：
```javascript
var gen = function* () {
    yield console.log('hello');
    yield console.log('world');
}

var g = gen();
g.next();

try{
    throw new Error();
} catch(e) {
    g.next();
}

//hello
//world
```

### 问题：为什么会这样输出？
```javascript
var gen = function* () {
    try{
        yield console.log('hello');
        yield console.log('world');
    } catch (e) {
        console.log(e);
    }
    
}

var g = gen();
g.next();

try{
    g.throw('err');
} catch(e) {
    g.next();
}
//hello
//err
//???这里为什么这样输出
```

这种<span style="color: red">函数体内捕获错误</span>的机制大大的方便了对错误的处理：
```javascript
var g = function* g() {
    try{
        var a = yield foo('a');
        var b = yield foo('b');
        var c = yield foo('c');
    } catch (e) {
        console.log('内部', e);
    }

    console.log('a, b, c');
}
```

函数体内抛出的错误也可以被函数体外的catch捕获:
```javascript
function* foo() {
    var x = yield 3;
    var y = x.toUperCase();
    yield y;
}

var it = foo();

it.next();

try {
    it.next(42);
} catch (e) {
    console.log(e);
}
```

一旦函数执行过程中抛出错误，就不会再执行下去了
```javascript
function* g() {
    yield 1;
    console.log('throwing an exception');
    throw new Error('generator broke');
    yield 2;
    yield 3;
}

function log(generator) {
    var v;
    console.log('starting generator');
    try{
        v = generator.next();
        console.log('第一次运行next方法', v);
    } catch(err) {
        console.log('捕捉错误', err);
    }
    try{
        v = generator.next();
        console.log('第二次运行next方法', v);
    } catch(err) {
        console.log('捕捉错误', err);
    }
    try{
        v = generator.next();
        console.log('第三次运行next方法', v);
    } catch(err) {
        console.log('捕捉错误', err);
    }
    console.log('caller done');
}

log(g());

//starting generator
//第一次运行next方法 {value: 1, done: false}
//throwing an exception
//捕捉错误 Error: generator broke
    //at g (<anonymous>:4:10)
    //at g.next (<anonymous>)
    //at log (<anonymous>:19:19)
    //at <anonymous>:33:3
//第三次运行next方法 {value: undefined, done: true}
//caller done
```







