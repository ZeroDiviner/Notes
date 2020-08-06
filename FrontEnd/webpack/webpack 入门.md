# Webpack 入门

Webpack 本质上是 JavaScript静态模块打包器，当 webpack 处理应用程序的时候，它会**递归的构建一个关系依赖图**。其中包含应用程序所需要的每一个模块，然后将这些模块打包成一个或者多个 bundle。

本身给我的感觉就像是在 linux 下写 c 时候要写的 makefile 文件，告诉编译器一个文件所需要的依赖文件，以及输出文件名称等等，只不过 webpack 作为 js 下的打包工具，功能相对于c 的 makefile 更加强大，有 loader 和 plugin 的存在，甚至可以解析一些非 js 的文件作为依赖， 这让关系依赖图更加复杂。

在开始前先理解四个概念

## 入口 entry

entry 项用来告诉 webpack，从哪个文件开始可以构建依赖图， 进入入口起点之后，webpack 会找出有哪些模块和库是入口起点（直接和间接）依赖的。

每个文件盒依赖被递归处理，最后输出到称之为 *bundles* 的文件中。

可以看到下面的 entry 属性指出了一个入口起点，默认值为'./src'

```javascript
//webpack.config.js
module.exports = {
  entry: './path/to/my/entry/file.js'
};
```



## 出口output

output 属性告诉 webpack 在哪里输出它生成的bundles 文件，以及如何命名这些文件。可以在配置文件中指出一个`output` 字段来配置这些处理过程。

```javascript
//webpack.config.js
const path = require('path');

module.exports = {
  entry: './path/to/my/entry/file.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'my-first-webpack.bundle.js'
  }
};
```

上面通过配置了 output.path 和 output.filename 来告诉了 webpack 把文件输出在哪，叫什么名字。



## Loader

理论上来说 webpack 是只能识别 js文件的，但是有时候项目中会有其他类型的文件，例如 txt 文件， 这时候就需要 loader，loader 可以将所有类型的文件转化为 webpack 可以处理的文件类型，然后就可以利用 webpack 的打包能力，对其进行处理。

```javascript
//webpack.config.js
const path = require('path');

const config = {
  output: {
    filename: 'my-first-webpack.bundle.js'
  },
  module: {
    rules: [
      { test: /\.txt$/, use: 'raw-loader' }
    ]
  }
};

module.exports = config;
```

上面在 module.rules 中定义的就是对于 txt 文件，在 webpack 对其进行打包之前，先要使用 raw-loader转换一下。



## 插件 plugins

loader 负责进行转化某种类型的文件，而 plugins 则功能更加强大，例如：

* 打包优化
* 压缩
* 重新定义环境中的变量

想要使用插件的时候，需要在配置文件中 require 它，然后添加到 plugins 数组中。也可以在一个页面中多次使用同一个插件，这时候需要用 new 操作符创建新的实例。

```javascript
//webpack.config.js
const HtmlWebpackPlugin = require('html-webpack-plugin'); // 通过 npm 安装
const webpack = require('webpack'); // 用于访问内置插件

const config = {
  module: {
    rules: [
      { test: /\.txt$/, use: 'raw-loader' }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({template: './src/index.html'})
  ]
};

module.exports = config;
```



## 模式

通过选择 `development` 或 `production` 之中的一个，来设置 `mode` 参数，你可以启用相应模式下的 webpack 内置的优化

```javascript
module.exports = {
  mode: 'production'
};
```

