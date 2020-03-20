# webpack，Babel，babel-loader的关系

本文将要介绍 webpack，Babel，babel-loader 的关系。理清楚他们各自做了什么事情。

通常我们新建一个项目，会先配置webpack，然后配置babel；babel是一个编译工具，实际上，babel也是可以单独使用的。

下面我们从Babel出发，简单配置一个react项目，来清晰认识一下webpack和babel的关系。

## Babel 和 Webpack 简介

Babel 是一个 JavaScript 编译器。（把浏览器不认识的语法，编译成浏览器认识的语法。）

webpack 是一个现代 JavaScript 应用程序的静态模块打包器。（项目打包）

下面会用到的：
|名称|描述|
|-|-|
|@babel/cli|Babel附带了一个内置的CLI，可用于从命令行编译文件。|
|@babel/core|使用本地配置文件|
|@babel/preset-env|编译最新版本JavaScript|
|@babel/preset-react|编译react|
|@babel/polyfill|通过 Polyfill 方式在目标环境中添加缺失的特性|
|@babel/plugin-proposal-class-properties|编译 class|

## 开始配置

新建项目

```shell
mkdir babel-in-webpack
```

进入项目

```shell
cd babel-in-webpack/
```

初始化 npm

```shell
npm init
```

不用管提示，一顿回车键。然后会生成一个文件 <code>package.json</code>

### 配置 Babel

安装 Babel 相关依赖

```shell
npm install --save-dev @babel/cli @babel/core @babel/preset-env @babel/polyfill
```

新建文件 <code>babel.config.json</code>

```json
{
  "presets": [
    "@babel/preset-env"
  ],
  "plugins": []
}
```

新建文件夹 <code>src</code>，<code>src</code> 内新建文件 <code>test.js</code>，随便写点啥es6语法。

<img style="width: 500px;" src="https://img2020.cnblogs.com/blog/1141466/202003/1141466-20200320211647973-443876499.png" />

使用下面命令编译

```shell
./node_modules/.bin/babel src --out-dir lib
```

<img style="width: 600px;" src="https://img2020.cnblogs.com/blog/1141466/202003/1141466-20200320212204224-399315473.png" />

编译完会新增目录<code>lib</code>, 里面放着编译好的文件

<img style="width: 500px;" src="https://img2020.cnblogs.com/blog/1141466/202003/1141466-20200320211752862-186141817.png">

### 配置 React

安装 <code>Babel</code> 编译 <code>React</code> 的依赖

```shell
npm install --save-dev @babel/preset-react @babel/plugin-proposal-class-properties
```

<code>babel.config.json</code> 添加 <code>React</code> 相关配置

```json
{
  "presets": [
    "@babel/preset-env",
    "@babel/preset-react"
  ],
  "plugins": [
    "@babel/plugin-proposal-class-properties"
  ]
}
```

安装 <code>React</code> 相关依赖

```shell
npm install --save react react-dom
```

<code>src</code> 下新增 <code>react</code> 文件 <code>main.js</code>

```js
import React from 'react';
import ReactDOM from 'react-dom';

class App extends React.Component {
  render() {
    return (
      <div>Hello World!</div>
    )
  }
}

ReactDOM.render(<App />, document.getElementById('root'));
```

运行命令编译

```shell
./node_modules/.bin/babel src --out-dir lib
```

编译完成后 <code>lib</code> 下多了一个 <code>main.js</code>

<img style="width: 800px;" src="https://img2020.cnblogs.com/blog/1141466/202003/1141466-20200320213132647-847838181.png" />

看起来编译很成功, 我们在 <code>lib</code> 下面新建一个 html 引入 <code>main.js</code> 看看效果

<img style="width: 600px;" src="https://img2020.cnblogs.com/blog/1141466/202003/1141466-20200320174059014-369351006.png" />

报错，浏览器不认识require，继续往下看。

### 配置 webpack

安装 <code>webpack</code> 依赖

```shell
npm install --save-dev webpack webpack-cli
```

根目录新建文件 <code>webpack.config.js</code>

```js
const path = require('path');

module.exports = {
  entry: './src/main.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].js'
  }
};
```

在 <code>package.json</code> 的 <code>scripts</code> 中加入命令

```txt
"build": "webpack --mode development",
```

运行命令

```shell
npm run build
```

<img style="width: 800px;" src="https://img2020.cnblogs.com/blog/1141466/202003/1141466-20200320215050350-389680613.png" />

<code>webpack</code> 不认识 <code>react</code> 语法，在 <code>webpack.config.js</code> 中加入 <code>babel-loader</code>。

```js
const path = require('path');

module.exports = {
  entry: './src/main.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].js'
  },
  module: {
    rules: [
      { test: /\.js$/, use: 'babel-loader' }
    ]
  },
  plugins: []
};
```

安装依赖 <code>babel-loader</code>

```shell
npm install --save-dev babel-loader
```

运行命令

```shell
npm run build
```

<img style="width: 800px;" src="https://img2020.cnblogs.com/blog/1141466/202003/1141466-20200320221633584-77810522.png" />

会看到 <code>dist/main.js</code>, 写个html引入试试

<img style="width: 300px;" src="https://img2020.cnblogs.com/blog/1141466/202003/1141466-20200320222151486-254242323.png" />

### 两种编译结果对比

我们来看 <code>Babel</code> 编译结果 <code>lib/main.js</code> 和 <code>webpack</code> 编译结果 <code>dist/main.js</code>，发现 <code>Babel</code> 仅仅是将 <code>src/main.js</code> 的react语法编译成了js语法，而 <code>webpack</code> 将 <code>src/main.js</code> 和引入的 <code>node_modules</code> 融合后用 <code>Babel</code> 编译。

浏览器不认识 <code>require</code>，<code>webpack</code> 实现了一套浏览器认识的 <code>require</code>。

## 总结

<code>Babel</code> 是编译工具，把高版本语法编译成低版本语法，或者将文件按照自定义规则转换成js语法。

<code>webpack</code> 是打包工具，定义入口文件，将所有模块引入整理后，通过loader和plugin处理后，打包输出。

<code>webpack</code> 通过 <code>babel-loader</code> 使用 <code>Babel</code> 。

[代码地址：GitHub](https://github.com/whosMeya/babel-in-webpack)

<br />
