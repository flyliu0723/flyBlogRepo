---
title: webpack基础
tags: ['JavaScript', 'webpack']
categories: 'webpack'
---

#### webpack基础知识

    webpack相关的知识一直是我的一个软肋，很早都想系统的学习一下，但是也一直没有真正的系统的看过一遍，这一次要把这些东西系统d的过一遍。

>  webpack是一个模块打包器，主要用于前端工程中的依赖梳理和模块打包，将我们开发中高可读性和可维护性的代码文件打包成浏览器可以识别并正常运行的压缩代码

      刚刚接触到webpack的人肯定会去想，它和grunt和gulp有什么区别？

      其实它们是没有可比性的，grunt和gulp的工作方式是：在一个配置文件中，指明对某些文件进行类似的编译，组合，压缩等任务的具体步骤。webpack的工作方式：把项目当作一个整体，通过一个主文件，webpack通过这个文件去发现项目的所有的依赖文件，最后打包成一个或多个浏览器可以识别的js文件。

##### webpack.config.js配置简介

        webpack.config.js是webpack的基础，可以说这个文件就是webpack的所有。

1. entry： 入口文件配置，Webpack 执行构建的第一步将从 Entry 开始，完成整个工程的打包。

2. Module：模块，在`Webpack`里一切皆模块，`Webpack`会从配置的`Entry`开始递归找出所有依赖的模块，最常用的是`rules`配置项，功能是匹配对应的后缀，从而针对代码文件完成格式转换和压缩合并等指定的操作。

3. Loader：模块转换器，用于把模块原内容按照需求转换成新内容，这个是配合`Module`模块中的`rules`中的配置项来使用。

4. Plugins：扩展插件，在`Webpack`构建流程中的特定时机注入扩展逻辑来改变构建结果或做你想要的事情。(插件`API`)

5. Output：输出结果，在`Webpack`经过一系列处理并得出最终想要的代码后输出结果，配置项用于指定输出文件夹，默认是`./dist`。

6. DevServer：用于配置开发过程中使用的本机服务器配置，属于`webpack-dev-server`这个插件的配置项。

##### 上手webpack配置

1. 安装nodejs

   webpack基于nodejs，所以我们首先需要的就是安装nodejs，怎么安装就不去介绍了，很简单，不会的百度。

2. 初始化项目

        首先我们创建一个webpack-test的文件夹，我们就在这个文件中进行webpack的配置。

        在这个文件夹中打开命令行，执行命令初始化项目

```
npm init
```

        这句命令会自动创建一个项目配置文件package.json。下面是我们的目录结构

```
|-- dist									打包输出目录
|-- node_mudules					npm安装的依赖包
|-- src										我们写代码的地方
	|-- components					模块文件
  |-- views								页面
			|-- foo
					|-- index.js
					|-- index.html
					|-- style.css
  |-- index.js						入口js
  |-- index.html					入口html
|-- package.json					项目配置信息
|-- webpack.config.js			webpack配置文件
```

##### 语法报错，代码规范

安装eslint，检查语法报错

```
npm install eslint eslint-config-enough babel-eslint eslint-loader --save-dev
```

--save和--save-dev的区别：

> `--save-dev`会把安装的包和版本号记录到`package.json`中的`devDependencies`对象中，还有一个`--save`， 会记录到`dependencies`对象中，它们的区别，我们可以先简单的理解为打包工具和测试工具用到的包使用`--save-dev`存到`devDependencies`， 比如 eslint、webpack。浏览器中执行的 js 用到的包存到`dependencies`， 比如 jQuery 等

eslint安装之后，我们需要在package.json中配置才会生效

```
{
  "eslintConfig": {
    "extends": "enough",
    "env": {
      "browser": true,
      "node": true
    }
  }
}
```

具体我们需要怎样的代码规范，怎样去配置，自己再去学习。下面我们介绍一下下载的这几个包的作用

* [babel-eslint](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fbabel%2Fbabel-eslint)是`eslint-config-enough`依赖的语法解析库，替代 eslint 默认的解析库以支持还未标准化的语法。比如[import()]

* [eslint-loader](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2FMoOx%2Feslint-loader)用于在 webpack 编译的时候检查代码，如果有错误，webpack 会报错。



##### 页面代码

上面我们写出了项目的结构，现在我们需要去填充一些内容，以便于我们看出webpack的效果。

首先是index.html

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
  </head>

  <body>
  </body>
</html>
```

然后是index.js

```javascript
// 引入 router
import router from './router'

// 启动 router
router.start()
```

然后我们要在src下创建一个router.js文件，这个就相当于vue的vue-router，负责更具url路径显示不同的页面内容

```javascript
// 引入页面文件
import foo from './views/foo'
import bar from './views/bar'

const routes = {
  '/foo': foo,
  '/bar': bar
}

// Router 类，用来控制页面根据当前 URL 切换
class Router {
  start() {
    // 点击浏览器后退 / 前进按钮时会触发 window.onpopstate 事件，我们在这时切换到相应页面
    // https://developer.mozilla.org/en-US/docs/Web/Events/popstate
    window.addEventListener('popstate', () => {
      this.load(location.pathname)
    })

    // 打开页面时加载当前页面
    this.load(location.pathname)
  }

  // 前往 path，变更地址栏 URL，并加载相应页面
  go(path) {
    // 变更地址栏 URL
    history.pushState({}, '', path)
    // 加载页面
    this.load(path)
  }

  // 加载 path 路径的页面
  load(path) {
    // 首页
    if (path === '/') path = '/foo'
    // 创建页面实例
    const view = new routes[path]()
    // 调用页面方法，把页面加载到 document.body 中
    view.mount(document.body)
  }
}

// 导出 router 实例
export default new Router()
```

在views中创建两个页面，用于页面的显示

首先是foo/index.js

```javascript
// 引入 router
import router from '../../router'

// 引入 html 模板，会被作为字符串引入
import template from './index.html'

// 引入 css, 会生成 <style> 块插入到 <head> 头中
import './style.css'

// 导出类
export default class {
  mount(container) {
    document.title = 'foo'
    container.innerHTML = template
    container.querySelector('.foo__gobar').addEventListener('click', () => {
      // 调用 router.go 方法加载 /bar 页面
      router.go('/bar')
    })
  }
}
```

根据这个js文件，我们可以看出我们需要去写一个index.html和一个style.css，这两个文件随便写点内容，看下效果就行。需要其他路径下的页面，依照这个进行编写。

接下来就是重点的webpack配置了。

##### 安装webpack和babel

```
npm install webpack webpack-cli webpack-dev-serve html-webpack-plugin html-loader css-loader style-loader file-loader url-loader --save-dev
```

* [webpack](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fwebpack%2Fwebpack)即 webpack 核心库。它提供了很多[API](https://link.juejin.im/?target=https%3A%2F%2Fwebpack.js.org%2Fapi%2Fnode%2F), 通过 Node.js 脚本中`require('webpack')`的方式来使用 webpack。

* [webpack-cli](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fwebpack%2Fwebpack-cli)是 webpack 的命令行工具。让我们可以不用写打包脚本，只需配置打包配置文件，然后在命令行输入`webpack-cli --config webpack.config.js`来使用 webpack, 简单很多。webpack 4 之前命令行工具是集成在 webpack 包中的，4.0 开始 webpack 包本身不再集成 cli。

* [webpack-dev-serve](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fwebpack-contrib%2Fwebpack-serve)是 webpack 提供的用来开发调试的服务器，让你可以用[http://127.0.0.1:8080/](https://link.juejin.im/?target=http%3A%2F%2F127.0.0.1%3A8080%2F)这样的 url 打开页面来调试，有了它就不用配置[nginx](https://link.juejin.im/?target=https%3A%2F%2Fnginx.org%2Fen%2F)了，方便很多。****

* [html-webpack-plugin](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fampedandwired%2Fhtml-webpack-plugin),[html-loader](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fwebpack%2Fhtml-loader),[css-loader](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fwebpack%2Fcss-loader),[style-loader](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fwebpack%2Fstyle-loader)等看名字就知道是打包 html 文件，css 文件的插件，大家在这里可能会有疑问，`html-webpack-plugin`和`html-loader`有什么区别，`css-loader`和`style-loader`有什么区别，我们等会看配置文件的时候再讲。

* [file-loader](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fwebpack%2Ffile-loader)和[url-loader](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fwebpack%2Furl-loader)是打包二进制文件的插件，具体也在配置文件章节讲解。

      接下来我们需要babel来转化代码，以便浏览器可以识别我们的代码

```
npm install babel-core babel-preset-env babel-loader --save-dev
```

* babel-core babel的核心编译器

* babel-preset-env是一个配置文件，我们可以使用这个文件转换es2015/2016/2017到es5

安装上之后并不会生效，这里我们还需要在package.json中加入babel配置

```json
{
  "babel": {
    "presets": ["env"]
  }
}
```

##### 配置webpack

现在就是重点中的重点了，webpack.config.js的配置

```javascript
const { resolve } = require('path')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const history = require('connect-history-api-fallback')
const convert = require('koa-connect')

// 使用 WEBPACK_SERVE 环境变量检测当前是否是在 webpack-server 启动的开发环境中
const dev = Boolean(process.env.WEBPACK_SERVE)

module.exports = {
  /*
  webpack 执行模式
  development：开发环境，它会在配置文件中插入调试相关的选项，比如 moduleId 使用文件路径方便调试
  production：生产环境，webpack 会将代码做压缩等优化
  */
  mode: dev ? 'development' : 'production',

  /*
  配置 source map
  开发模式下使用 cheap-module-eval-source-map, 生成的 source map 能和源码每行对应，方便打断点调试
  生产模式下使用 hidden-source-map, 生成独立的 source map 文件，并且不在 js 文件中插入 source map 路径，用于在 error report 工具中查看 （比如 Sentry)
  */
  devtool: dev ? 'cheap-module-eval-source-map' : 'hidden-source-map',

  // 配置页面入口 js 文件
  entry: './src/index.js',

  // 配置打包输出相关
  output: {
    // 打包输出目录
    path: resolve(__dirname, 'dist'),

    // 入口 js 的打包输出文件名
    filename: 'index.js'
  },

  module: {
    /*
    配置各种类型文件的加载器，称之为 loader
    webpack 当遇到 import ... 时，会调用这里配置的 loader 对引用的文件进行编译
    */
    rules: [
      {
        /*
        使用 babel 编译 ES6 / ES7 / ES8 为 ES5 代码
        使用正则表达式匹配后缀名为 .js 的文件
        */
        test: /\.(js|jsx)$/,

        // 排除 node_modules 目录下的文件，npm 安装的包不需要编译
        exclude: /node_modules/,

        /*
        use 指定该文件的 loader, 值可以是字符串或者数组。
        这里先使用 eslint-loader 处理，返回的结果交给 babel-loader 处理。loader 的处理顺序是从最后一个到第一个。
        eslint-loader 用来检查代码，如果有错误，编译的时候会报错。
        babel-loader 用来编译 js 文件。
        */
        use: ['babel-loader', 'eslint-loader']
      },

      {
        // 匹配 html 文件
        test: /\.html$/,
        /*
        使用 html-loader, 将 html 内容存为 js 字符串，比如当遇到
        import htmlString from './template.html';
        template.html 的文件内容会被转成一个 js 字符串，合并到 js 文件里。
        */
        use: 'html-loader'
      },

      {
        // 匹配 css 文件
        test: /\.css$/,

        /*
        先使用 css-loader 处理，返回的结果交给 style-loader 处理。
        css-loader 将 css 内容存为 js 字符串，并且会把 background, @font-face 等引用的图片，
        字体文件交给指定的 loader 打包，类似上面的 html-loader, 用什么 loader 同样在 loaders 对象中定义，等会下面就会看到。
        */
        use: ['style-loader', 'css-loader']
      },

      {
        /*
        匹配各种格式的图片和字体文件
        上面 html-loader 会把 html 中 <img> 标签的图片解析出来，文件名匹配到这里的 test 的正则表达式，
        css-loader 引用的图片和字体同样会匹配到这里的 test 条件
        */
        test: /\.(png|jpg|jpeg|gif|eot|ttf|woff|woff2|svg|svgz)(\?.+)?$/,

        /*
        使用 url-loader, 它接受一个 limit 参数，单位为字节(byte)

        当文件体积小于 limit 时，url-loader 把文件转为 Data URI 的格式内联到引用的地方
        当文件大于 limit 时，url-loader 会调用 file-loader, 把文件储存到输出目录，并把引用的文件路径改写成输出后的路径

        比如 views/foo/index.html 中
        <img src="smallpic.png">
        会被编译成
        <img src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACAAAAA...">

        而
        <img src="largepic.png">
        会被编译成
        <img src="/f78661bef717cf2cc2c2e5158f196384.png">
        */
        use: [
          {
            loader: 'url-loader',
            options: {
              limit: 10000
            }
          }
        ]
      }
    ]
  },

  /*
  配置 webpack 插件
  plugin 和 loader 的区别是，loader 是在 import 时根据不同的文件名，匹配不同的 loader 对这个文件做处理，
  而 plugin, 关注的不是文件的格式，而是在编译的各个阶段，会触发不同的事件，让你可以干预每个编译阶段。
  */
  plugins: [
    /*
    html-webpack-plugin 用来打包入口 html 文件
    entry 配置的入口是 js 文件，webpack 以 js 文件为入口，遇到 import, 用配置的 loader 加载引入文件
    但作为浏览器打开的入口 html, 是引用入口 js 的文件，它在整个编译过程的外面，
    所以，我们需要 html-webpack-plugin 来打包作为入口的 html 文件
    */
    new HtmlWebpackPlugin({
      /*
      template 参数指定入口 html 文件路径，插件会把这个文件交给 webpack 去编译，
      webpack 按照正常流程，找到 loaders 中 test 条件匹配的 loader 来编译，那么这里 html-loader 就是匹配的 loader
      html-loader 编译后产生的字符串，会由 html-webpack-plugin 储存为 html 文件到输出目录，默认文件名为 index.html
      可以通过 filename 参数指定输出的文件名
      html-webpack-plugin 也可以不指定 template 参数，它会使用默认的 html 模板。
      */
      template: './src/index.html',

      /*
      因为和 webpack 4 的兼容性问题，chunksSortMode 参数需要设置为 none
      https://github.com/jantimon/html-webpack-plugin/issues/870
      */
      chunksSortMode: 'none'
    })
  ],
  
  devServer: {
    port: 3824, // 端口
    host: 'localhost',
    overlay: true,
    compress: false // 服务器返回浏览器的时候是否启动gzip压缩
  },
  watch: true, // 监听文件更改，自动刷新
  watchOptions: {
    ignored: /node_modules/, //忽略不用监听变更的目录
    aggregateTimeout: 500, //防止重复保存频繁重新编译,500毫米内重复保存不打包
    poll:1000 //每秒询问的文件变更的次数
  },
}

```

这是最基础的webpack配置，对应的解释在代码注释中有详细解释，就不详细介绍了。真正项目中需要更多的设置，我们以后在慢慢添加学习。

现在先来在package.json中配置命令来执行webpack

```json
{
  "scripts": {
    "dev": "webpack-dev-server --open --mode development",
    "build": "webpack --mode production"
  }
}
```

#### 进阶配置

在上面的配置中，我们打包出来的代码只有一个文件，所有的文件都打包进了这个文件，这显然是不符合我们的要求的，等等吧。下面我们介绍几个我觉得有必要的配置。

##### 各个页面分开打包

分开打包的话，我们就可以访问一个页面的时候只去加载所需要的代码，优化项目性能。

我们可以用异步加载来实现这个功能

下面我们来修改router.js来实现这个功能

```javascript
// 将 async/await 转换成 ES5 代码后需要这个运行时库来支持
import 'regenerator-runtime/runtime'

const routes = {
  // import() 返回 promise
  '/foo': () => import('./views/foo'),
  '/bar.do': () => import('./views/bar.do')
}

class Router {
  // ...

  // 加载 path 路径的页面
  // 使用 async/await 语法
  async load(path) {
    // 首页
    if (path === '/') path = '/foo'

    // 动态加载页面
    const View = (await routes[path]()).default

    // 创建页面实例
    const view = new View()

    // 调用页面方法，把页面加载到 document.body 中
    view.mount(document.body)
  }
}
```

接下来我们还需要一个包来解析这些代码

```
npm install regenerator-runtime babel-preset-stage-2 --save-dev
```

* [regenerator-runtime](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Ffacebook%2Fregenerator%2Ftree%2Fmaster%2Fpackages%2Fregenerator-runtime)是[regenerator](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Ffacebook%2Fregenerator)的运行时库。Babel 通过插件[transform-regenerator](https://link.juejin.im/?target=https%3A%2F%2Fbabeljs.io%2Fdocs%2Fplugins%2Ftransform-regenerator)使用`regenerator`将[generator](https://link.juejin.im/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fen-US%2Fdocs%2FWeb%2FJavaScript%2FReference%2FStatements%2Ffunction*)函数和 async/await 语法转换成 ES5 语法后，需要运行时库才能正确执行。

* import也没有进入正式标准， 我们需要安装[babel-preset-stage-2](https://link.juejin.im/?target=https%3A%2F%2Fbabeljs.io%2Fdocs%2Fplugins%2Fpreset-stage-2%2F) 来支持import和其他的stage2的语法

同样的我们也需要在package.json中配置，使其生效

```json
{
  "babel": {
    "presets": [
      "env",
      "stage-2"
    ]
  }
}
```

然后修改webpack.config.js

```javascript
{
  output: {
    /*
    代码中引用的文件（js、css、图片等）会根据配置合并为一个或多个包，我们称一个包为 chunk。
    每个 chunk 包含多个 modules。无论是否是 js，webpack 都将引入的文件视为一个 module。
    chunkFilename 用来配置这个 chunk 输出的文件名。

    [chunkhash]：这个 chunk 的 hash 值，文件发生变化时该值也会变。使用 [chunkhash] 作为文件名可以防止浏览器读取旧的缓存文件。

    还有一个占位符 [id]，编译时每个 chunk 会有一个id。
    我们在这里不使用它，因为这个 id 是个递增的数字，增加或减少一个chunk，都可能导致其他 chunk 的 id 发生改变，导致缓存失效。
    */
    chunkFilename: '[chunkhash].js',
  }
}
```

##### 第三方库和业务代码分开打包

直接看配置

```javascript
{
  plugins: [
    // ...

    /*
    使用文件路径的 hash 作为 moduleId。
    虽然我们使用 [chunkhash] 作为 chunk 的输出名，但仍然不够。
    因为 chunk 内部的每个 module 都有一个 id，webpack 默认使用递增的数字作为 moduleId。
    如果引入了一个新文件或删掉一个文件，可能会导致其他文件的 moduleId 也发生改变，
    那么受影响的 module 所在的 chunk 的 [chunkhash] 就会发生改变，导致缓存失效。
    因此使用文件路径的 hash 作为 moduleId 来避免这个问题。
    */
    new webpack.HashedModuleIdsPlugin()
  ],

  optimization: {
    /*
    上面提到 chunkFilename 指定了 chunk 打包输出的名字，那么文件名存在哪里了呢？
    它就存在引用它的文件中。这意味着一个 chunk 文件名发生改变，会导致引用这个 chunk 文件也发生改变。

    runtimeChunk 设置为 true, webpack 就会把 chunk 文件名全部存到一个单独的 chunk 中，
    这样更新一个文件只会影响到它所在的 chunk 和 runtimeChunk，避免了引用这个 chunk 的文件也发生改变。
    */
    runtimeChunk: true,

    splitChunks: {
      /*
      默认 entry 的 chunk 不会被拆分
      因为我们使用了 html-webpack-plugin 来动态插入 <script> 标签，entry 被拆成多个 chunk 也能自动被插入到 html 中，
      所以我们可以配置成 all, 把 entry chunk 也拆分了
      */
      chunks: 'all'
    }
  }
}
```

##### 输出文件j加hash

```javascript
{
  output: {
    filename: dev ? '[name].js' : '[chunkhash].js'
  }
}
```




