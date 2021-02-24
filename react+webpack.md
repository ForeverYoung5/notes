# 简单的`demo`

我们先从一个简单的`demo`来看看具体的流程：

## 初始化

执行 `npm init ` 创建一个符合`node`规范的项目，创建之后会成一个`package.json`项目。

```
{
  "name": "tsdemo",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
  },
  "author": "",
  "license": "ISC",
}

```

## `react`

1. 执行  `npm i react react-dom` 安装 `react` 和 `react-dom` 。

2. 创建页面：

```
// ***src/App.tsx***

import React from 'react';

const App: React.FC = () => {
  return (<div>hello, world</div>);
};

export default App;

// ***src/index.tsx***

import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';

ReactDOM.render(<App />, document.getElementById('root'));

// ***src/index.html***
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="utf-8" />
    <meta name="viewport"
        content="width=device-width,initial-scale=1,minimum-scale=1,maximum-scale=1,user-scalable=no" />
    <title>react-app</title>
</head>
<body>
    <div id="root"></div>
</body>
</html>
```

## `typescript`

1. 执行 `npm i typescript -D   ` 安装 `typescript`

2. 创建配置文件：

```
// tsconfig.json
{
  "compilerOptions": {
    "outDir": "./dist",	// 输出的目录
    "module": "CommonJS",	// 指定生成哪个模块系统代码: "None"， "CommonJS"， "AMD"， "System"， "UMD"， "ES6"或 "ES2015"
    "target": "ES2015",	// 指定 ES 目标版本，默认 ES3
    "jsx": "react",	// 在 .tsx 文件里支持 jsx
    "declaration": true,	// 生成相应的 .d.ts 文件
    "removeComments": true,	// 删除所有注释，除了以 /!* 开头的版权信息。
  },
  "include": [
    "src/**/*",	 // 需要编译的文件
  ],
  "exclude": [
    "node_modules", // 不编译此文件夹下的文件
  ],
}
```

## `webpack`

1. 执行 `npm install webpack webpack-cli -S -D` 

`webpack` 用来打包，`webpack-cli` 让 `webpack` 命令可以执行

2. 项目中有`.tsx`文件，所以要安装`loader` 进行处理：执行 `npm install ts-loader --save-dev` 安装 `ts-loader` 对 `.tsx`文件进行打包。并进行如下配置。

3. 创建配置文件：

```
// ***webpack.config.js***
const path = require('path')

module.exports = {
    mode: 'development',  // 模式，当前为开发模式，还有个生产模式，生产模式会自动压缩编译后代码到一行
    entry: './src/index.tsx',
    module: {
        rules: [
            {
                test: /\.tsx?$/,    // ts-loader是官方提供的处理tsx的文件
                use: 'ts-loader',
                exclude: /node_modules/
            }
        ]
    },
    resolve:{
    	// 打包的时候报错：Module not found :Error : can't resolve 'App'
    	// 我们引入组件的时候，并没有加后缀(.tsx)，
    	// 此配置会按顺序为我们找App.js App.jsx...找到就返回，找不到会报错
        extensions: ['.js', '.jsx', '.ts', '.tsx', '.css', '.less', '.scss']
    },
    output:{
        filename:'bundle.js',
        path:path.resolve(__dirname,'dist')
    }
}
```

## 配置

最后在`package.json`文件中，配置如下

```
{
  "name": "tsdemo",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "build": "webpack" // 执行 npm run build 相当于 npx webpack
  },
  "author": "", 
  "license": "ISC",
  "dependencies": {
    "react": "^17.0.1",
    "react-dom": "^17.0.1"
  },
  "devDependencies": {
    "ts-loader": "^8.0.14",
    "typescript": "^4.1.3",
    "webpack": "^5.15.0",
    "webpack-cli": "^4.3.1"
  }
}

```

之后执行`npm run build`，就可以看到`dist`文件下打包好的文件。一个小的`demo`就完成了。

## `plugin`

经过上边的一些列操作之后，我们看到了`dist`下打包好的文件，但是并没有看到页面。可以在 `vscode` 中安装插件 `open in browser` ，之后在 `index.html` 中右击选中 `open in default browser`，就可以看到浏览器中出现了`html`页面。

但是存在一个新的问题：并没有显示 `App` 组件中的东西（`hello world`）。

我们需要在 `index.html`中引入打包好的文件(`bundle.js`):

```
 <script src="../dist/bundle.js"></script>
```

删除掉`dist`文件夹，从新打包 `npm run build`。

你会发现依旧没有任何东西，打开控制台，会报错：

```
Cannot read property 'createElement' of undefined
Consider adding an error boundary to your tree to customize error handling behavior.
```

这是因为 `react` 和 `react-dom` 引入不规范导致的，将所有引入改为：

```
import * as React from 'react'
import * as ReactDom from 'react-dom'
```

前边的过程中，存在三个问题：

1. 每次打包都要手动删除`dist` 文件夹
2. 手动添加打包后的 `bundle.js` 文件
3. `html` 页面并没有跑在 `localhost` 上，并且需要手动打开

接下来就逐步解决这些问题。

### [`clean-webpack-plugin`](https://www.npmjs.com/package/clean-webpack-plugin)

`clean-webpack-plugin` 的作用是在打包之前删除`output.path`下的所有内容之后再进行打包。所以不用传递参数。

1. 安装 ：`npm install --save-dev clean-webpack-plugin`

2. 引入并配置：

```
const { CleanWebpackPlugin } = require('clean-webpack-plugin');

module.exports = {
	plugins:[        
        new CleanWebpackPlugin()
    ],
}
```

这样就不用每次打包前手动删除`dist`文件夹了。

### [`html-webpack-plugin`](https://www.webpackjs.com/plugins/html-webpack-plugin/)

自行点击标题进入官网按照教程下载就好。

`HtmlWebpackPlugin`的作用是当打包完成时，以`src/index.html`文件为模板，生成`index.html`文件，并将打包好的`js`文件注入到里边。

```
const HtmlWebpackPlugin = require('html-webpack-plugin');
module.exports = {
	plugins:[        
        new HtmlWebpackPlugin({
            template: './src/index.html'
        })
    ],
}
```

打包后你会看到`dist`下会有个`html`文件。并且将打包好的`bundle.js`文件自动添加。打开这个HTML，我们可以看到正常的页面。这样我们就不用逐个把打包后的文件添加到HTML文件中（所入口打包的话可能会产生很多打包后的文件，逐个添加很麻烦）。

### [`webpack-dev-server`](https://www.npmjs.com/package/webpack-dev-server)

看官网自行安装。

```
module.exports = {
	devServer:{
        contentBase:'./dist', //借助devServer生成服务器放到dist目录下，但是dist目录下看不到任何东西，在内存中，这样可以提升速度
        open:true,  //可以自动打开网址不必手动打开
    },
}
```

配置命令：

```
  "scripts": {
    "build": "webpack",
    "start": "webpack serve"
    
    // "start": "webpack-dev-server" 
    // 报错Error: Cannot find module 'webpack-cli/bin/config-yargs'
    // code: 'MODULE_NOT_FOUND'
  },
```

运行 `npm run start`你会发现浏览器会自动打开窗口，并且当你修改完代码的啥时候，网页自动刷新。

## `loader`

### `.jsx`文件

执行`npm install -D babel-loader @babel/core @babel/preset-env @babel/preset-react`

根目录创建`.babelrc`文件并配置：

```
{
    "presets": ["@babel/preset-react","@babel/preset-env"]
}
```

配置`webpack`:

```
module:{
	rules:[
		{
          test: /\.jsx?$/,
          exclude: /node_modules/,
          use: {loader: "babel-loader"}
         }
	]
}
```

### `.css`文件

安装`style-loader` : `npm install --save-dev style-loader`

安装`css-loader` ：`npm install --save-dev css-loader`

配置如下：

```
{
	test: /\.css$/i,
    use: ['style-loader', 'css-loader'], //?modules 打开css-modules
},
```

`css-loader`负责分析各个文件之间的关系，将不同文件的代码生成一个代码块。`style-loader`负责将`css-loader`生成的代码块挂载到`head`中。所以`loader`使用顺序是从右向左，从下向上。

### `.less`文件

安装`npm i style-loader css-loader less-loader less -D`

配置：

```
 {
	test: /\.less$/,
    use: ['style-loader', 'css-loader', 'less-loader']
},
```

其中`less-loader`就是将`less`语法转化为`css`语法，之后就和`css`做一样的处理。

# `babel`

我们代码中经常会用到`es6`语法，但是低版本的浏览器并不支持`es6`，需要将其转化为`es5`。

可以去[`babel`官网](https://www.babeljs.cn/setup#installation)详细的了解下。

我们主要使用到的是`babel-loader`,`bable/core`还有`babel/preset-env`前边我们已经安装过。

接下来就是在配置文件中配置：

```
rules: [
            {   test: /\.(js|jsx)$/,, // 以js和jsx结尾的文件
                exclude: /node_modules/,   //文件不在node_modules文件夹下
                loader: "babel-loader" , //babel是js和babel的桥梁，但是并不完成代码转化
                options:{
                    "presets": ["@babel/preset-env"]  //代码转化
                  }
            },
		]
```

低版本的浏览器中不存在某些`es6`的对象，比如`map`之类的。这时候需要将这些不存在的函数或者对象补充到浏览器中。可能会想到`babelPolyfill` 但是他在注入的时候很多东西都是全局注入的。所以我们用`plugin-transform-runtime`

## `plugin-transform-runtime`

安装`npm install --save-dev @babel/plugin-transform-runtime` 和 `npm install --save @babel/runtime` 以及 `npm install --save @babel/runtime-corejs2`

配置如下：

```
		 {
                test: /\.(js|jsx)$/,
                exclude: /node_modules/,
                loader: "babel-loader",
                options: {
                	"presets": ["@babel/preset-env"]  //代码转化
                    "plugins": [
                        [
                            "@babel/plugin-transform-runtime",
                            {
                                "corejs": 2,
                                "helpers": true,
                                "regenerator": true,
                                "useESModules": false
                            }
                        ]
                    ]
                }

            },
```

`babel-loader`配置项`options`里面的内容可以单独抽出来放到根目录下`.babelrc`

## `.babelrc`文件配置`babel`

```
{
    "presets": [
        "@babel/preset-react",//之前配置.jsx文件添加的
        "@babel/preset-env"
    ],
    "plugins": [
        [
            "@babel/plugin-transform-runtime",
            {
                "corejs": 2,
                "helpers": true,
                "regenerator": true,
                "useESModules": false
            }
        ]
    ]
}
```



# `antd` 

执行`npm install antd --save`安装。

首先如果会使用到`icon`需要先安装：`npm install --save @ant-design/icons`

1. 报错

```
Support for the experimental syntax 'classProperties' isn't currently enable
```

解决：

`npm install --save-dev @babel/plugin-proposal-class-properties`

并配置`.babelrc`

```
"plugins": [
        "@babel/plugin-proposal-class-properties",
 ]
```

2. 报错：缺少处理`css`的`loader`:

解决：

`npm install --save-dev css-loader`

`npm install --save-dev style-loader`

并配置：

```
	{
        test: /\.css$/i,
        use: ['style-loader', 'css-loader'],
     },
```

在这个过程中，`css-loader`负责分析各个文件之间的关系，将不同文件的代码生成一个代码块。`style-loader`负责将`css-loader`生成的代码块挂载到`head`中。

## 全局引入样式

在入口文件（`index.tsx`）中全局引入样式：

```
import 'antd/dist/antd.css'
```

在使用组件的地方直接引入组件：

```
import * as React from 'react';
import {Button} from 'antd'

class Header extends React.Component {
  render() {
    return (
      <div>
        <Button type="primary">Primary</Button>
        <Button>Default</Button>
        <Button type="dashed">Dashed</Button>
        <Button type="danger">Danger</Button>
        <Button type="link">Link</Button>
      </div>
    );
  }
}
export default Header
```

## 局部引入样式

### 方法一：

入口文件中不再引入样式，在使用组件的页面引入：

```
import * as React from 'react';

import {Button} from 'antd'
import "antd/lib/button/style" //引入对应的样式

class Header extends React.Component {
  render() {
    return (
      <div>
        <div>header</div>
        <Button type="primary">Primary</Button>
        <Button>Default</Button>
        <Button type="dashed">Dashed</Button>
        <Button type="danger">Danger</Button>
        <Button type="link">Link</Button>
      </div>
    );
  }
}
export default Header
```

但是这样比较麻烦，每次使用组件都要引入对应的样式文件。

### 方法二（推荐）：[`babel-plugin-import`](https://www.npmjs.com/package/babel-plugin-import)

安装：`npm install babel-plugin-import --save-dev`

在`.babelrc`文件中配置：

```
"pulgins":[
	["import",{"libraryName": "antd","style": true}],
]
```

使用组件时直接引入组件即可：

```
import * as React from 'react';
import {Button} from 'antd'

class Header extends React.Component {
  render() {
    return (
      <div>
        <div>header</div>
        <Button type="primary">Primary</Button>
        <Button>Default</Button>
        <Button type="dashed">Dashed</Button>
        <Button type="danger">Danger</Button>
        <Button type="link">Link</Button>
      </div>
    );
  }
}
export default Header
```

## 坑坑坑~~~

**在这整个过程中，千万要保证没有开启`css Modules`功能，否则`className`会被编译，不再是原来样子，导致无论怎么配置，样式都是不起作用的。**

# `css-modules`

使`css`样式只在局部起作用。配置` Webpack `的`css-loader`插件，因为它对 `CSS Modules `的支持最好，而且很容易使用。

```
{
     test: /\.css$/i,
     use: ['style-loader', 'css-loader?modules'],
},
```

重点是在`css-loader`后边加上参数 `?modules` 打开`css modules` 功能。打开控制台你可以看到`className` 被编译成了类似于`Cijm8iX5re4lkWOQ2tG-p` 的形式。

## `less`

现在我们项目中越来越经常使用`less`。

### 安装

执行`npm install -g less`安装`less`。

### [使用](https://less.bootcss.com/)

### 打开`css-modules`

执行`npm i style-loader css-loader less-loader less -D` 安装要使用的`loader`

配置：

```
{
    test: /\.less$/,
	use: ['style-loader', 'css-loader?modules', 'less-loader']
}
```

如果运行时报错：`.bezierEasingMixin()`；将`less`版本降低到3.0以下，可以安装2.7.3。

## 第三方库：`antd`

项目中我使用了组件库`antd`，这是时候如果开启了`css modules`会导致引入的样式失效。原因是：`css modules`会把`className`编码成类似于`_1sNE87DmRAiZ8U-5Rbq8Zn`这样的形式，到时第三方库的样式对不上。

为了解决上述问题，我们可以进行如下配置：在`node_modules`中关闭`css modules`,在`src`下开启。

具体配置如下：

```
            {
                test: /\.css$/i,
                exclude: [/node_modules/], //不包含node_modules文件夹
                use: [
                    { loader: 'style-loader' },
                    {
                        loader: 'css-loader',
                        options: {
                            modules: true, //开启css-modules模式, 默认值为flase
                        }
                    }
                ],
            }, {
                test: /\.css$/i,
                use: ['style-loader', 'css-loader'],
                exclude: [/src/],   //不包含src文件夹
            },
            {
                test: /\.less$/,
                exclude: [/node_modules/],  //不包含node_modules文件夹
                use: [
                    'style-loader',
                    {
                        loader: 'css-loader',
                        options: {
                            importLoaders: 1, //在css-loader前应用的loader的数目, 默认为0
                            modules: true, //开启css-modules模式, 默认值为flase
                        }
                    },
                    'less-loader'
                ],
            },
            {
                test: /\.less$/,
                exclude: [/src/],  //不包含src文件夹
                use: ['style-loader', 'css-loader', 'less-loader'],
            },
```

重点就是通过`exclude`和`modules:true`进行配置。

# `DevServer`

`webpack-dev-server`是一个使用了`express`的`Http`服务器，它的作用主要是为了监听资源文件的改变，该`http`服务器和`client`使用了`websocket`通信协议，只要资源文件发生改变，`webpack-dev-server`就会实时的进行编译。

- `contentBase`：告诉服务器从哪里提供内容。
- `compress`：对服务启动`gzip`压缩。

- `headers`：在所有**响应**中添加首部内容。

- `host`：指定使用的`host`，如果希望服务器外部访问可设置为`0.0.0.0`

- `hot`：开启热模块替换特性。

  `DevServer` 默认行为是在发现源代码被更新后通过自动刷新整个页面来做到实时预览的，但是开启模块热替换功能后，它是通过在**不刷新整个页面的情况下通过使用新模块替换旧模块来做到实时预览的**。

- `open`：自动打开网页。

- `openPage`：指定打开浏览器时要浏览的页面。

- `overlay`：在编译出错的时候，在浏览器页面上显示错误。该属性值默认为false，需要的话，设置该参数为true。。

- `proxy`：代理。

  有单独的后端开发服务器 `API`，并且希望在同域名下发送 `API` 请求 ，那么代理某些 `URL` 会很有用。

  ```
  proxy: {
    //请求到 /api/users 现在会被代理到请求 http://localhost:3000/api/users。
    "/api": "http://localhost:3000"  
  }
  
  //如果你不想始终传递 /api ，则需要重写路径：
  proxy: {
    "/api": {
    	//请求中含有 /api 这样的域名,重定向到 'http://localhost:3000'来。
    	// axios.get('/api/widget?ajax=json&id=ad') 会自动补充前缀，
    	//也就是说，url: '/api/widget?ajax=json&id=ad' 
    	//等价于 url: 'http://localhost:3000/api/widget?ajax=json&id=ad'。
      target: "http://localhost:3000", 
      secure:true, 				//https 时使使用该参数
      hangeOrigin: true, 			// 是否跨域
      pathRewrite: {"^/api" : ""} // 重写路径，把url的地址里面含有 '/api' 这样的 替换成 ''
      //因此接口地址就变成了 http://localhost:3000/widget?ajax=json&id=ad
    }
  }
  
  
  ```

  配置`proxy`后，你在请求中看不到路由的改变，但是可以返回正常的接口数据。



## 定义项目的`IP`和端口

```
devServer: {
        host:'myip.com', 
        port:8081,         //开启服务的端口号
    },
```

在`SwitchHosts`中配置：

```
127.0.0.1 myip.com
```

访问`http://myip.com:8081/`就可以看到你的项目页面。

`SwitchHosts`可以自行百度下载，是一个管理`host`的工具。

# `MobX-state-tree`

`mobx-state-tree`（也称为“ MST”）是一种状态容器，它结合了**可变数据的简单易用**，**不可变数据的可追溯性**以及**可观察数据的反应性和性能**。基本概念的学习可以参考[这里](https://mobx-state-tree.js.org/intro/philosophy)。

说说在项目中该如何使用：

## 安装

执行`npm install mobx mobx-state-tree --save`进行安装。

## 准备

在使用之前我们先建设目录如下：

![image-20210128175943867](C:\Users\haoyujiao\AppData\Roaming\Typora\typora-user-images\image-20210128175943867.png)

- `components`：组件
- `pages`：页面
- `router`：路由
- `services`：放接口，从后端服务获取数据
- `store`：存放状态

`page`、`stores`、`services`一一对应。

## `MST`数据准备

安装

`npm install mobx-react --save`

报错：` Support for the experimental syntax 'decorators-legacy' isn't currently enabled` 没有装支持装饰器的组件。

解决：

执行`npm install --save-dev @babel/plugin-proposal-decorators`安装。

在`.babelrc`下配置如下：注意顺序，我的目前是在第一项（因为他也是从下向上执行的）

```
"plugins":[
	[
    	"@babel/plugin-proposal-decorators",
        {
        	"legacy": true
        }
    ],
]
```



