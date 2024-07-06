---
title: Svelte 杂记
date: 2024-06-10 11:08:00 -0400
categories: [笔记, Svelte]
tags: [svelte, web]
description: 本文记录一些自己在学习 Svelte 过程中的理解
---

## Props 基础

### 基本使用

父组件如下：

```svelte
<script lang="ts">
    import Fun from "./Fun.svelte";
</script>

<Fun />
```
{: file="App.svelte"}

子组件如下：

```svelte
<h2>You rock!!</h2>
```
{: file="Fun.svelte"}

平平无奇。但是如果我们想在父组件里通过参数控制子组件的字体大小，就可以使用 Props。

子组件修改如下：

```svelte
<script lang="ts">
    export let fontSize = 30;
</script>

<h2 style="font-size: {fontSize}px">You rock!!</h2>
```
{: file="Fun.svelte"}

`export` 的作用就是标记 `fontSize` 为一个 Prop，这一点可能跟 JavaScript 中原本的功能有些不同。

给该变量赋值即代表该 Prop 有一个默认值，这样即便父组件中没有提供该参数，也不影响啥。

此时在父组件中可以这样使用：

```svelte
<script lang="ts">
    import Fun from "./Fun.svelte";
</script>

<Fun />
<Fun fontSize={40} />
<Fun fontSize={60} />
```
{: file="App.svelte"}

### 绑定

如果想让父组件的一个变量绑定到子组件的一个变量，让父组件的变量随着子组件变化，可以使用绑定。

因为默认来说，父组件的变量传递给子组件后，就像函数传参一样，是拷贝了一份值，子组件内的 Prop 在子组件内如何变化不影响父组件的变量，但是绑定后就会影响了。

子组件如下：

```svelte
<script lang="ts">
    export let fontSize = 30;
    setInterval(() => {
        if (fontSize >= 60) {
            fontSize = 30;
        } else {
            fontSize += 1;
        }
    }, 50);
</script>

<h2 style="font-size: {fontSize}px">You rock!!</h2>
```
{: file="Fun.svelte"}

父组件如下：

```svelte
<script lang="ts">
    import Fun from "./Fun.svelte";
    let size = 30;
</script>

<h1>Font size: {size}</h1>
<Fun bind:fontSize={size} />
```
{: file="App.svelte"}

`bind:` 的作用就是绑定父组件的变量到子组件的 Prop。

一个小技巧是，如果父组件的变量和子组件的参数相同，可以简写：

```svelte
<script lang="ts">
    import Fun from "./Fun.svelte";
    let fontSize = 30;
</script>

<h1>Font size: {fontSize}</h1>
<Fun bind:fontSize />
```
{: file="App.svelte"}

如果是不绑定的状态，则简写为：

```svelte
<script lang="ts">
    import Fun from "./Fun.svelte";
    let fontSize = 30;
</script>

<h1>Font size: {fontSize}</h1>
<Fun {fontSize} />
```
{: file="App.svelte"}

## 样式基础

子组件如下：

```svelte
<h2>You rock!!</h2>
```
{: file="Fun.svelte"}

父组件如下：

```svelte
<script lang="ts">
    import Fun from "./Fun.svelte";
</script>

<h1>Hello Svelte Developer</h1>
<h2>Rock and Roll</h2>
<Fun />

<style>
    h1 {
        color: red;
    }
    h2 {
        color: blue;
    }
</style>
```
{: file="App.svelte"}

**父组件的 Style 不会影响到子组件**。

如果想要产生影响，可以设置一个全局样式。比如在 `src`{: .filepath} 目录下创建一个 `global.css`{: .filepath} 文件，然后在 `main.ts`{: .filepath} 文件中导入：

```ts
import App from "./App.svelte";
import "./global.css"

const app = new App({
    target: document.getElementById("app")!,
});

export default app;
```
{: file="main.ts"}

## 练习：Color Picker

```svelte
<script lang="ts">
    import Slider from "./Slider.svelte";
    let red = 0;
    let green = 0;
    let blue = 0;
</script>

<h1>Color Picker</h1>

<Slider name="Red" bind:value={red} />
<Slider name="Green" bind:value={green} />
<Slider name="Blue" bind:value={blue} />

<div style="background-color: rgb({red}, {green}, {blue})" id="show"></div>
<p style="font-size: 25px; text-align: center">rgb({red}, {green}, {blue})</p>

<style>
    #show {
        width: 100%;
        height: 300px;
        border: 2px solid grey;
        border-radius: 8px;
    }
</style>
```
{: file="App.svelte"}

```svelte
<script lang="ts">
    export let value: number;
    export let name: string;
</script>

<p>{name} ({value})</p>
<input type="range" min="0" max="255" bind:value />

<style>
    input[type="range"] {
        width: 100%;
    }
</style>
```
{: file="Slider.svelte"}

这里注意每个颜色值都绑定了两遍，父组件一遍，子组件一遍。

## 事件基础

对于响应控件的交互事件，可以使用如下方式：

```svelte
<script>
    function onClick(e) {
        console.log("Clicked");
    }
    function onInput(e) {
        console.log("Inputted");
    }
    function onChange(e) {
        console.log("Changed");
    }
</script>

<button on:click={onClick}>Click Me</button>
<input type="range" on:input={onInput} on:change={onChange} />
```

`on:` 后面跟事件，比较简单，但是对于有哪些事件，需要查文档。

例如 `input` 的 `input` 事件，文档如下：[Element: input event - Web APIs](https://developer.mozilla.org/en-US/docs/Web/API/Element/input_event)

至于事件包含哪些属性哪些值，暂时可以通过打印到控制台查看。

要监控键盘事件，可以使用如下方式：

```svelte
<script>
    function onKeyPressed(e) {
        console.log(e.key, e, "keypressed");
    }
</script>

<svelte:window on:keypress={onKeyPressed} />
```

## 嵌套组件 vs 复合组件

假设有 `App.svelte`{: .filepath} 和 `Nested.svelte`{: .filepath}，嵌套组件是在 `App.svelte`{: .filepath} 这个文件内导入并使用 `<Nested />`，而复合组件则是用类似如下的方法使用组件：

```svelte
<Nested>
    <div>...</div>
    <p>...</p>
    ...
</Nested>
```

`<Nested />` 本身就是一个独立的单标签，其内容完全由 `Nested.svelte`{: .filepath} 控制。但是复合组件会接收外来标签，因此就需要确定这些外来标签在内部的显示位置，也就有了 `slot`。

## 响应式

由 `$:` 定义的响应式代码，是由 **赋值** 触发的，且赋值的左边必须出现变量的名字，变量才会更新。

## Props 就像类的参数

如果把一个组件（一个 `.svelte`{: .filepath} 文件）看作一个类的话，属性可以看作是类暴露出来的参数，在初始化这个类的时候，可以传一些参数进去。

一般来说一个组件内的 `<script>` 代码中的变量都是不可被外界控制的，就跟类的私有变量一样，但是加了 `export` 之后就表示把这个变量变为公有了，外界就可以给这个变量赋值了。

同样这个变量（属性）也可以有默认值。

## 派发自定义事件

DOM 有一系列内置事件，但是 Svelte 也允许我们自定义事件。就跟 Qt 里的信号一样，有一些是内置信号，但是我们也可以自己定义自己的信号。

以 [组件事件](https://learn-svelte-oranje.vercel.app/tutorial/component-events) 中的例子为例，`Inner.svelte`{: .filepath} 这样写

```svelte
<script>
    import { createEventDispatcher } from "svelte";

    const dispatch = createEventDispatcher();
    function sayMessage() {
        dispatch("message", {
            text: "message"
        })
    }
</script>

<button on:click={sayMessage}>
    Click to say message
</button>
```
{: file="Inner.svelte"}

`dispatch` 调用后就是创建了事件，第一个参数是事件的名字，第二个参数是一个对象（应该是个 `event.detail`），包含了事件的一些细节，比如这里的文本内容。而什么时候要创建这个事件，就是由按钮的 `click` 事件来控制了。

跟 Qt 差不多，Qt 中自定义信号的发送一般也是在某些槽函数内发生的。我们总得找个东西来控制何时发生这个事件。

一个组件内可以定义多个事件，也就是可以调用 `dispatch` 多次。

## 绑定的优势

假设有一个文本框，有一段文字，我们希望在文本框内输入的值会实时显现在文字中，该怎么做呢，可以像如下：

```svelte
<script>
    let name = "world";
    function inputChange(event) {
        name = event.target.value;
    }
</script>

<input type="text" value={name} on:input={inputChange} />
<p>Hello, {name}</p>
```

定义一个变量 `name`，然后监听 `<input>` 的 `input` 事件，写函数用新值更新 `name`，然后文本显示变量 `name`，即可。

但这又是事件，又是函数的，挺麻烦的，尤其事件要是多了，函数一大堆，因此有一个简单办法：直接用 `bind:`：

```svelte
<script>
    let name = "world";
</script>

<input type="text" bind:value={name} />
<p>Hello, {name}</p>
```

这就表示输入框的值和变量 `name` 绑定上了，两者双向改变。

当然，整两个输入框相互影响肯定也都是没问题的了：

```svelte
<script>
    let name = "world";
</script>

<input type="text" bind:value={name} />
<p>Hello, {name}</p>
<input type="text" bind:value={name} />
```

## 单选组和多选组

要定义一组单选很简单，如下：

```svelte
<label>
    <input type="radio" value="Dog" name="Pets" />
    Dog
</label>
<label>
    <input type="radio" value="Cat" name="Pets" />
    Cat
</label>
<label>
    <input type="radio" value="Fish" name="Pets" />
    Fish
</label>
```

只要 `name` 相同，它们就是一组的，选择就是互斥的，但是我们如何知道这一组中到底哪个被选中了呢？这就要用到 `group` 属性了，给每个选项都绑定 `group` 到相同的变量，则这个变量就是这个单选组选中的值：

```svelte
<script>
    let selected_pet;
</script>

<label>
    <input type="radio" value="Dog" name="Pets" bind:group={selected_pet}/>
    Dog
</label>
<label>
    <input type="radio" value="Cat" name="Pets" bind:group={selected_pet}/>
    Cat
</label>
<label>
    <input type="radio" value="Fish" name="Pets" bind:group={selected_pet}/>
    Fish
</label>
<p>{selected_pet ? `${selected_pet} is selected` : "No selection"}</p>
```

这个变量就是选中的那个单选框的 `value` 值。当然，因为在有选项之前，这个变量是未定义的，所以输出前需要做一个判断。

多选组也是一样的，只不过被绑定的变量不是单个值，而是所有选中值的数组。

```svelte
<script>
    let selected_pet = [];
</script>

<label>
    <input type="checkbox" value="Dog" name="Pets" bind:group={selected_pet}/>
    Dog
</label>
<label>
    <input type="checkbox" value="Cat" name="Pets" bind:group={selected_pet}/>
    Cat
</label>
<label>
    <input type="checkbox" value="Fish" name="Pets" bind:group={selected_pet}/>
    Fish
</label>
<p>{selected_pet.length ? `${selected_pet} is selected` : "No selection"}</p>

<h3>Selected:</h3>
<ul>
    {#each selected_pet as pet (pet)}
        <li>{pet}</li>
    {/each}
</ul>
```

对于多选组，还有一个办法如下：

```svelte
<script>
    let selected_pet = [];
</script>

<select multiple bind:value={selected_pet}>
    {#each ["Dog", "Cat", "Fish"] as pet (pet)}
        <option>{pet}</option>
    {/each}
</select>

<p>{selected_pet.length ? `${selected_pet} is selected` : "No selection"}</p>

<h3>Selected:</h3>
<ul>
    {#each selected_pet as pet (pet)}
        <li>{pet}</li>
    {/each}
</ul>
```

其中 `<option>{pet}</option>` 是 `<option value={pet}>{pet}</option>` 的简写。
