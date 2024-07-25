---
title: TypeScript 学习笔记
date: 2024-07-25 11:46:00 -0400
categories: [学习笔记, TypeScript]
tags: [typescript]
description: 本文记录 TypeScript 的学习笔记，基于官网手册
---

## 常见类型

### 变量

基本类型：`string`、 `number`、 `boolean`。

数组：`number[]`、`string[]` 等。等价于 `Array<number>` 等。

任意类型（无类型检查）：`any`。

在 `tsconfig.json` 文件中，开启 `noImplicitAny` 可以防止没有显式标注类型的变量被隐式标注为 `any`：

```json
{
    "compilerOptions": {
        "noImplicitAny": true,
    }
}
```
{: file="tsconfig.json"}

给变量指定类型：`let name: string = "Alice";`，但是大部分时候不需要手动指定。

### 函数

函数的参数和返回值可以指定类型：

```ts
function say_age(age: number): string {
    return `I am ${age} years old.`
}
```

一个返回 promise 的函数应该返回 `Promise` 类型：

```ts
async function getFavoriteNumber(): Promise<number> {
    return 26;
}
```

匿名函数的参数类型可以由上下文推断，比如如下示例：

```ts
const names = ["Alice", "Bob", "Eve"];
 
// Contextual typing for function - parameter s inferred to have type string
names.forEach(function (s) {
    console.log(s.toUpperCase());
});
 
// Contextual typing also applies to arrow functions
names.forEach((s) => {
    console.log(s.toUpperCase());
});
```

因为 `forEach` 函数对参数类型有规定，它规定了传入的函数参数需要有怎样的参数类型以及返回值类型，因此在匿名函数中不需要额外指定。

### 对象

可以通过给属性指定类型的方式声明对象类型：

```ts
function printCoord(pt: {x: number, y: number}) {
    console.log("x: " + pt.x);
    console.log("y: " + pt.y);
}

printCoord({x: 3, y: 7});
```

其中 `{x: number, y: number}` 中间换成分号也是可以的。

如果一个对象中的某个属性是可选的，则可以加一个 `?`：

```ts
function printName(name: {first: string, last?: string}) {
    if (name.last === undefined) {
        console.log(name.first);
    } else {
        console.log(`${name.first} ${name.last}`);
    }
}

printName({first: "Bob"});
printName({first: "Alice", last: "Alison"});
```

### 联合

联合类型，顾名思义，就是两个以上的类型的联合，写作 `string | number`。

但是注意，联合类型的变量可以使用的方法必须是所有成员类型都能用的才行，比如下面这样的就不行，因为 `toUpperCase` 只存在于 `string` 类型，`number` 不支持。

```ts
function printId(id: number | string) {
    console.log(id.toUpperCase());  // error
}
```

要解决这个问题，可以通过条件判断缩小类型的范围：

```ts
function printId(id: number | string) {
    if (typeof id === "string") {
        console.log(id.toUpperCase());
    } else {
        console.log(id);
    }
}
```

如果类型里有数组，可以这样判断：

```ts
function printId(id: string[] | string) {
    if (Array.isArray(id)) {
        console.log(id.join("."));
    } else {
        console.log(id);
    }
}
```

### 类型别名

顾名思义，可以给上述提到的对象类型和联合类型起一个别名。

```ts
type Point = {
    x: number;
    y: number;
};

function printCoord(pt: Point) {
    console.log("x: " + pt.x);
    console.log("y: " + pt.y);
}

printCoord({x: 3, y: 7});
```

```ts
type ID = number | string;
```

> 别名只是别名，与它所代指的类型别无二致。因此两个同一类型的别名也完全相同。
{: .prompt-warning}

### 接口

接口看上去跟类型别名没有太大不同，除了声明语法不一样，用起来感觉差不多：

```ts
interface Point {
    x: number;
    y: number;
}

function printCoord(pt: Point) {
    console.log("x: " + pt.x);
    console.log("y: " + pt.y);
}

printCoord({x: 3, y: 7});
```

但是它们还是有一些不同的，比如扩展的方式。从旧的接口扩展新的接口语法：

```ts
interface Animal {
    name: string;
}

interface Bear extends Animal {
    honey: boolean;
}
```

从旧的别名扩展新的别名的语法：

```ts
type Animal = {
    name: string;
};

type Bear = Animal & {
    honey: boolean;
};
```

上述结果是一样的，`Bear` 同时拥有 `name` 和 `honey` 属性。

但如果要把新的属性添加到已有的别名上，只有接口能做到了，别名不支持：

```ts
interface Window {
    title: string;
}

interface Window {
    ts: number;
}
```

### 类型断言

有时候一些函数只能在返回值标注比较抽象的基类，但实际上函数返回的可能是某个具体的子类，这时候可以使用断言类指定具体的子类，比如：

```ts
const myCanvas1 = document.getElementById("main_canvas") as HTMLCanvasElement;
// ...
const myCanvas2 = <HTMLCanvasElement>document.getElementById("main_canvas");
```

以上两种写法相同，不过后者在 `tsx` 的文件中因为语法问题不能这么用。

类型断言会在编译期被移除，不会留到运行期，所以哪怕断言的类型与实际的类型不同，也不会报错。

断言只允许把一个类型转为更具体或者更不具体的类型，而不允许随意转，比如下面这种就不行：

```ts
const x = "hello" as number;  // error
```

这是合理的，但有时候这种判断机制在面对复杂的类型时可能会出错，因此我们可以采取迂回战术：

```ts
const x = "hello" as any as number;  // ok
```

先指定为 `any` 再指定为 `number`。不会报错了，不过还是要记住，这些断言在运行之前就被移除了，对代码运行本身无任何影响。

### 字面量类型

一个值也可以是一个类型。比如说，`string` 指的是所有值为字符串的类型，而 `"hello"` 则仅指值为 `"hello"` 的值的类型：

```ts
let x: "hello" = "hello";
x = "world";  // error
```

单个值没啥用，但是联合起来就有用了，比如某个函数参数仅允许某几个特定的值：

```ts
function printText(s: string, alignment: "left" | "right" | "center") {
    // ...
}

printText("Hello world", "left");  //ok
printText("G'day, mate", "centre");  // not ok
```

```ts
function compare(a: string, b: string): -1 | 0 | 1 {
    return a === b ? 0 : a > b ? 1 : -1;
}
```

把字面量值和非字面量值结合起来也是可以的：

```ts
interface Option {
    width: number;
}

function configure(x: Option | "auto") {
    // ...
}

configure({width: 100});  // ok
configure("auto");  // ok
```

> 其实 `boolean` 就是两个字面量值联合的别名：`true | false`。
{: .prompt-tip}

在初始化一个对象时，TypeScript 会默认对象的属性的值是可变的，比如说你定义一个 `const obj = {counter: 0}`，TypeScript 会认为 `counter` 是一个 `number` 类型，而不是说只能是 `0` 这一个值。

这很正常，但是假如说有如下代码：

```ts
declare function handleRequest(url: string, method: "GET" | "POST"): void;

const req = {url: "https://www.example.com/", method: "GET"};
handleRequest(req.url, req.method);  // error
```

第四行会出问题，原因是尽管 `req.method` 的值是 `"GET"`，但 TypeScript 不认为它只能是 `"GET"`，而是把 `req.method` 当作 `string` 类型。但是 `handleRequest` 函数的声明中要求 `method` 只能是 `"GET" | "POST"`。类型不匹配了。

要解决这个问题，有 2 个办法：

- 将 `req.method` 声明为 `"GET"`：

```ts
const req = {url: "https://www.example.com/", method: "GET" as "GET"};
handleRequest(req.url, req.method);
```

这样表示固定了 `req.method` 作为 `"GET"` 类型只能是 `"GET"`，如果之后赋值为其他就会报错。

```ts
const req = {url: "https://www.example.com/", method: "GET"};
handleRequest(req.url, req.method as "GET");
```

这样表示明确告知 `handleRequest` 函数这个 `req.method` 就是 `"GET"`，给我过！但是不影响之后把 `req.method` 改成其他值。

- 把整个对象声明为 `const`：

```ts
const req = {url: "https://www.example.com/", method: "GET"} as const;
handleRequest(req.url, req.method);
```

这表示声明整个 `req` 的属性和属性值都是不可变的。`const req` 中的 `const` 限定 `req` 的值不可变，`as const` 中的 `const` 限定 `req` 中的属性和属性值不可变。

现在 `req` 的类型为：

```ts
const req: {
    readonly url: "https://www.example.com/"
    readonly method: "GET"
}
```

### 空和未定义

JavaScript 有 `null` 和 `undefined` 两个类型来表示空或者未定义。TypeScript 也有两个对应且相同名称的类型。但是它们的行为取决于有没有开 `strictNullChecks`。

```json
{
    "compilerOptions": {
        "noImplicitAny": true,
        "strictNullChecks": true
    }
}
```
{: file="tsconfig.json"}

- 如果没开

就跟普通的 JavaScript 一样，你可以把 `null` 或 `undefined` 赋值给任意类型的属性。但是如果你不对这些值进行检查的话，就可能会造成很多 bugs。所以我们建议开启这个选项。

- 如果开了

如果一个变量可能是 `null` 或者 `undefined`，则你必须在使用它之前检查其值，比如：

```ts
function doSomething(x: string | null) {
    if (x === null) {
        // ...
    } else {
        console.log("hello, " + x.toUpperCase());
    }
}
```

或者，当你十分确定一个变量在当前情况下不可能为 `null` 或者 `undefined` 的时候，TypeScript 也提供了一个语法帮助我们简化代码：

```ts
function doSomething(x: string | null) {
    console.log("hello, " + x!.toUpperCase());
}
```

只需要在变量后加一个 `!`。

> 但是这只是一个针对编译器的声明，并不影响运行时，所以只有当你十分确定的时候再用。
{: .prompt-warning}



