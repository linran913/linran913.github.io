---
layout: blog
title: "[译] Creating a React App… From Scratch."
date: 2022-10-15 14:57:36 +0800
author: "林染"
category: Translation
published: true
---

> 原文：[https://medium.com/@JedaiSaboteur/creating-a-react-app-from-scratch-f3c693b84658](https://medium.com/@JedaiSaboteur/creating-a-react-app-from-scratch-f3c693b84658)
>
> 作者：[Jedai Saboteur](https://medium.com/@JedaiSaboteur)
>
> 代码：[https://github.com/paradoxinversion/creating-a-react-app-from-scratch](https://github.com/paradoxinversion/creating-a-react-app-from-scratch)

## 从零开始构建 React 应用

![ ](/assets/pic/1_BbK-XuJKmIpB0T8MeTndZQ.png)

React 并不是开箱即用的。它使用的关键字和语法 Node （直到本教程中所使用的 9.3.0 版本）尚未支持。它需要相当多很难弄清楚的设置，因而 Facebook 提供了一个轻松建立 React 应用的选项。何需苦恼呢，是吧？

事实上，[`create-react-app`](https://github.com/facebook/create-react-app) 命令抽像出许多过程，使得 React 应用建立的细节离你远去——至少你不需要弹出它并手动调整所有的选项。你可能有很多想要自己实现它的原因，或者至少想要对它内部所做的事情心中有数。

正如我提到的，开始建立一个 React 应用有几个难关。首先是 Node 不能处理所有的语法（例如 [import/export](https://medium.com/@JedaiSaboteur/import-export-babel-and-node-a2e332d15673) 和 JSX）。其次是在开发应用程序的过程中你会需要编译文件或者以某种方式提供文件使其工作——这在后一种情况下很重要。

幸运的是我们可以用 [Bable](http://babeljs.io/) 和 [Webpack](https://webpack.js.org/) 处理这些问题。

### 安装

首先，为你的 React 应用创建一个新目录。然后，用 `npm init` 命令初始化你的项目并在你选择的编辑器里打开它。此时也是运行 `git init` 命令的好时机。在你的新项目文件夹中建立以下结构：

``` 
 .
 +-- public
 +-- src
```

稍微提前考虑一下，我们最终会编译应用并且很有可能想要把编译后的版本和 Node 模块库排除在 Git 的提交外，所以继续添加一个 `.gitignore` 文件（至少）把 `node_modules` 和 `dist` 目录排除。

我们的 `public` 文件夹会保存所有的静态资源文件，最重要的是包含 `index.html` 文件，React 会利用它来渲染应用。接下来的代码来源于 React 文档并作了些许修改。把下示的 HTML 代码复制到 `public` 目录下一个名为 `index.html` 的新文件中。

``` html
<!-- sourced from https://raw.githubusercontent.com/reactjs/reactjs.org/master/static/html/single-file-example.html -->
<!DOCTYPE html>
<html>

<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
  <title>React Starter</title>
</head>

<body>
  <div id="root"></div> <!-- 根节点 -->
  <noscript>
    You need to enable JavaScript to run this app.
  </noscript>
  <script src="../dist/bundle.js"></script>
</body>

</html>
```

需要注意的最重要的是第 10 行，这是 React 应用控制（hook）的根节点。以及第 14 行，引用了即将编译的 React 应用。你可以随意命名你的编译脚本，但我在本教程中会使用 `bundle.js`。

既然现在我们已经设置好了 HTML 页面，是时候切入正题了。我们需要再设置一些更多的东西。首先，我们需要确保编写的代码能够被编译，所以我们将会用到 Babel。

### **Babel**

继续运行 `npm install --save-dev @babel/core@7.1.0 @babel/cli@7.1.0 @babel/preset-env@7.1.0 @babel/preset-react@7.0.0` 命令。

`babel-core` 是 Babel 主要的包——我们需要用它让 Babel 对代码做任何转换。 `babel-cli` 允许你在命令行中编译文件。`preset-react` 和 `preset-env` 都是转换特定代码风格的预设——在本例中， `env` 预设（preset）允许我们把 ES6+ 转换为兼容性更好的 JavaScript 代码， `react` 预设也是如此，但转换的是 JSX。

在项目根目录中，创建一个名为 `.babelrc` 的文件。在这里，我们告诉 Babel 要使用 `env` 和 `react` 预设。

``` json
{
  "presets": ["@babel/env", "@babel/preset-react"]
}
```

Babel 也有其它大量的插件可供使用，如果你只需要转换特定的功能，或者你需要的某些功能没有被 `env` 预设覆盖，就可以使用这些插件。我们暂时不必担心这些，不过你可以点击[这里](https://babeljs.io/docs/plugins/)查看它们。

### **Webpack**

现在我们需要获取和配置 Webpack。我们需要更多的包，需要将它们保存为开发依赖项：`npm install --save-dev webpack@4.19.1 webpack-cli@3.1.1 webpack-dev-server@3.1.8 style-loader@0.23.0 css-loader@1.0.0 babel-loader@8.0.2`。

Webpack 使用 [加载器（loaders）](https://webpack.js.org/loaders/)处理不同类型的文件来进行打包。它可以轻松地和开发服务器一起运行，我们在开发中会使用它为 React 项目提供服务并且在 React 组件发生（保存）更改时重新加载浏览器页面。为了使用这些服务，需要配置 Webpack 使用我们安装的加载器并准备 dev-server 。

在项目根目录创建一个名为 `webpack.config.js` 的新文件。这个文件导出了一个包含 Webpack 配置的对象。

``` javascript
const path = require("path");
const webpack = require("webpack");

module.exports = {
  entry: "./src/index.js",
  mode: "development",
  module: {
    rules: [
      {
        test: /\.(js|jsx)$/,
        exclude: /(node_modules|bower_components)/,
        loader: "babel-loader",
        options: { presets: ["@babel/env"] }
      },
      {
        test: /\.css$/,
        use: ["style-loader", "css-loader"]
      }
    ]
  },
  resolve: { extensions: ["*", ".js", ".jsx"] },
  output: {
    path: path.resolve(__dirname, "dist/"),
    publicPath: "/dist/",
    filename: "bundle.js"
  },
  devServer: {
    contentBase: path.join(__dirname, "public/"),
    port: 3000,
    publicPath: "http://localhost:3000/dist/",
    hotOnly: true
  },
  plugins: [new webpack.HotModuleReplacementPlugin()]
}
```

让我们快速浏览一下这些配置：[`entry`](https://webpack.js.org/configuration/entry-context/#entry) （第5行）告诉 Webpack 我们的应用程序从哪里开始以及从哪开始打包文件。下一行让 Webpack 知道目前处于开发模式下工作——这使我们不必在运行开发服务器时添加一个模式标志。

[`module`](https://webpack.js.org/configuration/module) 对象帮助定义如何转换导出的 JavaScript 模块以及根据给定的 [`rules`](https://webpack.js.org/configuration/module#module-rules) 数组包含哪些模块。

第一条规则是关于转换我们的 ES6 和 JSX 语法。[`test`](https://webpack.js.org/configuration/module#rule-test) 和 [`exclude`](https://webpack.js.org/configuration/module#rule-exclude) 属性是匹配文件的条件。在本例中，它会匹配任何在 `node_modules` 和 `bower_components` 目录外的文件。由于我们同样会 *转换（transforming）* `js` 和 `jsx` 文件，需要引导 Webpack 使用 Babel。最后，我们在[选项](https://webpack.js.org/configuration/module#rule-options-rule-query)中指定想要使用 `env` 预设。

下一条规则用来处理 CSS 文件。 由于我们没有前后置处理（ pre-or-post-processing）我们的 CSS 文件。我们只需确保在 `use` 属性中添加了 `style-loader` 和 `css-loader`。 `css-loader` 需要 `style-loader` 才能工作。当只使用一个加载器时，[`loader (Rule.loader)`](https://webpack.js.org/configuration/module/#ruleloader) 是 `use (Rule.use: [ { loader } ])` 属性的简写。

[`resolve`](https://webpack.js.org/configuration/resolve) 属性允许我们指定 Webpack 将要解析的文件的扩展名——这允许我们引入模块而无需添加它们的扩展名。

[`output`](https://webpack.js.org/configuration/output) 属性告诉 Webpack 在哪输出打包好的代码。[`publicPath`](https://webpack.js.org/configuration/output#output-publicpath) 属性指定了打好的包输出的目录，并且也告诉了 `webpack-dev-server` 从哪里提供文件服务。

[`publicPath`](https://webpack.js.org/configuration/output#output-publicpath) 属性是一个帮助我们使用 dev-server 的特殊属性。它指定了目录的公共URL——至少在 `webpack-dev-server` 知道或关心的范围。如果设置不正确你将会得到 404，因为服务器无法从正确的地址提供文件服务。

我们在 `webpack-dev-server` 设置 [`devServer`](https://webpack.js.org/configuration/dev-server) 属性。比起我们的需求，这并不要求许多——只需我们静态文件服务的位置（例如我们的 `index.html` 文件）以及我们想要服务器运行的端口。注意 `devServer` 也有一个 [`publicPath`](https://webpack.js.org/configuration/dev-server/#devserver-publicpath-) 属性。这个 `publicPath` 告诉服务器我们打包后的代码的真实位置。

最后一点可能有点令人困惑——这里要特别注意：[`output.publicPath`](https://webpack.js.org/configuration/output/#output-publicpath) 和 [`devServer.publicPath`](https://webpack.js.org/configuration/dev-server/#devserver-publicpath-) 是不同的。请把这两个条目重读一遍。

最后，由于我们想使用 [Hot Module Replacement（HMR）](https://webpack.js.org/guides/hot-module-replacement/) 所以我们不必不断刷新去查看更改。我们为其所做对这个文件来说是在 [`plugins`](https://webpack.js.org/configuration/plugins/) 属性中实例化一个新的插件实例并确保我们在 `devServer` 中将 `hotOnly` 设置为 `true` 。不过，在 HMR 开始工作前，还有一样东西我们需要在 React 中设置。

我们已经完成繁杂的设置。现在让 React 运行起来吧！

### **React**

首先，我们需要再获取两个包：`react@16.5.2` 和 `react-dom@16.5.2`。继续并将它们保存为常规依赖项。

我们需要告诉我们的 React 应用从哪挂载到 DOM（在 `index.html`中）。在 `src` 目录中创建一个名为 `index.js` 的文件。这是一个很小的文件但是在你的 React 应用中却起了很大作用。

``` javascript
import React from "react";
import ReactDOM from "react-dom";
import App from "./App.js";

ReactDOM.render(<App />, document.getElementById("root"));
```

`ReactDOM.render` 函数告诉 React 该渲染什么以及在哪里渲染它——在本例中我们渲染一个名为 App 的组件（我们稍后创建它），它会被渲染在 ID 为 root （`index.html` 第10行中）的 DOM 元素中。

现在，在 `src` 中创建另一个名为 `App.js` 的文件。如果你在React 中使用 `create-react-app` ，你会十分熟悉这部分。该文件只是一个 React 组件。

``` javascript
import React, { Component} from "react";
import "./App.css";

class App extends Component{
  render(){
    return(
      <div className="App">
        <h1> Hello, World! </h1>
      </div>
    );
  }
}

export default App;
```

虽然我们还在这里，但我确实提到 Webpack 也处理 CSS 文件（我们需要它作为组件）。让我们在 `src` 目录下添加一个很简单的样式表。

``` css
.App {
  margin: 1rem;
  font-family: Arial, Helvetica, sans-serif;
}
```

你项目的最终目录结构应该和下面看起来一样，除非你在过程中更改过一些文件名：

``` 
.
+-- public
| +-- index.html
+-- src
| +-- App.css
| +-- App.js
| +-- index.js
+-- .babelrc
+-- .gitignore
+-- package-lock.json
+-- package.json
+-- webpack.config.js
```

我们现在拥有了一个功能正常的 React 应用！我们可以通过在终端中执行 `webpack-dev-server --mode development` 来启动开发服务器。我建议把它放到 `package.json` 中的 `start` 脚本中，这样每次仅需九次按键。

### **完成HMR**

如果你现在运行服务器， 你会注意到你的任何更改都不会导致客户端产生变化。怎么回事？

HMR 需要知道实际替换什么内容，目前我们还没有给它任何东西。为此，我们将使用一名 React 团队成员提供给我们的一个包：`react-hot-loader@4.3.11` 。

你将其安装为常规依赖项——就像文档中出现的任何一个包。

> *注意：你可以放心地把 react-hot-loader 安装为常规依赖项，而不是作为一个开发环境的依赖。因为它自动确保它不会在生产环境中执行并占用空间很小。*

现在，在 App.js 中导入 react-hot-loader 并将导出的对象标记为热更新，代码修改如下。

``` javascript
import React, { Component} from "react";
import {hot} from "react-hot-loader";
import "./App.css";

class App extends Component{
  render(){
    return(
      <div className="App">
        <h1> Hello, World! </h1>
      </div>
    );
  }
}

export default hot(module)(App);
```

当现在运行你的应用程序时，对代码的更改应该在保存后立即更新在客户端。

### **最后的细节**

在启动项目时，你可能会注意到一些有趣（或可能令人吃惊）的事情：编译的文件从未在 `dist` 目录出现。`webpack-dev-server` 事实上从内存中提供文件服务——一旦服务器停止它们就消失了。为了真正编译文件，我们会正确使用 Webpack ——在 `package.json` 中添加一个名为 `build` 的脚本，内容是：`webpack --mode development.`。你可以用 `development` 替换其中的 `production` 。但当你省略 `--mode` ，它会回退到后者并给你一个警告。

以上涵盖了你能够渲染基本 React 应用所需要的一切，而无需使用 `create-react-app` 。不过，还有更多内容需要添加到实现中以使其更加完整——实际上图像文件还没有设置为由 Webpack 处理，但有一个[加载器](https://webpack.js.org/loaders/file-loader/)处理它。我会把这个实现留给你。毕竟如果你不需要或是不想提供文件，那只是累赘，对吧？

希望这篇文章帮你照亮了如何建立一个 React 应用的细节以及其幕后的基础工作。我没有深入讨论 Babel 和 Webpack 的细节，但请探索文中添加的任一链接或直接查看它们的文档。它们是很棒的工具，咋一看比实际上还令人生畏，它们可以帮助把你的开发水平提升到更深的层次。

如果你仍然不清楚其中任何一个或想要其他参考，这里是[代码的 Github 仓库](https://github.com/paradoxinversion/creating-a-react-app-from-scratch)。 你还可以查看[一个先前的实现](https://github.com/paradoxinversion/react-starter)（它更深入）。

> *本文编辑于2018年4月24日添加了包的版本；2018年5月13日，修复有关 webpack-dev-server 的错误；2018年6月16日更新了本教程中用到的 React 和其他包；2018年9月23日同上。*
