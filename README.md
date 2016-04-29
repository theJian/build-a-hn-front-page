# Learn React & Webpack by building the Hacker News front page

> 这篇教程仍然处于修订阶段, 可能会存在错误疏漏与不合理之处. 欢迎你参与修改和撰写.

这是一篇给初学者的教程, 在这篇教程中我们将通过构建一个 [Hacker News](https://news.ycombinator.com/) 的前端页面来学习 [React](https://facebook.github.io/react/) 与 [Webpack](https://webpack.github.io/). 它不会覆盖所有的技术细节, 因此它不会使一个初学者变成大师, 但希望能给初学者一个大致印象.

## 准备工作

- 安装 webpack

  在此之前你应该已经安装了 [node.js](https://nodejs.org/).
  ```
  npm install webpack -g

  ```
  参数`-g`表示我们将全局(global)安装 webpack, 这样你就能使用 `webpack` 命令了.

  webpack 也有一个 web 服务器 webpack-dev-server, 我们也安装上
  ```
  npm install webpack-dev-server -g
  ```

- webpack 配置文件
  
  webpack 使用一个名为 `webpack.config.js` 的配置文件, 现在在你的项目根目录下创建该文件.
  我们假设我们的工程有一个入口文件 `app.js`, 该文件位于 `app/` 目录下, 并且希望 webpack 将它打包输出为 `build/` 目录下的 `bundle.js` 文件. `webpack.config.js` 配置如下:

  ```
  var path = require('path');

  module.exports = {
    entry: path.resolve(__dirname, 'app/app.js'),
    output: {
      path: path.resolve(__dirname, 'build'),
      filename: 'bundle.js'
    }
  }
  ```
  现在让我们测试一下, 创建 `app/app.js` 文件, 填入一下内容:
  ```
  document.write('It works');
  ```
  创建 `build/index.html`, 填入以下内容:
  ```
  <!DOCTYPE html>
  <head>
    <meta charset="UTF-8">
    <title>Hacker News Front Page</title>
  </head>
  <body>
    <script src="./bundle.js"></script>  
  </body>
  </html>
  ```
  其中 script 引入了 `bundle.js`, 这是 webpack 打包后的输出文件.

  运行 `webpack` 打包, 运行 `webpack-dev-server` 启动服务器.
  访问 <http://localhost:8080/build/index.html>, 如果一切顺利, 你会看到打印出了 `It works`.

- 配置 `package.json`

  在项目根目录下运行 `npm init` 会生成 `package.json`, 修改 `scripts` 的键值如下:
  ```
  "scripts": {
    "start": "webpack-dev-server",
    "build": "webpack"
  }
  ```
  现在执行 `npm run build` 相当于 `webpack`, `npm run start` 相当于 `webpack-dev-server`.
  当项目变得相当复杂时, 你可以使用这种技巧隐藏其中的细节.

- 安装依赖

  安装 React:
  ```
  npm install react react-dom --save
  ```

  为了简化 AJAX 请求代码, 这里引入 jQuery:
  ```
  npm install jquery --save
  ```

  安装 [Babel](https://babeljs.io/) 的 loader 以支持 ES6 语法:
  ```
  npm install babel-core babel-loader babel-preset-es2015 babel-preset-react --save-dev
  ```

  然后配置 `webpack.config.js` 来使用安装的 loader.
  ```
  // webpack.config.js

  var path = require('path');

  module.exports = {
    entry: path.resolve(__dirname, 'app/app.js'),
    output: {
      path: path.resolve(__dirname, 'build'),
      filename: 'bundle.js'
    },
    module: {
      loaders: [
      {
        test: /\.jsx?$/,
        exclude: /node_modules/,
        loader: 'babel',
        query: {
          presets: ['es2015','react']
        }
      },
      ]
    }
  };
  ```

  接下来测试一下开发环境是否搭建完成.

  打开 `app.js`, 修改内容为:
  ```
  // app.js

  import React from 'react';
  import { render } from 'react-dom';

  class HelloWorld extends React.Component {
    render() {
      return (
          <div>Hello World</div>
          );
    }
  }

  render(<HelloWorld />, $('#content')[0]);
  ```
  这里组件 `HelloWorld` 被装载到 id 为 content 的 DOM 元素, 所以相应的需要修改 `index.html` .
  ```
  ...
  <body>
    <div id="content"></div>
    <script src="./bundle.js"></script>  
  </body>
  ...
  ```
  再次打包运行, 访问 <http://localhost:8080/build/index.html> 如果可以看到打印出了 Hello World 那么开发环境就算搭建完成了.

## 在开始写第一个组件之前 ...

在开始写第一个组件之前, 让我们分析一下到底我们需要哪些组件.
  
  ![]()
  
上图是的最终效果的部分截图, 我们将其分为几个组件, 用不同颜色的线框标出
  
  ![]()

我们可以看出其中包含以下几个组件.
  
  - `NewsList` (蓝色): 所有组件的容器.
  - `NewsHeader` (绿色): Logo, 标题, 导航栏等.
  - `NewsItem` (红色): 对应每条资讯.

把这些组成为一棵组件树.
  
  - `NewsList`
    + `NewsHeader`
	+ `NewsItem` * n


