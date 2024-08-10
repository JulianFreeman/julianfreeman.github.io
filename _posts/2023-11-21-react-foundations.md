---
title: React 基础
date: 2023-11-21 15:51:00 -0500
categories: [翻译, React]
tags: [react, web]
description: Next.js 官方教程翻译，后续若网站内容更新可能会有所不同
media_subpath: /assets/img/react-foundations/
---

> 官方教程链接：https://nextjs.org/learn/react-foundations
{: .prompt-info}

## React 基础

为了更有效地学习 Next.js，我们需要对 JavaScript、React，和一些相关的前端开发概念有所了解。但是 JavaScript 和 React 都是很大的话题，我们怎么知道什么时候就可以开始学习 Next.js 了呢？

欢迎来到 React 基础课程！这是一个对初学者友好，由案例引导的课程，它会帮助你了解学习 Next.js 的必备知识。你会一步步构建一个简单的项目，先从一个 JavaScript 应用开始，然后一点点转移到 React 和 Next.js。

每一章节都是基于前一章节构建的，所以你可以根据自己知道多少来选择从哪里开始。

### 预备知识

本教程需要一些 HTML，CSS，JavaScript 基础，但不需要了解 React。如果你已经了解 React 了，你可以直接跳到[从 React 到 Next.js](#第八章-从-react-到-nextjs) 或者[仪表盘应用]({{site.url}}/posts/learn-nextjs)课程。

### 系统要求

在开始教程之前，请确保你安装了如下软件/系统：

- Node.js 20.12.0 或者更高
- 操作系统：macOS、Windows(包括 WSL)，或者 Linux
- VSCode 或者其他文本编辑器

## 第一章 React 和 Next.js

Next.js 是一个灵活的 **React 框架**，提供了许多构建模块来创建快速的全栈 **网页应用**。

但这具体是什么意思？让我们先花点时间看一下 React 和 Next.js 到底是什么，以及它们能做什么。

### 一个网页应用的构建模块

如果你要构建一个现代的网页应用，就需要考虑很多方面的内容，比如：

- **用户界面**：用户如何与你的应用交互。
- **路由**：用户如何在应用的不同部分进行切换。
- **数据抓取**：你的数据存在哪里以及如何获取。
- **渲染**：何时何地该渲染静态内容或者动态内容。
- **整合**：你使用什么第三方应用（CMS，用户验证，支付功能等）以及如何连接到它们。
- **基础设施**：你在哪里部署、存储，和运行你的应用代码（无服务器、CND、Edge 等）。
- **性能**：如何为用户优化应用性能。
- **可扩展性**：当你的团队、数据、用户流量逐渐增长时你的应用如何适应。
- **开发体验**：你团队开发和维护该应用的体验。

对于你应用的每一部分，你都需要思考你是自己想办法解决，还是使用第三方的库或者框架解决。

### React 是什么？

[React](https://react.dev/) 是一个用于构建交互式 **用户界面** 的 JavaScript **库**。

这里的用户界面，指的是用户在屏幕上看到的并且可以交互的各种元素。

![learn-react-components](light/learn-react-components.avif){: .light}
![learn-react-components](dark/learn-react-components.avif){: .dark}

这里的库，指的是 React 提供了很多很有用的功能来构建 UI，但是由开发者自己决定在哪里使用这些功能。

React 能够成功的一部分原因也在于它相对不那么关心构建应用的其他方面。这也造就了一个拥有丰富的第三方工具和解决方案（比如 Next.js）的生态系统。

这也意味着，要从头开始构建一个完整的 React 应用，是需要花些工夫的。开发者需要花时间配置各种工具以及为某些常见的应用需求重复造轮子。

### Next.js 是什么？

Next.js 是一个 React **框架**，提供了创建网页应用的构建模块。

这里的框架，指的是 Next.js 负责处理 React 需要的工具配置，也提供了一些额外的工具，功能和优化。

![learn-ecosystem](light/learn-ecosystem.avif){: .light}
![learn-ecosystem](dark/learn-ecosystem.avif){: .dark}

你可以使用 React 来构建你的 UI，然后逐渐用 Next.js 的功能来解决常见的应用需求，比如路由、数据抓取、整合等，一步步提升开发体验和用户体验。

不管你是独立开发者，还是一个大团队的一员，你都可以使用 React 和 Next.js 来构建一个全交互式、高动态的、优秀的网页应用。

在接下来的几个章节，我们会讨论如何使用 React 和 Next.js。

## 第二章 渲染用户界面

要理解 React 是如何工作的，我们首先需要理解浏览器是如何解析我们的代码并创建（或者说渲染）一个用户界面（UI）的。

当用户访问一个网页时，服务器会返回一个 HTML 文件给浏览器，这个文件可能像如下的样子：

![learn-html-and-dom](light/learn-html-and-dom.avif){: .light}
![learn-html-and-dom](dark/learn-html-and-dom.avif){: .dark}

然后浏览器读取这个 HTML 文件并构建了一个文件对象模型（DOM）。

### 什么是 DOM？

DOM 是一种对 HTML 元素的呈现，这种呈现是基于对象的。它是代码和用户界面之间的桥梁，有树状结构并包含一些父子关系。

![learn-dom-and-ui](light/learn-dom-and-ui.avif){: .light}
![learn-dom-and-ui](dark/learn-dom-and-ui.avif){: .dark}

你可以使用 DOM 方法和 JavaScript，来监听用户事件，[修改 DOM 结构](https://developer.mozilla.org/docs/Learn/JavaScript/Client-side_web_APIs/Manipulating_documents)（选择、添加、更新、删除用户界面上的特定元素）。修改 DOM 不仅是针对特定的元素，也可以修改元素的样式和内容。

我们在下一节学习如何使用 JavaScript 和 DOM 方法。

> **延伸阅读**：
> - [DOM 介绍](https://developer.mozilla.org/docs/Web/API/Document_Object_Model/Introduction)
> - [如何在谷歌浏览器中查看 DOM](https://developer.chrome.com/docs/devtools/dom/)
> - [如何在火狐浏览器中查看 DOM](https://developer.mozilla.org/docs/Tools/Debugger/How_to/Highlight_and_inspect_DOM_nodes)

## 第三章 用 JavaScript 更新 UI

让我们看看该如何使用 JavaScript 和 DOM 来给我们的项目插入一个 `h1` 标签。

打开你的代码编辑器，然后创建一个新的 `index.html` 文件，在文件中添加如下内容：

```html
<html>
  <body>
    <div></div>
  </body>
</html>
```
{: file="index.html"}

然后给 `div` 一个唯一的 `id` 以便后续我们可以选中它。

```html
<html>
  <body>
    <div id="app"></div>
  </body>
</html>
```
{: file="index.html"}

要想在 HTML 中编写 JavaScript 代码，添加一个 `script` 标签。

```html
<html>
  <body>
    <div id="app"></div>
    <script type="text/javascript"></script>
  </body>
</html>
```
{: file="index.html"}

现在，在 `script` 标签内，你可以使用 DOM 方法 [`getElementById()`](https://developer.mozilla.org/en-US/docs/Web/API/Document/getElementById)，然后通过 `id` 选中 `<div>` 元素。

```html
<html>
  <body>
    <div id="app"></div>
 
    <script type="text/javascript">
      const app = document.getElementById('app');
    </script>
  </body>
</html>
```
{: file="index.html"}

然后你可以继续使用 DOM 方法创建一个新的 `h1` 元素。

```html
<html>
  <body>
    <div id="app"></div>
 
    <script type="text/javascript">
      // Select the div element with 'app' id
      const app = document.getElementById('app');
 
      // Create a new H1 element
      const header = document.createElement('h1');
 
      // Create a new text node for the H1 element
      const text = 'Develop. Preview. Ship.';
      const headerContent = document.createTextNode(text);
 
      // Append the text to the H1 element
      header.appendChild(headerContent);
 
      // Place the H1 element inside the div
      app.appendChild(header);
    </script>
  </body>
</html>
```
{: file="index.html"}

为了确保一切正常，在浏览器里打开这个 HTML 文件，你应该会看到一个 h1 标签，写着“Develop. Preview. Ship.”。

### HTML 和 DOM

如果你在[浏览器开发工具](https://developer.chrome.com/docs/devtools/overview/)中查看 DOM 元素，你会发现 DOM 中包含一个 `<h1>` 元素。页面的 DOM 结构和我们的源码不一致。

![learn-dom-and-source](light/learn-dom-and-source.avif){: .light}
![learn-dom-and-source](dark/learn-dom-and-source.avif){: .dark}

这是因为 HTML 呈现的是 **原始的页面内容**，而 DOM 呈现的是 **更新后的页面内容**，我们刚刚用 JavaScript 更新的。

使用纯 JavaScript 代码更新 DOM，很厉害，但也很繁琐。你需要写这么多代码才能把一个 `<h1>` 元素插入到页面中：

```html
<script type="text/javascript">
  const app = document.getElementById('app');
  const header = document.createElement('h1');
  const text = 'Develop. Preview. Ship.';
  const headerContent = document.createTextNode(text);
  header.appendChild(headerContent);
  app.appendChild(header);
</script>
```
{: file="index.html"}

当应用或者团队越来越大的时候，用这种方式构建应用会变得越来越棘手。

这种方式需要开发者花很多时间写很多代码告诉计算机它要 **如何** 工作。但是如果我们只需要描述我们想要 **什么**，然后让计算机自己处理该 **如何** 更新 DOM，岂不美哉？

### 命令式编程和声明式编程

上面的代码是一个 **命令式编程** 的很好的例子。你一步步写代码告诉计算机该 **如何** 更新用户界面。但是对于构建用户界面，往往声明式才是更好的方法，因为它可以加快开发进度。相较于使用 DOM 方法，如果开发者可以直接声明他们想展示 **什么**（此例中即一个 `h1` 标签以及一些文字）会更好。

换句话说，**命令式编程** 就像你告诉一个厨师如何一步步制作一个披萨。**声明式编程** 则是直接点一个披萨，而不需要关心这个披萨是怎么一步步做出来的。🍕

[React](https://react.dev/) 就是一个知名的帮助开发者构建用户界面的声明式编程库。

### React：一个声明式 UI 库

作为一个开发者，你可以告诉 React 你想要什么样的用户界面，然后 React 会自己想办法更新 DOM。

下一节，我们会开始讨论如何使用 React。

> **延伸阅读**：
> - [HTML 和 DOM](https://developer.chrome.com/docs/devtools/dom/#appendix)
> - [声明式 UI 和命令式 UI 的对比](https://react.dev/learn/reacting-to-input-with-state#how-declarative-ui-compares-to-imperative)

## 第四章 开始使用 React

要在项目中使用 React，你可以使用一个第三方网站 [unpkg.com](https://unpkg.com/) 加载两个 React 脚本：

- **react** 是 React 的核心库。
- **react-dom** 提供了一些特定于 DOM 的方法以便于你可以使用 React 操作 DOM。

```html
<html>
  <body>
    <div id="app"></div>
 
    <script src="https://unpkg.com/react@17/umd/react.development.js"></script>
    <script src="https://unpkg.com/react-dom@17/umd/react-dom.development.js"></script>
 
    <script type="text/javascript">
      const app = document.getElementById('app');
    </script>
  </body>
</html>
```

此时我们不通过纯 JavaSript 代码来操作 DOM 了，而是使用 `react-dom` 里的 `ReactDOM.render()` 方法告诉 React 我们想在 `#app` 元素内渲染一个 `h1` 标题。

```html
<html>
  <body>
    <div id="app"></div>
 
    <script src="https://unpkg.com/react@17/umd/react.development.js"></script>
    <script src="https://unpkg.com/react-dom@17/umd/react-dom.development.js"></script>
 
    <script type="text/javascript">
      const app = document.getElementById('app');
      ReactDOM.render(<h1>Develop. Preview. Ship. 🚀</h1>, app);
    </script>
  </body>
</html>
```

但是如果你尝试在浏览器里运行这段代码，你会得到一个语法错误：

![error](error.avif)

这是因为 `<h1>...</h1>` 并不是合法的 JavaScript 语法。这其实是一段 JSX 代码。

### 什么是 JSX？

JSX 是一种对 JavaScript 的语法扩展，它允许你用一种跟 HTML 相似的语法来描述你的 UI。有关 JSX 特别棒的一点是，除了遵守 [JSX 的三条规则](https://react.dev/learn/writing-markup-with-jsx#the-rules-of-jsx)之外，你不需要再学习任何 HTML 和 JavaScript 之外的语法了。

要注意，浏览器本身并不理解 JSX，所以你需要一个 JavaScript 编译器，比如 [Babel](https://babeljs.io/)，来把 JSX 代码转换为正常的 JavaScript 代码。

### 在项目中添加 Babel

把下面的脚本复制粘贴到 `index.html` 文件中，就把 Babel 添加到项目中了：

```html
<script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
```

另外，你还需要告诉 Babel 哪段代码需要进行转换，所以我们把 `script` 标签的类型改为 `type=text/jsx`。

```html
<html>
  <body>
    <div id="app"></div>
    <script src="https://unpkg.com/react@17/umd/react.development.js"></script>
    <script src="https://unpkg.com/react-dom@17/umd/react-dom.development.js"></script>
    <!-- Babel Script -->
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
    <script type="text/jsx">
      const app = document.getElementById('app');
      ReactDOM.render(<h1>Develop. Preview. Ship. 🚀</h1>, app);
    </script>
  </body>
</html>
```

现在你就可以在浏览器里运行代码并检查是否一切正常了。

相比于只需要写这几行声明式的 React 代码：

```html
<script type="text/jsx">
  const app = document.getElementById("app")
  ReactDOM.render(<h1>Develop. Preview. Ship. 🚀</h1>, app)
</script>
```

在上一节你需要写这么多命令式的 JavaScript 代码：

```html
<script type="text/javascript">
  const app = document.getElementById('app');
  const header = document.createElement('h1');
  const text = 'Develop. Preview. Ship. 🚀';
  const headerContent = document.createTextNode(text);
  header.appendChild(headerContent);
  app.appendChild(header);
</script>
```

我们可以看到，使用 React 可以帮助我们砍掉很多重复代码。

这就是 React 能做的，它是一个包含了很多可复用代码片段的库，来帮助我们完成各种任务。比如更新 UI。

### 学习 React 必备的 JavaScript 知识

尽管你可以同时学习 JavaScript 和 React，但如果你熟悉 JavaScript，学习 React 就会更容易一些。

在后面的章节，我们会从 JavaScript 的角度介绍一些 React 的核心概念。下面是一些我们会提到的 JavaScript 内容：

- [函数](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Functions)和[箭头函数](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Functions/Arrow_functions)
- [对象](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)
- [数组和数组方法](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Array)
- [解构](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)
- [模板字符串](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Template_literals)
- [三元远算符](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Operators/Conditional_Operator)
- [ES 模块和导入/导出语法](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Modules)

尽管本教程不会涉及到很深的 JavaScript，但是保持跟进 JavaScript 的最新版本也是个好习惯。如果你对 JavaScript 尚不自信，不用担心，不要因为此而停止了学习 React 的脚步！

## 第五章 用组件构建 UI

### React 的核心概念

要用 React 构建应用，你需要了解 React 的三个核心概念：

- 组件
- 属性
- 状态

在之后的几个章节，我们会介绍这些概念，并提供一些学习资源。

### 组件

用户界面可以分解成更小的构建模块，我们称为 **组件**。

使用组件可以构建一些自包含的、可复用的代码片段。你可以把组件想象成 **乐高积木**，用一个个的积木可以搭建出很多高大的结构。如果你想更新 UI 的某一个部分，你可以更新这部分的组件。

![components](components.avif)

这种模块化的结构可以提高代码的可维护性，因为你可以轻松地添加、更新或删除特定的组件，而不需要改动应用剩余的部分。

关于 React 组件很棒的一点是，它们都是 JavaScript。我们来看看如何从 JavaScript 的角度编写一个 React 组件。

### 创建组件

在 React 中，组件都是 **函数**。在你的 `script` 标签中，编写一个名为 `header` 的函数：

```html
<script type="text/jsx">
  const app = document.getElementById("app")
 
 
  function header() {
  }
 
  ReactDOM.render(<h1>Develop. Preview. Ship. 🚀</h1>, app)
</script>
```

组件是一个 **返回 UI 元素** 的函数。在函数的返回语句中，你可以编写 JSX 代码：

```html
<script type="text/jsx">
  const app = document.getElementById("app")
 
  function header() {
     return (<h1>Develop. Preview. Ship. 🚀</h1>)
   }
 
  ReactDOM.render(, app)
</script>
```

要渲染这个组件到 DOM，你只需要把它作为第一个参数传递给 `ReactDOM.render()`：

```html
<script type="text/jsx">
 
  const app = document.getElementById("app")
 
  function header() {
     return (<h1>Develop. Preview. Ship. 🚀</h1>)
   }
 
 
   ReactDOM.render(header, app)
</script>
```

但是等等，如果你尝试在浏览器里运行这段代码，你会得到一个错误。要修正这个错误，你要做两件事：

首先，React 的组件必须是首字母大写的，这样可以跟纯 HTML 和 JavaScript 代码区分开。

```tsx
function Header() {
  return <h1>Develop. Preview. Ship. 🚀</h1>;
}
 
// Capitalize the React Component
ReactDOM.render(Header, app);
```

其次，你可以像使用 HTML 标签那样使用 React 组件，也就是用尖括号 `<>`。

```tsx
function Header() {
  return <h1>Develop. Preview. Ship. 🚀</h1>;
}
 
ReactDOM.render(<Header />, app);
```

### 嵌套组件

通常一个应用不只有一个组件。你可以在 React 组件中 **嵌套** 其他组件，就像常规的 HTML 元素那样。

在这个例子中，创建一个名为 `HomePage` 的组件：

```tsx
function Header() {
  return <h1>Develop. Preview. Ship. 🚀</h1>;
}
function HomePage() {
  return <div></div>;
}
 
ReactDOM.render(<Header />, app);
```

然后把 `<Header>` 组件嵌入到 `<HomePage>` 组件中：

```tsx
function Header() {
  return <h1>Develop. Preview. Ship. 🚀</h1>;
}
 
function HomePage() {
  return (
    <div>
      {/* Nesting the Header component */}
      <Header />
    </div>
  );
}
 
ReactDOM.render(<Header />, app);
```

### 组件树

你可以持续嵌套 React 组件，组成一个组件树。

![component-tree](component-tree.avif)

比如说，你最顶层的 `<HomPage>` 组件可以包含一个 `<Header>`，一个 `<Article>` 和一个 `<Footer>` 组件。然后每个组件又可以包含它们各自的子组件。比如说 `<Header>` 组件可以包含 `<Logo>`、`<Title>` 和 `<Navigation>` 等。

这样的模块化结构允许你在应用的不同部分复用某些组件。

在我们的项目中，因为 `<HomePage>` 是现在最顶层的组件了，你可以把它传递给 `ReactDOM.render()` 方法。

```tsx
function Header() {
  return <h1>Develop. Preview. Ship. 🚀</h1>;
}
 
function HomePage() {
  return (
    <div>
      <Header />
    </div>
  );
}
 
ReactDOM.render(<HomePage />, app);
```

## 第六章 使用属性呈现数据

目前为止，如果你想多次使用 `<Header />` 的话，你每次都会得到相同的内容。

```tsx
function Header() {
  return <h1>Develop. Preview. Ship. 🚀</h1>;
}
 
function HomePage() {
  return (
    <div>
      <Header />
      <Header />
    </div>
  );
}
```

但如果你想展示不同的内容呢，或者说你也不知道要展示什么内容，因为数据要从外部资源获取呢？

常规的 HTML 元素中有一些属性可以传递一些信息，并允许你借此修改这些元素的某些表现。比如说，我们修改 `<img>` 的 `src` 属性可以改变其展示的图片，修改 `<a>` 标签的 `href` 属性可以改变链接的目标地址。

同样的，我们也可以给 React 组件传递一些信息，作为组件的属性，它们被称为 `props`。

![props](props.avif)

跟 JavaScript 函数一样，你可以设计一些组件，让它们能接受自定义的参数，以便改变自身行为，或者选择性地呈现内容。然后你可以通过父组件把这些参数传递给子组件。

> 在 React 中，数据是沿着组件树向下传递的。这是一种 *单向数据传递*。状态（我们会在下一节讨论）可以作为属性从父组件传递给子组件。
{: .prompt-tip}

### 使用属性

在 `HomePage` 组件中，我们可以传递一个自定义的属性 `title` 给 `Header` 组件，就像常规的 HTML 标签属性一样。

```tsx
function HomePage() {
  return (
    <div>
      <Header title="React 💙" />
    </div>
  );
}
```

而子组件 `Header` 可以把这些属性作为第一个 **函数参数** 接收：

```tsx
function Header(props) {
  return <h1>Develop. Preview. Ship. 🚀</h1>;
}
```

如果你打印 `props`，你会看到这是一个 **`object`**，里面有一个 `title` 属性。

```tsx
function Header(props) {
  console.log(props); // { title: "React 💙" }
  return <h1>React 💙</h1>;
}
```

既然 `props` 是一个 `object`，那么我们可以使用 **对象解构** 来显式地给属性命名：

```tsx
function Header({ title }) {
  console.log(title); // "React 💙"
  return <h1>React 💙</h1>;
}
```

然后你可以把 `h1` 内的文字替换为 `title` 变量。

```tsx
function Header({ title }) {
  console.log(title);
  return <h1>title</h1>;
}
```

如果现在打开浏览器，你会看到实际呈现的是 `"title"` 本身，而不是这个变量的内容。这是因为 React 以为你只是想渲染一个纯文本。

你需要某种方式告诉 React 这是一个 JavaScript 变量。

### 在 JSX 中使用变量

要使用自定义的变量，我们可以用 **花括号 `{}`**，这是一个特殊的 JSX 语法，可以允许你在 JSX 标记中编写常规的 JavaScript 代码。

```tsx
function Header({ title }) {
  console.log(title);
  return <h1>{title}</h1>;
}
```

你可以把花括号想象为从 “JSX 之地”进入 “JavaScript 之地”的方式。你可以在花括号内添加任意 **JavaScript 表达式**（能够计算出某个单个值的东西），比如说：

1. 一个 **对象的属性**

```tsx
function Header(props) {
  return <h1>{props.title}</h1>;
}
```

2. 一个 **模板字符串**

```tsx
function Header({ title }) {
  return <h1>{`Cool ${title}`}</h1>;
}
```

3. **函数的返回值**

```tsx
function createTitle(title) {
  if (title) {
    return title;
  } else {
    return 'Default title';
  }
}
 
function Header({ title }) {
  return <h1>{createTitle(title)}</h1>;
}
```

4. 或者 **三元运算符**

```tsx
function Header({ title }) {
  return <h1>{title ? title : 'Default Title'}</h1>;
}
```

现在你可以向 `title` 这个属性传递任意字符串，而且因为你在组件里写了一个三元运算符来处理默认情况，你甚至可以不用传递这个 `title` 属性。

```tsx
function Header({ title }) {
  return <h1>{title ? title : 'Default title'}</h1>;
}
 
function HomePage() {
  return (
    <div>
      <Header />
    </div>
  );
}
```

现在你的组件可以接受任意 `title` 属性了，你可以在应用的任何部分重用，只需要改动一下 `title` 值就可以：

```tsx
function HomePage() {
  return (
    <div>
      <Header title="React 💙" />
      <Header title="A new title" />
    </div>
  );
}
```

### 遍历列表

有时候按照列表的形式呈现数据也是很常见的。你可以使用数组方法来操作数据，然后生成样式上相同但内容不同的 UI 元素。

> React 并不关心如何获取数据，这也就意味着你可以采用任何你需要的方式解决这个问题。稍后我们会讨论 Next.js 的[数据抓取功能](https://nextjs.org/learn/basics/data-fetching)。但是目前为止，我们可以使用一个简单的数组来表示数据。
{: .prompt-tip}

向 `HomePage` 组件添加一个名称数组：

```tsx
function HomePage() {
  const names = ['Ada Lovelace', 'Grace Hopper', 'Margaret Hamilton'];
 
  return (
    <div>
      <Header title="Develop. Preview. Ship. 🚀" />
    </div>
  );
}
```

你可以使用 `array.map()` 方法遍历数组，然后用 **箭头函数** 把名称映射到列表元素。

```tsx
function HomePage() {
  const names = ['Ada Lovelace', 'Grace Hopper', 'Margaret Hamilton'];
 
  return (
    <div>
      <Header title="Develop. Preview. Ship. 🚀" />
      <ul>
        {names.map((name) => (
          <li>{name}</li>
        ))}
      </ul>
    </div>
  );
}
```

注意观察如何用花括号在 “JavaScript 之地”和 “JSX 之地”反复横跳。

如果你运行这段代码，React 会给你一个警告说缺少了 `key` 属性。这是因为 React 需要一个东西来唯一确定数组里的每个元素，这样它才知道如何在 DOM 里更新元素。

我们可以使用名称本身，因为它们目前就是唯一的，但是通常我们推荐用一些其他能保证唯一性的东西，比如说一个元素 ID。

```tsx
function HomePage() {
  const names = ['Ada Lovelace', 'Grace Hopper', 'Margaret Hamilton'];
 
  return (
    <div>
      <Header title="Develop. Preview. Ship. 🚀" />
      <ul>
        {names.map((name) => (
          <li key={name}>{name}</li>
        ))}
      </ul>
    </div>
  );
}
```

## 第七章 使用状态添加交互性

让我们来看看 React 是如何通过 **状态** 和 **事件处理器** 来帮助我们增加交互性的。

在我们的例子中，让我们在 `HomePage` 组件中创建一个点赞按钮。首先，在 `return()` 语句中添加一个按钮元素。

```tsx
function HomePage() {
  const names = ['Ada Lovelace', 'Grace Hopper', 'Margaret Hamilton'];
 
  return (
    <div>
      <Header title="Develop. Preview. Ship. 🚀" />
      <ul>
        {names.map((name) => (
          <li key={name}>{name}</li>
        ))}
      </ul>
 
      <button>Like</button>
    </div>
  );
}
```

### 监听事件

要让按钮被点击时做点什么，我们可以使用 `onClick` 事件。

```tsx
function HomePage() {
  // ...
  return (
    <div>
      {/* ... */}
      <button onClick={}>Like</button>
    </div>
  );
}
```

在 React 中，事件名称都是驼峰式命名的。`onClick` 事件是其中一个你可以回应用户交互的事件。其他的比如 `onChange` 可以回应用户输入，`onSubmit` 可以回应表格提交。

### 处理事件

我们可以定义一个函数来“处理”事件，在返回语句之前创建一个 `handleClick()` 函数。

```tsx
function HomePage() {
  // ...
 
  function handleClick() {
    console.log("increment like count")
  }
 
  return (
    <div>
      {/* ... */}
	  <button onClick={}>Like</button>
    </div>
     )
   }
```

然后我们可以在 `onClick` 事件被触发时调用 `handleClick` 函数。

```tsx
function HomePage() {
  // 	...
  function handleClick() {
    console.log('increment like count');
  }
 
  return (
    <div>
      {/* ... */}
      <button onClick={handleClick}>Like</button>
    </div>
  );
}
```

### 状态和钩子

React 中有一些函数被称为[钩子](https://react.dev/learn)。钩子可以允许我们给组件添加一些额外的逻辑，比如说 **状态**。你可以把状态想象为 UI 中会时常变化的东西，通常是由用户交互触发的。

![state](state.avif)

你可以 *使用状态* 来存储或者改变一个点赞按钮的点击次数。这就是这个 React 钩子的名称：`useState()`。

```tsx
function HomePage() {
  React.useState();
}
```

`useState()` 会返回一个数组，你可以通过 **数组解构** 来获取数组的值：

```tsx
function HomePage() {
  const [] = React.useState();
 
  // ...
}
```

数组的第一个值是状态 `value`，你可以起任意的名字，但是建议是具有一些描述性的：

```tsx
function HomePage() {
  const [likes] = React.useState();
 
  // ...
}
```

数组的第二个值是一个可以更新 `value` 的函数，你也可以起任意的名字，但通常建议使用状态名加 `set` 前缀：

```tsx
function HomePage() {
  const [likes, setLikes] = React.useState();
 
  // ...
}
```

你也可以给你的 `likes` 状态加一个初始值：0

```tsx
function HomePage() {
  const [likes, setLikes] = React.useState(0);
}
```

然后，我们可以检查一下这些状态在组件里是否能正常工作：

```tsx
function HomePage() {
  // ...
  const [likes, setLikes] = React.useState(0);
 
  return (
    // ...
    <button onClick={handleClick}>Like({likes})</button>
  );
}
```

最后，我们可以在 `handleClick()` 中添加这个状态更新函数 `setLikes` 了：

```tsx
function HomePage() {
  // ...
  const [likes, setLikes] = React.useState(0);
 
  function handleClick() {
    setLikes(likes + 1);
  }
 
  return (
    <div>
      {/* ... */}
      <button onClick={handleClick}>Likes ({likes})</button>
    </div>
  );
}
```

现在点击按钮就会调用 `handleClick` 函数，然后调用 `setLikes` 函数来给当前的状态加一。

> 属性是传递给组件的第一个函数参数，而状态是在组件内初始化和存储的。你可以把状态信息作为属性传递给子组件，但更新状态的逻辑代码应该保留在创建状态的组件中。
{: .prompt-tip}

### 管理状态

这只是对状态的一个入门介绍，对于管理状态和数据传递，我们还有很多要学的内容。推荐前往 React 文档的[增加交互性](https://react.dev/learn/adding-interactivity)和[管理状态](https://react.dev/learn/managing-state)章节学习更多内容。

## 第八章 从 React 到 Next.js

在前面的课程中，我们学习了如何使用 React。下面是我们最终的代码。

```html
<html>
  <body>
    <div id="app"></div>
 
    <script src="https://unpkg.com/react@17/umd/react.development.js"></script>
    <script src="https://unpkg.com/react-dom@17/umd/react-dom.development.js"></script>
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
 
    <script type="text/jsx">
      const app = document.getElementById("app")
 
      function Header({ title }) {
        return <h1>{title ? title : "Default title"}</h1>
      }
 
      function HomePage() {
        const names = ["Ada Lovelace", "Grace Hopper", "Margaret Hamilton"]
 
        const [likes, setLikes] = React.useState(0)
 
        function handleClick() {
          setLikes(likes + 1)
        }
 
        return (
          <div>
            <Header title="Develop. Preview. Ship. 🚀" />
            <ul>
              {names.map((name) => (
                <li key={name}>{name}</li>
              ))}
            </ul>
 
            <button onClick={handleClick}>Like ({likes})</button>
          </div>
        )
      }
 
      ReactDOM.render(<HomePage />, app)
    </script>
  </body>
</html>
```

我们刚刚入门了 React 的三大核心概念：**组件**，**属性** 和 **状态**。对这三大概念有坚实的基本了解会帮助我们学习构建 React 应用。如果你自我感觉良好，也可以看看下面的 React 内容：

- [React 如何处理渲染](https://react.dev/learn/render-and-commit)和[如何使用引用](https://react.dev/learn/referencing-values-with-refs)
- [如何管理状态](https://react.dev/learn/managing-state)
- [如何使用上下文传递深层嵌套的数据](https://react.dev/learn/passing-data-deeply-with-context)
- [如何使用 React API 钩子](https://react.dev/reference/react)，比如 `useEffect()`

### React 文档

这些年诞生了很多课程、视频和文章来帮助开发者学习 React。尽管要推荐一个符合大家学习方式的资源并不容易，但是 [React 文档](https://react.dev/)绝对是其中一个宝贵的资源，里面包含了可以交互的沙盒，以帮助我们练习学到的内容。

说到学习 React，**最好的方式就是练习**。你可以通过使用 `script` 标签往已有的项目中添加一些小组件来逐渐练习 React。但是很多开发者发现 React 所带来的开发和用户体验是如此优秀，以至于他们直接将他们的整个前端项目都用 React 改写了。

### 从 React 到 Next.js

尽管 React 在构建 UI 上很出色，但是要把构建的 UI 变成一个全功能可扩展的应用还是要下一番功夫的。好消息是，Next.js 可以处理这些配置的步骤，同时还包含了其他的功能来帮助我们构建 React 应用。

下面，我们会把我们的例子从 React 转换到 Next.js，并讨论 Next.js 是如何工作的，同时介绍一些网页开发概念来帮助我们学习更高级的 Next.js 特性。

## 第九章 安装 Next.js

要把 Next.js 添加到我们的项目中，我们就不需要 [unpkg.com](https://unpkg.com/) 的 `react` 和 `react-dom` 脚本了。相反，我们可以直接通过 Node 的包管理器 `npm` 把它们安装到本地。

> 你需要在电脑上安装 Node.js（[最低版本要求](https://nextjs.org/docs/upgrading#minimum-nodejs-version)），你可以[在这里下载](https://nodejs.org/en/)。
{: .prompt-tip}

下载前，我们先创建一个名为 `package.json` 的文件，并写入一个空对象：

```json
{}
```

打开终端，运行 `npm install react react-dom next`。安装完成后，你就可以在 `package.json` 文件中看到你的项目依赖了。

```json
{
  "dependencies": {
    "next": "^12.1.0",
    "react": "^17.0.2",
    "react-dom": "^17.0.2"
  }
}
```

你还会看到一个名为 `node_modules` 的新文件夹，里面放着实际的依赖文件（根据设置的不同，这个文件有可能是隐藏的）。

回到 `index.html` 文件，我们可以删除如下代码：

1. `react` 和 `react-dom` 脚本，因为我们已经使用 NPM 安装在本地了。
2. `<html>` 和 `<body>` 标签，因为 Next.js 会自动创建这些。
3. 跟 `app` 元素交互的代码和 `ReactDOM.render()` 方法。
4. `Babel` 脚本，因为 Next.js 自己有编译器，可以把 JSX 转换为合法的 JavaScript 代码。
5. `<script type="text/jsx">` 标签。
6. `React.useState(0)` 里面的 `React.`。

删除上面的代码后，在文件最开头添加一行 `import { useState } from "react"`，你最终的代码应该如下所示：

```tsx
import { useState } from 'react';
 
function Header({ title }) {
  return <h1>{title ? title : 'Default title'}</h1>;
}
 
function HomePage() {
  const names = ['Ada Lovelace', 'Grace Hopper', 'Margaret Hamilton'];
 
  const [likes, setLikes] = useState(0);
 
  function handleClick() {
    setLikes(likes + 1);
  }
 
  return (
    <div>
      <Header title="Develop. Preview. Ship. 🚀" />
      <ul>
        {names.map((name) => (
          <li key={name}>{name}</li>
        ))}
      </ul>
 
      <button onClick={handleClick}>Like ({likes})</button>
    </div>
  );
}
```

因为现在这个 HTML 文件里只剩下 JSX 代码了，所以你可以把文件后缀名从 `.html` 改为 `.js` 或者 `.jsx`。

现在，要想把它完全转为 Next.js 应用，你还需要做几件事：

1. 把 `index.js` 文件转移到一个名为 `pages` 的新文件夹中（我们稍后会讨论这个）。
2. 在主要的 React 组件前添加 `export default` 以帮助 Next.js 识别出哪个组件是当前页面要渲染的主组件。

```tsx
export default function HomePage() {
  // ...
}
```

3. 向 `package.json` 文件中添加一个 `scripts` 键，以允许你在开发时运行 Next.js 开发服务器。

```json
{
  "scripts": {
    "dev": "next dev"
  }
  "dependencies": {
     "next": "^11.1.0",
     "react": "^17.0.2",
     "react-dom": "^17.0.2"
  }
}
```

### 运行开发服务器

要查看能不能正常工作，你可以在终端里运行 `npm run dev` 然后在浏览器里打开 [localhost:3000](http://localhost:3000/)。然后你可以稍稍修改一点代码然后保存一下。

当你保存文件的时候，你应该会注意到，浏览器自动刷新了页面，并更新了你更改的内容。

这个功能叫[快速刷新](https://nextjs.org/docs/basic-features/fast-refresh)。当你更改或者重新配置 Next.js 的时候，这个功能会给你一个即时的反馈。

让我们回溯一下，我们的代码从这样

```html
<html>
  <body>
    <div id="app"></div>
 
    <script src="https://unpkg.com/react@17/umd/react.development.js"></script>
    <script src="https://unpkg.com/react-dom@17/umd/react-dom.development.js"></script>
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
 
    <script type="text/jsx">
      const app = document.getElementById("app")
 
      function Header({ title }) {
        return <h1>{title ? title : "Default title"}</h1>
      }
 
      function HomePage() {
        const names = ["Ada Lovelace", "Grace Hopper", "Margaret Hamilton"]
        const [likes, setLikes] = React.useState(0)
 
        function handleClick() {
          setLikes(likes + 1)
        }
 
        return (
          <div>
            <Header title="Develop. Preview. Ship. 🚀" />
            <ul>
              {names.map((name) => (
                <li key={name}>{name}</li>
              ))}
            </ul>
 
            <button onClick={handleClick}>Like ({likes})</button>
          </div>
        )
      }
 
      ReactDOM.render(<HomePage />, app)
    </script>
  </body>
</html>
```

变成了这样

```tsx
import { useState } from 'react';
 
function Header({ title }) {
  return <h1>{title ? title : 'Default title'}</h1>;
}
 
export default function HomePage() {
  const names = ['Ada Lovelace', 'Grace Hopper', 'Margaret Hamilton'];
  const [likes, setLikes] = useState(0);
 
  function handleClick() {
    setLikes(likes + 1);
  }
 
  return (
    <div>
      <Header title="Develop. Preview. Ship. 🚀" />
      <ul>
        {names.map((name) => (
          <li key={name}>{name}</li>
        ))}
      </ul>
 
      <button onClick={handleClick}>Like ({likes})</button>
    </div>
  );
}
```

从外表上看，这只是代码上的一点小小的删减，但是它也向我们展示了一些重要的事情：React 是一个 **库**，它提供了很多可以用于构建现代交互式 UI 的 **重要** 工具。当然，要把这些 UI 整合进一个应用里还是要些功夫的。

回看一下这个转移，你可能已经感觉到了那么一点使用 Next.js 的好处了。你移除了 babel 脚本，这是一个有点复杂的工具配置，不过你再也不用考虑这个问题了。你也见识到了快速刷新，这是一个你会在之后的 Next.js 开发生涯中经常用到的功能。

## 下一步

恭喜你创建了你的第一个 Next.js 应用！

总结一下，你学习了 React 和 Next.js 的基础知识，把一个简单的 React 应用转换为一个 Next.js 应用。

下一步，学习如何使用 Next.js 来[创建你的第一个应用](https://nextjs.org/learn/dashboard-app)吧——这个课程会向你介绍 **主要的** Next.js 特性以及练习构建一个更复杂一点的项目。
