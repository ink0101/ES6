# Iterator和for...of循环

## 概念
    为各种数据结构提供统一的接口，完成遍历操作（即依次处理改数据结构的所有成员）

### 作用：
1. 为各种数据结构提供一个统一的，简单的访问接口；
2. 使数据结构的成员能够按照某种次序排列；
3. 主要供for...of循环消费；

### 遍历过程：
* 1.

```javascript
// 模拟next()方法返回值
function makeIterator(array) {
    var nextIndex = 0;
    return {
        next: function() {
            return nextIndex < array.length ?
                {value: array[nextIndex ++], done: false} :
                {value: undefined, done: true};
        }
    }
}

var it = makeIterator(['a', 'b']);
it.next();//{value: "a", done: false}
it.next();//{value: "b", done: false}
it.next();//{value: undefined, done: true}
```

```javascript
// 由遍历器对象模拟出数据结构
function idMaker() {
    var index = 0;

    return {
        next: function() {
            return {value: index ++, done: false};
        }
    }
}

var it = idMaker();
it.next();//{value: 0, done: false}
it.next();//{value: 1, done: false}
it.next();//{value: 2, done: false}
    
```

## 默认Iterator接口

    默认的Iterator接口部署在Symbol.iterator属性上

### Symbol.iterator

* 具有Symbol.iterator属性就是可遍历的
* 返回当前数据结构默认的遍历器生成函数
* 要放在[]中（点运算符后面总是字符串）

### 原生具备Iterator接口

1. 数组
2. 类似数组的对象
3. Set和Map结构
```javascript
let arr = ['a', 'b', 'c'];
let iter = arr[Symbol.iterator]();

iter.next();//{value: "a", done: true}
```

### 遍历对象

* 在Symbol.iterator属性上部署遍历器生成方法
* 将对象转换为Map结构

```javascript
//一个类部署遍历器接口
class RangeIterator {
    constructor(start, stop) {
        this.value = start;
        this.stop = stop;
    }

    [Symbol.iterator]() {return this;}

    next() {
        var value = this.value;
        if(value < this.stop) {
            this.value ++;
            return {done: false, value: value};
        } else {
            return {done: true, value: undefined};
        }
    }
}

function range(start, stop) {
    return new RangeIterator(start, stop);
}

for(var value of range(0, 3)) {
    console.log(value);
}
```

<span style="color: red">应用场景是什么?????</span>
```javascript
//实现指针结构
function Obj(value) {
    this.value = value;
    this.next = null;
}
Obj.prototype[Symbol.iterator] = function() {
    //???
    var iterator = {
        next: next
    };

    var current = this;

    function next() {
        if(current) {
            var value = current.value;
            var done = current === null;
            current = current.next;
            return {
                done: done,
                value: value
            }
        } else {
            return {
                done: true
            }
        }
    }

    return iterator;
}

var one = new Obj(1);
var two = new Obj(2);
var three = new Obj(3);

one.next = two;
two.next = three;

for(var i of one) {
    console.log(i);
}
```

```javascript
//为对象添加Iterator接口
let obj = {
    data: ['hello', 'world'],
    [Symbol.iterator]() {
        const self = this;
        let index = 0;
        return {
            next() {
                if(index < self.data.length) {
                    return {
                        value: self.data[index ++],
                        done: false
                    };
                } else {
                    return {
                        value: undefined,
                        done: true,
                    };
                }
            }
        }
    }
}

let objIter = obj[Symbol.iterator]();
objIter.next();//{value: "hello", done: false}
objIter.next();//{value: "world", done: false}
objIter.next();//{value: undefined, done: true}
```

```javascript
//类数组对象部署Iterator接口
let iterator = {
    0: 'a',
    1: 'b',
    2: 'c',
    length: 3,
    [Symbol.iterator]: Array.prototype[Symbol.iterator]
};
for(let item of iterator) {
    console.log(item);
}
//a
//b
//c
```

## 使用场合

* 解构赋值
* 扩展运算符
* yield*
* 其他场合
    - for...of循环
    - Array.from()
    - Map(), Set(), WeakMap()和WeakSet()
    - Promise.all()
    - Promise.race()

## 字符串的Iterator接口
## 和Generator函数关系
## return(), throw()

### return()
```javascript
// return()方法
function readLinesSync(file) {
    return {
        next() {
            if(file.isAtEndOfFile()) {
                file.close();
                return {done: true};
            }
        },
        return() {
            file.close();
            return {done: true};
        }
    };
}

for(let line of readLinesSync(fileName)) {
    console.log(x);
    break;
}
```

### throw()
主要配合Generator函数使用

## for...of循环

### 数组

* 代替数组的forEach方法
* 与for...in循环的不同
    - for...in只能获取键名，不能获取键值，for...of可以遍历获得就键值，获取键名（索引）需要借助数组实例的entries方法和keys方法 
    - for...of循环只返回具有数字索引的属性（number类型）, for...in循环返回值为string类型
```javascript
    let arr = [3, 5, 7];
    arr.foo = 'hello';

    for(let i in arr) {
        console.log(typeof i, i);
    }

    for(let i of arr.keys()) {
        console.log(typeof i, i);
    }
```

### Set和Map结构
    可以直接使用for...of循环

* 遍历的顺序是按照各个成员被添加进数据结构的顺序；
* Set结构遍历时返回的是一个值，而Map结构遍历时返回的是一个数组，数组成员分别为Map成员的键名和键值；

### 计算生成的数据结构
    有些数据结构是在现有的数据结构的基础上生成的

* entries()
* keys()
* values()

### 类似数组的对象
* 字符串
    - for...of循环会正确识别32位UTF-16字符 <span style="color: red">???</span>
* DOM NodeList对象
* arguments对象

```javascript
//for...of循环会正确识别32位UTF-16字符
for(let x of '
') {
    console.log(x);
}
```

* 没有Iterator接口的类似数组的对象--使用Array.from方法将其转为数组



### 对象

* 普通对象无法使用for...of循环，可以使用for...in循环遍历键名
    - 解决办法：
        + 使用Object.keys方法将键名生成一个数组
        + 使用Generator函数将对象重新包装一下
```javascript
const obj = {
    a: '1',
    b: '2',
    c: '3',
    d: '4'
};
//使用Object.keys方法将键名生成一个数组
for(let key of Object.keys(obj)) {
    console.log(key + ':' + obj[key]);
}
//使用Generator函数将对象重新包装一下
function* entries(obj) {
    for(let key of Object.keys(obj)) {
        yield [key, obj[key]];
    }
}

for(let [key, value] of entries(obj)) {
    console.log(key, '->', value);
}
```

### 与其他遍历语法的比较

* for循环 >>>> 太麻烦
* forEach循环  >>>> 无法中途跳出，break命令和return命令都不能奏效
* for...in 循环
    - 缺点
        + 返回数组的键名为字符串
        + 不仅会遍历数字键名，还会遍历手动添加的其他键，甚至包括原型链上的键
        + 某些情况下， for...in会以任意顺序遍历键名
    - 主要为遍历对象而设计，不适用于数组
* for...of循环
    - 优点
        + 语法简介
        + 不会有for...in的缺点
        + 不同于forEach, 它可以与break、continue和return配合使用
        + 提供了遍历所有数据结构的统一操作接口



