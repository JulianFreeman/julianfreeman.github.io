---
title: JavaScript 杂记
date: 2023-10-06 22:08:00 -0400
categories: [笔记, JavaScript]
tags: [javascript]
description: 本文记录一些关于 JavaScript 的内容
---

## 简说函数

函数有三种类型的写法：

### 常规的写法

```js
function sum(a, b) {
    return a + b;
}
```

### 函数变量的写法

```js
//注意，函数变量一般用 const 声明
const sum = function (a, b) {
    return a + b;
};
```

### 箭头函数的写法

```js
const sum = (a, b) => {
    return a + b;
};
```

箭头函数的写法又可以有好多种简化：

#### 当参数有且只有一个时

```js
//包裹参数的括号可以省略
const greet = name => {
    return `Hello, ${name}`;
};
```

#### 当函数体有且只有一条返回语句时

```js
//包裹函数体的大括号和 return 关键词可以省略
const sum = (a, b) => a + b;
```

## 简说相等

在 JavaScript 里，`==` 是个历史遗留问题，真正等价于其它语言的相等运算符是 `===` 。所以当遇到要进行相等比较时，不要考虑用 `==` 还是 `===` ，一律用 `===` 。

细节来说，双等于在比较时会有隐式转换，就会导致不同类型的数据也会产生相等的结果，而三等于会先进行类型比较，如果类型不一样直接返回 `false` ，所以三等于才是严格的相等比较。

## 简说对象

在对象中添加一个值为函数的键值对：

```js
let dog = {
    "eat": function () {
        console.log("eating...");
    },
};
```

但是如果键是一个合法的变量名，那就可以省略引号：

```js
let dog = {
    eat: function () {
        console.log("eating...");
    },
};
```

同时新语法还支持我们这样写：

```js
let dog = {
    eat () {
        console.log("eating...");
    },
};
```

当然你也可以这样写：

```js
let dog = {
    "eat" () {
        console.log("eating...");
    },
};
```

weird……

但是这样写应该就熟悉些了：

```js
let dog = {
    eat() {
        console.log("eating...");
    },
};
```

### getter 和 setter

```js
let person = {
    name: "John",
    _age: 33,
    get age() {
        return this._age;
    },
    set age(num) {
        this._age = num;
    }
};

person.age = 50;
console.log(person.age);
```

### Property Value Shorthand

```js
function createPerson(name, age) {
    return {
        name: name,
        age: age,
    };
}
//equals to
function createPerson(name, age) {
    return {
        name,
        age,
    };
}
```

### Destructured Assignment

```js
let person = {
    name: "John",
    age: 13,
};

let name = person.name;
//equals to
let { name } = person;
```

## 简说类

### 构造

```js
class Person {
    constructor(name, age) {
        this._name = name;
        this._age = age;
    }
    get name() {
        return this._name;
    }
    eat(food) {
        //...
    }
}
let john = new Person("John", 12);
```

类包含一个 `constructor` 方法，用于构造对象并初始化。getter 和 setter 跟 `Object` 一样。不过类的方法与方法之间不需要用逗号分隔。

### 继承

```js
class Person {
    constructor(name, age) {
        this._name = name;
        this._age = age;
    }
    get name() {
        return this._name;
    }
    eat(food) {
        //...
    }
}
class Student extends Person {
    constructor(name, age, grade) {
        super(name, age);
        this.grade = grade;
    }
    study() {
        //...
    }
}
```

继承使用 `extends` 关键字。在构造函数中建议首先调用 `super` 。

### 静态方法

```js
class Person {
    constructor(name, age) {
        this._name = name;
        this._age = age;
    }
    get name() {
        return this._name;
    }
    eat(food) {
        //...
    }
    static generateNumber() {
        return Math.random();
    }
}
console.log(Person.generateNumber());
```

对象实例不能调用静态方法。

## Node.js 运行时环境下的模块导入导出

```js
//foo.js
function greet(name) {
    return `Hello, ${name}`;
}
module.exports.greet = greet;
//或者如果要同时导出好几个变量
let obj = "abc";
module.exports = { greet, obj };

//bar.js
const foo = require("./foo.js");
foo.greet("John");
//or ... destructured assignment
const { greet, obj } = require("./foo.js");
greet("Karl");
console.log(obj);
```

在浏览器环境下就不是这样了哈。

## 简说 this

> 以下只讨论严格模式下的情况。
{: .prompt-info}

看 `this` 是不是 `undefined` ，就看调用 `this` 所在的函数时，是直接作为函数调用的，还是作为对象的方法调用的。

比如：

```js
"use strict";

function makeUser() {
    return {
        name: "John",
        ref: this,
    };
}

let user = makeUser();
alert(user.ref.name);
```

这个 `this` 最外层的函数是 `makeUser` ，虽然 this 在一个匿名对象里，但它并不在这个对象的方法中，所以这个对象对它没有影响。

**`this` 是在其最外层函数被调用时确定指向的。**

比如在上述代码中， `this` 的值是在 `let user = makeUser();` 这一行确定的，这一行中， `this` 所在的函数是单纯作为函数被调用的，因此为 `undefined` 。

如果把代码改一下：

```js
"use strict";

function makeUser() {
    return {
        name: "John",
        ref() {
            return this;
        },
    };
}

let user = makeUser();
alert(user.ref().name);
```

此时 `this` 的最外层函数是 ref ，这时 `this` 的值是在 `alert(user.ref().name);` 这里确定的，具体说是在 `user.ref()` 时确定的，因为这就是它最外层的函数被调用的时刻。这里该函数是作为对象的方法被调用的，因此 `this` 就指向对象 `user` ，所以结果是 `"John"` 。

> 注意，这里最外层的函数不包括箭头函数，因为箭头函数没有 `this` 。
{: .prompt-tip}

## 类型转换和值的比较

### 类型转换

先谈谈转换，转字符串很简单，转布尔也很简单，转数字这里需要特别说一下。

在算数运算中，如果运算符是加号，且其中任意一个运算元是字符串，那另一个运算元就会被转换为字符串并进行字符串拼接。

```console
> 'a' + true
'atrue'
> 'a' + undefined
'aundefined'
```

除此之外的所有其他情况（其他运算符和其他类型的运算元），运算元都会被转换成数字进行运算。

```console
> true + null
1
> '6' - '4'
2
```

那其他类型都是怎么转换为数字的呢？下面是对照表：

| 值 | 变成…… |
| -- | -- |
| `undefined` |	`NaN` |
| `null` | `0` |
| `true` and `false` | `1` and `0` |
| `string` | 去掉首尾空白字符（空格、换行符 `\n`、制表符 `\t` 等）后的<br>纯数字字符串中含有的数字。如果剩余字符串为空，<nr>则转换结果为 `0` 。否则，将会从剩余字符串中“读取”数字。<br>当类型转换出现 error 时返回 `NaN` 。 |

### 值的比较

同类型的比较很简单，就不说了，主要说一下不同类型的比较。

虽然 JavaScript 中的 `===` 是严格的相等判断，但是其他的比较可没这么严格，比如大于或小于等等这些不等比较都是不严格的，所以还是有必要了解一下这些不严格比较的规则。

抛开 `===` 不谈，**不同类型** 在进行比较时，JavaScript 都会先把它们转换为 **数字类型** 再进行比较。 `==` 和 `<` 或者 `>=` 等相等和不等比较都是如此。

除了这个简单的基本规则之外，还有一些值得注意的地方。

1. 有 `NaN` 参与的比较总是返回 `false` ，哪怕是 `NaN == NaN` 也是 `false` 。这里注意， `undefined` 在比较中会转换为 `NaN` ，字符串也可能会转换为 `NaN` 。
2. `null == undefined` 返回 `true` 。这是一个特殊规则，在进行这个比较时，运算元不会进行类型转换。除此之外，经测试， `null == null` 是 `true` ，三等也一样。而且 `undefined == undefined` 也是 `true` ，三等也一样。对于 `null` 来说，我们尚且可以用转换数字的思路去想，但是对于 `undefined` ，如果转换数字就是 `NaN == NaN` ，这其实是 `false` ，也就是说，并不能用转换数字的思路去思考 `undefined == undefined` ，姑且就死记硬背吧。不过 `null === undefined` 是 `false` ，估计是因为不同的类型总返回 `false` 吧。 `undefined >= undefined` 是 `false` ，这里大概进行了数字转换吧。但 `null >= null` 是 `true` 。
3. 相等比较总是用 `===` ，不等比较要尽力避免一端是 `null` 或者 `undefined` 。

## 空值合并运算符 ??

众所周知，逻辑运算符 `&&` 和 `||` 不止可以运算布尔值，而是所有值。它们的一般规则是这样：

- `&&` 会返回第一个假值，如果没有假值，就返回最后一个值。
- `||` 会返回第一个真值，如果没有真值，就返回最后一个值。

现在又有一种运算符 `??` ，它的一般规则是这样的：

1. 如果一个值既不是 `null` 也不是 `undefined` ，我们则称其为已定义的值。
2. `??` 会返回第一个已定义的值，如果没有已定义的值，就返回最后一个值。

所以有如下示例：

```console
> null ?? 0 ?? "a"
0
> "" ?? undefined
''
> undefined ?? undefined ?? null
null
```

`??` 的优先级与 `||` 相同，比 `&&` 略低。

另一条强制规则是这样的：

- 如果没有显式地添加括号， `??` 将不能与 `||` 或 `&&` 一起使用。

## 函数默认值

之所以要把函数默认值拿出来说，是因为它跟其他语言不太一样。

首先，

```js
"use strict"

function greet(from, text) {
    console.log(`${from}: ${text}`);
}

greet("Ann", "hello"); //Ann: hello
greet("Ben"); //Ben: undefined
```

如果有的函数参数没有传递，是 **不会报错** 的，此时没有传递的函数参数在函数内被指定为 `undefined` 。

在出现默认值的语法之前，函数内通常是这么判断某个参数是否有传递值的：

```js
function greet(from, text) {
    //1
    if (text === undefined) {
        text = "Hello";
    }
    //2
    text = text || "Hello";
    //3
    text = text ?? "Hello";
    console.log(`${from}: ${text}`);
}
```

方法一是原理，方法二有缺陷，所以方法三是更简洁的写法。

但是后来有默认值了，就不这么麻烦了：

```js
"use strict"

function greet(from, text="Hello") {
    console.log(`${from}: ${text}`);
}

greet("Ann", "hello"); //Ann: hello
greet("Ben"); //Ben: Hello
```

但是关于默认值，有一点需要注意，参数默认值只有在参数没有指定的时候才进行计算。比如：

```js
"use strict"

function getMsg() {
    console.log("[called]");
    return "Hello";
}

function greet(from, text=getMsg()) {
    console.log(`${from}: ${text}`);
}

greet("Ann", "hello");
//Ann: hello

greet("Ben");
//[called]
//Ben: Hello
```

虽然参数 `text` 给的默认值是一个函数调用，但并不代表它会在创建函数时执行，它只在该参数为 `undefined` 时才执行并将返回值赋值给 `text` 。

也许像如下这样理解就更容易些了：

```js
"use strict"

function getMsg() {
    console.log("[called]");
    return "Hello";
}

function greet(from, text) {
    text = text ?? getMsg();
    console.log(`${from}: ${text}`);
}

greet("Ann", "hello");
//Ann: hello

greet("Ben");
//[called]
//Ben: Hello
```

但两者并不等同，因为后者传递 `null` 时也会触发默认值，而前者不会。

## 函数返回值

如果没有显式指定返回值，或者使用 `return;` 返回，则会返回 `undefined` 。

`return` 和返回的表达式之间不要断行，比如如果写作这样：

```js
return
(some + long + expression + or + whatever * f(a) + f(b));
```

实际的作用是：

```js
return;
(some + long + expression + or + whatever * f(a) + f(b));
```

JavaScript 默认会在 `return` 后加分号，所以如果想断行，至少这样写：

```js
return (
    some + long + expression
    + or +
    whatever * f(a) + f(b)
);
```

## 注释文档

因为 JavaScript 的变量并不会限定类型，所以像函数传参这种变量默认没什么代码提示。因此可以加一些注释来说明参数的类型。比如 [JSDoc](https://en.wikipedia.org/wiki/JSDoc)。

## 可选链 ?.

对于这样的代码，会报错：

```js
"use strict"

let user = {};
console.log(user.address.street); //error
```

因为 `user` 中没有 `address` ，所以 `user.address` 返回 `undefined` ，一个 `undefined` 肯定是没有 `street` 属性的。这个时候要判断就得这样写：

```js
"use strict"

let user = {};
if (user.address) {
    console.log(user.address.street);
}
```

这样如果 `user.address` 未定义，也不会报错。

但是如果我们想要 `user.address.street.name` 呢？要判断几次？显然每次都 `if` 判断很麻烦，因此有了 **可选链** 语法。

在此之前，先定义一个概念：如果一个值不是 `null` 也不是 `undefined` ，我们称其为 **已存在**。

```js
"use strict"

let user = {};
console.log(user.address?.street);
```

`?.` 的作用就是，如果它左边的值 **不存在**，那就不会继续向右获取值，而是直接返回 `undefined` ，这样就省下了我们去 `if` 判断了。

其他地方也可以用 `?.` ，比如 `?.()` 和 `?.[]` 。

```js
"use strict"

let user = {};
console.log(user.greet?.());
console.log(user.email?.["today"]);
```

如果直接执行 `user.greet()` ，若 `user` 内没有 `greet` 方法，就会报错，但使用 `user.greet?.()` 会返回 `undefined` ，而非报错。

如果直接获取 `user.email["today"]` ，若 `user` 内没有 `email` 属性，就会报错，但使用 `user.email?.["today"]` 会返回 `undefined` ，而非报错。

注意，不要过度使用可选链，在一些技术上不应该不存在的变量后不要使用可选链，因为这会导致调试变得更难。

## 灵活的 this

```js
"use strict"

let arrLike = {
    0: "hello",
    1: "world",
    length: 2,
    [Symbol.iterator]() {
        return {
            current: 0,
            length: this.length, //1
            super: this, //2
            next() {
                if (this.current < this.length) { //3
                    return {done: false, value: this.super[this.current++]}; //4
                }
                else {
                    return {done: true};
                }
            },
        };
    },
};

for (let v of arrLike) {
    console.log(v);
}
```

通过以上代码了解一下 `this` 。

JavaScript 中的 `this` 是灵活的，并不一定非得出现在对象的方法中，但是它应该如此。所以我们只关心对象方法中的 `this` ，如果 `this` 出现在对象的方法中， `this` 所指的就是这个对象。

我们分析一下上述代码中的 `this` 分别都是指的谁：

1. 这里 `length: this.length` 中的 `this` 实际上是 `[Symbol.iterator]` 方法中的，所以它指的是 `arrLike` 这个对象。
2. 同上。
3. 这里的 `this.current < this.length` 中的两个 `this` 是 `next` 方法中的，所以它们指的是 `next` 所存在的对象，这个对象没有显式的名字，是由 `[Symbol.iterator]` 方法直接返回的一个匿名对象。
4. 同上。这就是为什么如果我们想要获取到 `arrLike` 中的值，需要指定一个 `super` 的原因，因为直接在 `next` 中使用 `this` 是获取不到 `arrLike` 的。

更多内容见 [函数的上下文](#函数的上下文)。

## 对象转原始类型

所有对象转布尔类型都是真，就很简单。

主要是如何转字符串和数字。

- 如果一个场景需要一个字符串，那么就按照 `hint="string"` 转换。比如一些输出函数。
- 如果一个场景需要一个数字，那么就按照 `hint="number"` 转换。比如大部分数学运算。
- 如果不太确定这个场景需要什么，就按照 `hint="default"` 转换，比如二元加法和双等比较。

除了 `Date` 外，所有内建方法都使用跟 `number` 相同的方法实现 `default` ，我们也可以这样。即 `hint="default"` 也按照数字去转换：

```js
"use strict"

let user = {
    name: "John",
    id: 1234,
    [Symbol.toPrimitive](hint) {
        switch (hint) {
            case "string":
                return this.name;
            case "number":
            case "default":
                return this.id;
        }
    },
}

console.log(String(user)); //John
console.log(Number(user)); //1234
```

对于转换，

- 首先找 `[Symbol.toPrimitive]` 方法，找到了只用它处理所有 `hint` 。
- 如果没找到，而且 `hint="string"` 的情况下，先找 `toString` 方法，找不到就找 `valueOf` 方法。
- 如果 `hint="number"` 或者 `hint="default"` ，先找 `valueOf` 方法，找不到就找 `toString` 方法。

因为在早期，还没有 `symbol` 的概念的时候，只有 `toString` 和 `valueOf` 可以把对象转换为原始类型，只是对于不同的 `hint` 它们的优先级不同。后来出现 `[Symbol.toPrimitive]` 了，就可以处理所有 `hint` 了。

`toString` 和 `valueOf` 如果返回的是对象，不会报错，但等同于不存在该函数。

`[Symbol.toPrimitive]` 如果返回的是对象，会报错。 

**为什么说这个呢？**

想记录一个题目，巧妙地用了转换。

如何写一个函数 `sum` 实现如下输出呢？

```js
console.log(String(sum(10))); //10
console.log(String(sum(10)(20))); //30
console.log(String(sum(10)(20)(30))); //60
```

其实这里既然 `sum()` 可以被一直调用，说明 `sum` 返回的并非一个普通的数字，而得是一个函数，问题就在于打印函数怎么会出现相加的结果呢？其实结果无非就是一个数字，打印函数如何返回数字呢？

因为函数本身也是个对象，所以可以将函数转换为原始类型（字符串或数字）输出。

```js
"use strict"

function sum(num) {
    let s = num;
    function inner(n) {
        s += n;
        return inner;
    }
    inner.toString = function() {
        return s;
    }
    return inner;
}

console.log(String(sum(10))); //10
console.log(String(sum(10)(20))); //30
console.log(String(sum(10)(20)(30))); //60
```

其实 `toString` 不一定非得返回字符串， `valueOf` 也不一定非得返回数字，这都是随意的，没有规定，只要是原始类型就行。

## 函数的上下文

`bind` 返回的是一个绑定了上下文或者参数的函数， `call` 和 `apply` 是用一个上下文或一些参数执行一个函数。

相比来说， `apply` 的第二个参数是一个类数组，比 `call` 一个个传参要灵活一些。不过差别也不大。

### this 到底指啥

简单说，谁调用这个函数， `this` 就是谁。

函数无非就出现在两个地方，一种是对象外的函数，一种是对象内的函数。

对象外的函数，就是全局函数，或者嵌套函数，不是对象内的方法，这类函数中 `this` 是 `undefined` 。

```js
"use strict"

function outer() {
    function inner() {
        console.log(this);
    }
    inner();
}

outer(); //undefined
```

对象内的函数，也就是对象方法，一般来说 `this` 指的是包含该方法的对象，但是因为函数也可以随便赋值传值么，有时候调用这个方法的对象就不一定是包含该方法的对象了。比如：

```js
"use strict"

let user = {
    name: "John",
    hello() {
        console.log(`Hello ${this.name}`);
    },
};

let admin = {};
admin.hi = user.hello;

user.hello(); //Hello John
admin.hi(); //Hello undefined
```

第二次函数运行，调用 `hi` 方法的对象是 `admin` ，所以函数内的 `this` 也就变成了 `admin` ，但是 `admin` 没有 `name` 属性，所以就返回了 `undefined` 。

当然，有的时候也会直接丢失 `this` ：

```js
"use strict"

let user = {
    name: "John",
    hello() {
        console.log(`Hello ${this.name}`);
    },
};

let hi = user.hello;

user.hello(); //Hello John
hi(); //error，因为 this 是 undefined
```

在传递函数当作参数的时候要格外注意这点，因为你不知道在函数内，这个被传递过来的参数是怎么被调用的，很可能就会失去 `this` ，或者被换掉 `this` （比如 Node.js 中，传递到 `setTimeout` 的函数，其中的 `this` 指向的是 `setTimeout` 返回的 `timer` 对象）。

我们稍微改一下代码：

```js
"use strict"

let user = {
    name: "John",
    hello() {
        return function() {
            console.log(`Hello ${this.name}`);
        };
    },
};

let hi = user.hello();
hi(); //error
```

因为 `user.hello()` 返回的是一个函数，所以就相当于 ``let hi = function() {console.log(`Hello ${this.name}`)};`` ，然后又调用函数，相当于个全局函数，所以 `this` 不存在。

如果调用时提供了对象， `this` 就是这个对象：

```js
"use strict"

let user = {
    name: "John",
    hello() {
        return function() {
            console.log(`Hello ${this.name}`);
        };
    },
};

let admin = {
    name: "admin",
};
admin.hi = user.hello();
admin.hi(); //Hello amdin
```

但是如果改成箭头函数呢？

> 笔记中断。
{: .prompt-danger}
