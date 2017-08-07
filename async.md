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
* 如何在第一次使用next方法

####语法意义
    可以在Generator函数运行的不同阶段，从外部向内部注入不同的值，从而调整函数行为












