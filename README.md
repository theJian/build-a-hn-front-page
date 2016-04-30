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


## 编写大致的模板

上一节我们知道了整个页面的组件结构, 在这一节我们根据这些结果编写出一个大致的模板.
先在 `app` 目录下为每个组件建立文件, `NewsList.js`, `NewsHeader.js` 和 `NewsItem.js`.

```
cd app
touch NewsList.js NewsHeader.js NewsItem.js
```
编辑 `NewsHeader.js`
```
// NewsHeader.js

import React from 'react';

export default class NewsHeader extends React.Component {
  render() {
    return (
        <div className="newsHeader">
          I am NewsHeader.
        </div>
        );
  }
}
```

`NewsHeader` 组件就先这样, 具体的实现现在先不去考虑, 记得我们现在只是编写一个大致的结构.

同样的, 编辑 `NewsItem.js`

```
// NewsItem.js

import React from 'react';

export default class NewsItem extends React.Component {
  render() {
    return (
        <div className="newsItem">
          I am NewsItem.
        </div>
        );
  }
}
```

接着是 `NewsList.js`, 因为 `NewsList` 是前两个组件的容器, 所以我们需要引入它们
```
// NewsList.js

import React from 'react';
import NewsHeader from './NewsHeader.js';
import NewsItem from './NewsItem.js';

export default class NewsList extends React.Component {
  render() {
    return (
        <div className="newsList">
          <NewsHeader />
          <NewsItem />
        </div>
        );
  }
}

```

最后修改入口文件 `app.js`
```
// app.js

import React from 'react'
import { render } from 'react-dom';
import $ from 'jquery';
import NewsList from './NewsList.js';

render(<NewsList />, $('#content')[0]);
```

这时访问 <http://localhost:8080/build/index.html>

  ![]()

接下来让我们逐步完善各个组件.

## `NewsHeader`

整个 `NewsHeader` 包含了三个部分, 左边的Logo和标题, 中间的导航栏和右边的登录入口.

让我们先从左边的 Logo 和标题开始.

1. Logo 和标题

   先下载 Logo

   在 `NewsHeader` 组件里新增一个方法 `getLogo`, 就像下面这样.

   ```
   // NewsHeader.js

   // ...

   export default class NewsHeader extends React.Component {
     //...
     getLogo() {
       /* Do Something */
     }
     //...
   }
   ```
   这个方法返回一个包含 Logo 的子组件.
   ```
   getLogo() {
     return (
        <div className="newsHeader-logo">
          <a href="https://news.ycombinator.com/"><img src="PATH_TO_IMAGE" /></a>
        </div>
        );
   }
   ```
   这里遇到一个问题, img 的 src 填什么?
   
   在前面几节的开发中, 还记得你是怎么引入其他的 js 文件的吗? `import`. 实际上这是 ES6 的模块系统, 这里的 js 文件作为模块被其他模块引入. 但除了 js 文件, 在开发时我们还会涉及其他的资源文件, 如图像, 字体, 样式等, 它们也需要被模块化. 在这里, 如果 Logo 图片也能被模块化然后引入该多好. 我们需要再次配置 Webpack.
   
   安装对应的 loader:

   ```
   npm install url-loader file-loader --save-dev
   ```

   配置 `webpack.config.js`
   ```
   //...

     loaders: [
     {
       test: /\.jsx?$/,
       exclude: /node_modules/,
       loader: 'babel',
       query: {
         presets: ['es2015','react']
       }
     },
     {
       test: /\.(png|jpg|gif)$/,
       loader: 'url-loader?limit=8192' // 这里的 limit=8192 表示用 base64 编码 <= ８K 的图像
     }
     ]

   //...
   ```
   然后回到 `NewsHeader.js`

   这时候你就可以使用 `import` 引入图片了.
   ```
   import imageLogo from './y18.gif';
   ```
   然后像这样使用.
   ```
   <img src={imageLogo} />
   ```
   注意这里用`{}`包起来, 这样其中的内容会作为表达式.

   `getLogo` 方法完成了, 再写一个 `getTitle` 方法.
   ```
   getTitle() {
     return (
         <div className="newsHeader-title">
           <a className="newsHeader-textLink" href="https://news.ycombinator.com/">Hacker News</a>
         </div>
         );
   }
   ```
   修改 `render` 方法, 引用我们刚刚写好的那两个方法.
   ```
   render() {
     return (
         <div className="newsHeader">
           {this.getLogo()}
           {this.getTitle()}
         </div>
         );
   }
   ```
   还是一样别忘了 `{}`.

   最后我们需要添加样式, 还是回到刚刚的问题, 怎么引入样式?

   我们也需要将样式模块化.

   安装相应的 loader:
   ```
   npm install css-loader style-loader --save-dev
   ```
   `css-loader` 处理 css 文件中的 `url()` 表达式.

   `style-loader` 将 css 代码插入页面中的 style 标签中.

   在 `webpack.config.js` 中配置新的 loader.
   ```
   {
     test: /\.css$/,
     loader: 'style!css'
   }
   ```

   新建一个 css 文件 `NewsHeader.css`
   ```
   .newsHeader {
     align-items: center;
     background: #ff6600;
     color: black;
     display: flex;
     font-size: 10pt;
     padding: 2px;
   }

   .newsHeader-logo {
     border: 1px solid white;
     flex-basis: 18px;
     height: 18px;
   }

   .newsHeader-textLink {
     color: black;
     text-decoration: none;
   }

   .newsHeader-title {
     font-weight: bold;
     margin-left: 4px;
   }
   ```
   然后在 `NewsHeader.js` 中引入它
   ```
   import './NewsHeader.css';
   ```
   打包运行看看吧.
2. 导航栏
   接下来是导航栏, 也就是中间那部分.

   回到 `NewsHeader.js`, 增加一个 `getNav` 方法.
   ```
   getNav() {
     var navLinks = [
     {
       name: 'new',
       url: 'newest'
     },
     {
       name: 'comments',
       url: 'newcomments'
     },
     {
       name: 'show',
       url: 'show'
     },
     {
       name: 'ask',
       url: 'ask'
     },
     {
       name: 'jobs',
       url: 'jobs'
     },
     {
       name: 'submit',
       url: 'submit'
     }
     ];

     return (
         <div className="newsHeader-nav">
           {
             navLinks.map(function(navLink) {
               return (
                   <a key={navLink.url} className="newsHeader-navLink newsHeader-textLink" href={"https://news.ycombinator.com/" + navLink.url} >
                     {navLink.name}
                   </a>
                   );
             })
           }
         </div>
         );
   }
   ```
   同样, 记得在 `render` 方法中引用
   ```
   render() {
     return (
         <div className="newsHeader">
           {this.getLogo()}
           {this.getTitle()}
           {this.getNav()}
         </div>
         );
   }
   ```
   添加样式.
   ```
   .newsHeader-nav {
     flex-grow: 1;
     margin-left: 10px;
   }

   .newsHeader-navLink:not(:first-child)::before {
     content: ' | ';
   }
   ```
3. 登录入口
   
   增加一个 `getLogin` 方法.
   ```
   getLogin() {
     return (
         <div className="newsHeader-login">
           <a className="newsHeader-textLink" href="https://news.ycombinator.com/login?goto=news">login</a>
         </div>
         );
   }
   ```
   在 `render` 中引用
   ```
   render() {
     return (
         <div className="newsHeader">
           {this.getLogo()}
           {this.getTitle()}
           {this.getNav()}
           {this.getLogin()}
         </div>
         );
   }
   ```
   更新样式
   ```
   .newsHeader-login {
     margin-right: 5px;
   }
   ```

至此整个 `NewsHeader` 就完成了, 你应该能看到如本节初所展示的效果.
