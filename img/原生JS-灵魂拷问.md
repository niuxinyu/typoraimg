---
title: 原生JS 灵魂拷问
date: 2019-12-02 22:20:55
tags: JS
---

## 第一篇: JS数据类型之问——概念篇

### 1.JS原始数据类型有哪些？引用数据类型有哪些？

在 JS 中，存在着 7 种原始值，分别是：

- boolean
- null
- undefined
- number
- string
- symbol
- bigint

引用数据类型: 对象Object（包含普通对象-Object，数组对象-Array，正则对象-RegExp，日期对象-Date，数学函数-Math，函数对象-Function）

### 2.说出下面运行的结果，解释原因。

```js
function test (person) {
    person.age = 26
    person = {
        name: 'zs',
        age: 18
    }
    return person
}

const p1 = {
    name: 'wr',
    age: 19
}

const p2 = test(p1)

console.log(p1) ? 
console.log(p2) ?
```

结果

```js
p1: {name: 'wr', age: 26}
p2: {name: 'zs', age: 18}
```

> 原因: 在函数传参的时候传递的是对象在堆中的内存地址值，test函数中的实参person是p1对象的内存地址，通过调用person.age = 26确实改变了p1的值，但随后person变成了另一块内存空间的地址，并且在最后将这另外一份内存空间的地址返回，赋给了p2。

### 3.null是对象吗？为什么？

结论: null不是对象。

解释: 虽然 typeof null 会输出 object，但是这只是 JS 存在的一个悠久 Bug。在 JS 的最初版本中使用的是 32 位系统，为了性能考虑使用低位存储变量的类型信息，000 开头代表是对象然而 null 表示为全零，所以将它错误的判断为 object 。

### 4.'1'.toString()为什么可以调用？

其实在这个语句运行的过程中做了这样几件事情：

```js
var s = new Object('1')
s.toString()
s = null
```

第一步: 创建Object类实例。注意为什么不是String ？由于Symbol和BigInt的出现，对它们调用new都会报错，目前ES6规范也不建议用new来创建基本类型的包装类。

第二步: 调用实例方法。

第三步: 执行完方法立即销毁这个实例。

整个过程体现了 `基本包装类型`的性质，而基本包装类型恰恰属于基本数据类型，包括Boolean, Number和String。

> 参考:《JavaScript高级程序设计(第三版)》P118

### 5.0.1+0.2为什么不等于0.3？

0.1和0.2在转换成二进制后会无限循环，由于标准位数的限制后面多余的位数会被截掉，此时就已经出现了精度的损失，相加后因浮点数小数位的限制而截断的二进制数字在转换为十进制就会变成0.30000000000000004。

### 6.如何理解BigInt?

#### 什么是BigInt?

> BigInt是一种新的数据类型，用于当整数值大于Number数据类型支持的范围时。这种数据类型允许我们安全地对 `大整数`执行算术操作，表示高分辨率的时间戳，使用大整数id，等等，而不需要使用库。

#### 为什么需要BigInt?

在JS中，所有的数字都以双精度64位浮点格式表示，那这会带来什么问题呢？

这导致JS中的Number无法精确表示非常大的整数，它会将非常大的整数四舍五入，确切地说，JS中的Number类型只能安全地表示-9007199254740991(-(2^53-1))和9007199254740991（(2^53-1)），任何超出此范围的整数值都可能失去精度。

```js
console.log(999999999999999) -> 10000000000000000
```

同时也会有一定的安全性问题:

```js
9007199254740992 === 9007199254740993;    // → true 居然是true!
```

#### 如何创建并使用BigInt？

要创建BigInt，只需要在数字末尾追加n即可。

```js
console.log( 9007199254740995n );    // → 9007199254740995n
console.log( 9007199254740995 );     // → 9007199254740996
```

另一种创建BigInt的方法是用BigInt()构造函数、

```js
BigInt("9007199254740995");    // → 9007199254740995n
```

简单使用如下:

```js
10n + 20n => 30n
10n - 20n => -10n
+10n => TypeError:Cannot convert a BigInt value to a number
-10 => -10n
10n * 20n => 200n
20n / 10n => 2n
23n % 10n  => 3n
10n ** 3n => 1000n

const x = 10n
++x => 11n
--x => -9n
console.log(typeof x) bigint
```

#### 值得警惕的点

1. BigInt不支持一元加号运算符, 这可能是某些程序可能依赖于 + 始终生成 Number 的不变量，或者抛出异常。另外，更改 + 的行为也会破坏 asm.js代码。
2. 因为隐式类型转换可能丢失信息，所以不允许在bigint和 Number 之间进行混合操作。当混合使用大整数和浮点数时，结果值可能无法由BigInt或Number精确表示。

```js
10 + 10n => TypeError
```

3. 不能将BigInt传递给Web api和内置的 JS 函数，这些函数需要一个 Number 类型的数字。尝试这样做会报TypeError错误。

```js
Math.max(2n, 4n, 6n) => TypeError
```

4. 当 Boolean 类型与 BigInt 类型相遇时，BigInt的处理方式与Number类似，换句话说，只要不是0n，BigInt就被视为truthy的值。

```js
if (0n) { // 判断为false
    
}
if (3n) { // 判断为true
    
}
```

5. 元素都为BigInt的数组可以进行sort。

6. BigInt可以正常地进行位运算，如|、&、<<、>>和^

## 第二篇: JS数据类型之问——检测篇

### 1. typeof 是否能正确判断类型？

对于原始类型来说，除了 null 都可以调用typeof显示正确的类型。

```js
typeof 1 => 'number'
typeof '1' => 'string'
typeof undefined => 'undefined'
typeof true => 'boolean'
typeof Symbol() => 'symbol'
```

但对于引用数据类型，除了函数之外，都会显示"object"。

```js
typeof [] => 'object'
typeof {} => 'object'
typeof console.log => 'function'
```

因此采用typeof判断对象数据类型是不合适的，采用instanceof会更好，instanceof的原理是基于原型链的查询，只要处于原型链中，判断永远为true

```js
const Person = function() {}
const p1 = new Person()
p1 instanceof Person // true

var str1 = 'hello world'
str1 instanceof String // false

var str2 = new String('hello world')
str2 instanceof String // true
```

### 2. instanceof能否判断基本数据类型？

能

比如下面这种方式:

```js
class PrimitiveNumber {
    static [Symbol.hasInstance] (x) {
        return typeof x === 'number'
    }
}

console.log(111 instanceof PrimitiveNumber) // true
```

如果你不知道Symbol，可以看看MDN上关于hasInstance的解释。

其实就是自定义instanceof行为的一种方式，这里将原有的instanceof方法重定义，换成了typeof，因此能够判断基本数据类型。

### 3. 能不能手动实现一下instanceof的功能？

核心: 原型链的向上查找。





















