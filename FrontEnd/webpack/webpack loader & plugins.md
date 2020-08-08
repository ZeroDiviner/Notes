# loader & plugins

loader 使得让 webpack 可以打包的文件变得不止是 js， 甚至可以在 js 文件中 import 一个 css

```javascript
module.exports = {
  module: {
    rules: [
      { test: /\.css$/, use: 'css-loader' },
      { test: /\.ts$/, use: 'ts-loader' }
    ]
  }
};
```

这个表示对于 ts 文件使用 ts-loader, 对于 css 文件使用 css-loader

## 使用 loader 的方法

有三种方式使用 loader

* 配置: 在 webpack.config.js 文件中指定
* 内联: 在每个 import 语句中显示指定 loader
* CLI: 在命令行中指定



## 配置方法

```javascript
 module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          { loader: 'style-loader' },
          {
            loader: 'css-loader',
            options: {
              modules: true
            }
          }
        ]
      }
    ]
  }
```



## 内联

```js
import Styles from 'style-loader!css-loader?modules!./styles.css';
```

这里使用! 将资源中的 loader 分开，分开的每个部分都相对于当前目录解析。



## CLI 调用

在命令行调用用`--module-bind`指定

```shell
webpack --module-bind jade-loader --module-bind 'css=style-loader!css-loader'
```

上面表示对于 jade 使用jade-loader, 对于 css 使用 style-loader 和 css-loader



## loader 特性

- loader 支持链式传递。能够对资源使用流水线(pipeline)。一组链式的 loader 将按照相反的顺序执行。loader 链中的第一个 loader 返回值给下一个 loader。在最后一个 loader，返回 webpack 所预期的 JavaScript。
- loader 可以是同步的，也可以是异步的。
- loader 运行在 Node.js 中，并且能够执行任何可能的操作。
- loader 接收查询参数。用于对 loader 传递配置。
- loader 也能够使用 `options` 对象进行配置。
- 除了使用 `package.json` 常见的 `main` 属性，还可以将普通的 npm 模块导出为 loader，做法是在 `package.json` 里定义一个 `loader` 字段。
- 插件(plugin)可以为 loader 带来更多特性。
- loader 能够产生额外的任意文件。





## Plugin

插件是 webpack 的[支柱](https://github.com/webpack/tapable)功能。webpack 自身也是构建于，你在 webpack 配置中用到的**相同的插件系统**之上！

插件目的在于解决 [loader](https://www.webpackjs.com/concepts/loaders) 无法实现的**其他事**。



用法：

```javascript
// webpack.config.js
const HtmlWebpackPlugin = require('html-webpack-plugin'); //通过 npm 安装
const webpack = require('webpack'); //访问内置的插件
const path = require('path');

const config = {
  entry: './path/to/my/entry/file.js',
  output: {
    filename: 'my-first-webpack.bundle.js',
    path: path.resolve(__dirname, 'dist')
  },
  module: {
    rules: [
      {
        test: /\.(js|jsx)$/,
        use: 'babel-loader'
      }
    ]
  },
  plugins: [
    new webpack.optimize.UglifyJsPlugin(),
    new HtmlWebpackPlugin({template: './src/index.html'})
  ]
};

module.exports = config;
```

