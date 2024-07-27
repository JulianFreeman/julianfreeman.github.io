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

## 关于函数

### 函数表达式类型

这是最简单的方法：

```ts
function greeter(fn: (a: string) => void) {
    fn("hello world");
}

function printToConsole(s: string) {
    console.log(s);
}

greeter(printToConsole);
```

`(a: string) => void` 的意思是接收一个 `string` 类型的参数，没有返回值。**函数参数名必须要指定（也就是这里的 `a`）**。

如果一个类型太长，也可以用别名：

```ts
type GreetFunction = (a: string) => void;

function greeter(fn: GreetFunction) {
    // ...
}
```

### 调用签名

在 JavaScript 中，函数除了可以被调用，也可以拥有属性。想要定义拥有某个特定属性的函数，可以如下：

```ts
type DescribableFunction = {
    description: string;
    (arg: number): boolean;
};

function doSomething(fn: DescribableFunction) {
    console.log(fn.description + " returned " + fn(6));
}

function myFunc(arg: number) {
    return arg > 3;
}

myFunc.description = "default description";

doSomething(myFunc);
```

注意这里跟单纯的函数表达式类型的定义有点区别，`(arg: number): boolean` 中间是 `:`，不是 `=>`。

### 构造签名

JavaScript 的函数也可以用 `new` 创建，在 TypeScript 中可以这样指定：

```ts
interface SomeObject {}

type SomeConstructor = {
    new (s: string): SomeObject;
};

function fn(ctor: SomeConstructor) {
    return new ctor("hello");
}
```

有 `new` 和没有 `new` 两种构造方式可以共存：

```ts
interface CallOrConstruct {
    (n?: number): string;
    new (s: string): Date;
}
```

### 泛函

如果一个函数的返回值类型和参数类型有一定关联，那就可以用泛型：

```ts
function firstElem<T>(arr: T[]): T | undefined {
    return arr[0];
}
```

指定多个泛型：

```ts
function map<T, S>(arr: T[], func: (arg: T) => S): S[] {
    return arr.map(func);
}
```

对泛型可以增加一些限制，比如我们需要一个拥有 `length` 属性的类型：

```ts
function longest<T extends {length: number}>(a: T, b: T) {
    return a.length >= b.length ? a : b;
}
```

因为对 `T` 增加了限制，TypeScript 也就明白 `a` 和 `b` 都拥有 `length` 属性。

但是这里有个常见的错误得注意：

```ts
function minimumLength<T extends {length: number}>(obj: T, minimum: number): T {
    if (obj.length >= minimum) {
        return obj;
    } else {
        return {length: minimum};  // error
    }
}
```

错误的原因在于，`T extends {length: number}` 限定了 `T` 必须有 `length` 属性，但是 `{length: minimum}` 不一定符合 `T` 类型。比如如果 `T` 是一个数组，那么 `{length: minimum}` 就缺少应该有的 `0`、 `1` 等索引属性。

大部分时候 TypeScript 可以自己推断泛型的实际类型，但有时候也需要我们手动指定：

```ts
function combine<T>(a: T[], b: T[]): T[] {
    return a.concat(b);
}

const arr1 = combine([1, 2, 3], ["hello"]);  // error
const arr2 = combine<string | number>([1, 2, 3], ["hello"]);  // ok
```

> 泛型不要滥用：能不用就不用；能不限制就不限制；能简单点就简单点。
{: .prompt-warning}

### 函数重载

函数重载的语法：

```ts
function makeDate(t: number): Date;
function makeDate(m: number, d: number, y: number): Date;
function makeDate(m_t: number, d?: number, y?: number): Date {
    if (d !== undefined && y !== undefined) {
        return new Date(y, m_t, d);
    } else {
        return new Date(m_t);
    }
}
```

这里有几个点要注意，

- 前两行是两个重载的定义
- 第三行是函数的具体实现，但是这个实现不能被直接调用，也是不可见的
- 函数实现的签名必须能够兼容前面的每个重载函数的签名

错误一：无法直接调用函数实现

```ts
function fn(x: string): void;
function fn() {
    // ...
}

fn();  // error
```

> 因为函数实现对外不可见，所以其上的重载函数至少有两个才有意义。
{: .prompt-warning}

错误二：参数没有兼容

```ts
function fn(x: boolean): void;
function fn(x: string): void;  // error
function fn(x: boolean) {
    // ...
}
```

修改如下：

```ts
function fn(x: boolean): void;
function fn(x: string): void;
function fn(x: boolean | string) {
    // ...
}
```

错误三：返回值没有兼容

```ts
function fn(x: string): string;
function fn(x: number): boolean;  // error
function fn(x: number | string) {
    return "oops";
}
```

修改如下：

```ts
function fn(x: string): string;
function fn(x: number): boolean;
function fn(x: number | string): string | boolean {
    return "oops";
}
```

> 同样的，函数重载能不用就不用，优先考虑联合类型。
{: .prompt-warning}

### 其他类型

当一个函数没有返回值，返回值类型就是 `void`。但是 `void` 与 `undefined` 并不同。

非基础类型（`string`、 `number`、 `bigint`、 `boolean`、 `symbol`、 `null`、 `undefined`）的类型可以用 `object` 类型表示。在 TypeScript 中函数也是 `object`。

`unknown` 可以表示任何类型，但是跟 `any` 不同的是，对 `any` 类型的变量作任何操作都行，但是对 `unknown` 类型的变量则不行。

如果一个函数永远不返回值，则标记为 `never` 类型。

### 扩展语法

```ts
function multiply(n: number, ...m: number[]) {
    return m.map((x) => n * x);
}

const a = multiply(10, 1, 2, 3, 4);
```

`m` 这里要声明为 `number[]` 而非 `number`。

```ts
const args = [8, 5] as const;
const angle = Math.atan2(...args);
```

`Math.atan2` 只接受两个参数，但是 `args` 不一定总是两个，因此这里需要增加 `as const` 限制这个 `args` 就是固定两个。

### 参数解构

要给解构的参数指定类型，可以这样

```ts
function sum({a, b, c}: {a: number, b: number, c: number}) {
    console.log(a + b + c);
}
```

如果太长，可以用别名

```ts
type ABC = {a: number, b: number, c: number};

function sum({a, b, c}: ABC) {
    console.log(a + b + c);
}
```

但是注意，下面函数的意思是，将传入的对象参数解构出属性 `a`，但是在函数体内重命名为 `number`，类型是 `any`。

```ts
function fn({a: number}) {
    console.log(number)
}
```

### 返回值是 void 的情况

如果一个函数类型的返回值是 `void`，则以下写法也是合法的

```ts
type voidFunc = () => void;

const f1: voidFunc = () => {
    return true;
};

const f2: voidFunc = () => true;

const f3: voidFunc = function () {
    return true;
};

const v1 = f1();
const v2 = f2();
const v3 = f3();
```

但是变量 `v1`、 `v2` 和 `v3` 的类型还是 `void` 而不会变成 `boolean`。

这个的应用场景比如

```ts
const src = [1, 2, 4];
const dst = [0];

src.forEach((e) => dst.push(e));
```

尽管 `push` 函数返回值不是 `void`，但是因为 `forEach` 规定了函数参数的返回值就是 `void`，因此这样用没问题。

但是对于函数表达式就不行了，规定了返回值是 `void` 就必须是 `void`：

```ts
function f1(): void {
    return true;  // error
}
```
这样的也不行，也是因为强制规定了返回值为 `void`：

```ts
const f2 = function (): void {
    return true;  // error
};
```

