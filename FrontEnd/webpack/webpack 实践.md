# Webpack 实践

## 最简单的方法：

webpack4.x 之后的工具都是开箱即用的，因此即使没有写配置文件，系统还是会按照默认参数进行打包。

>  默认的入口为.src/

> 默认的出口为dist/main.js

使用webpack的命令为:

```shell
npx webpack
```

上面的命令默认模式为 production，如果想要在开发环境使用，则可以使用

```shell
npx webpack --mode=development
# 什么都不加默认为 npx webpack --mode=production
```

也可以把 mode 写在 webpack.config.js里面，这样就只需要使用`npx webpack`就可以了, 假如现在有 ./src/index.js, 内容为

```javascript
class Animal {
    constructor(name) {
        this.name = name;
    }
    getName() {
        return this.name;
    }
}

const dog = new Animal('dog');
```

使用上面--mode=development 的命令，导出的对应代码为:

```javascript
eval("class Animal {\n    constructor(name) {\n        this.name = name;\n    }\n    getName() {\n        return this.name;\n    }\n}\n\nconst dog = new Animal('dog');\n\n\n//# sourceURL=webpack:///./src/index.js?");
```



## 加入 loader

假如我们希望 js 的版本被编译到低版本，那么我们必然需要 babel，在 webpack 中，需要引入 babel-loader, 因此我们需要编写 webpack.config.js, 在其中，对于 .js结尾的文件，使用 babel-loader, 并且不会对 node_modules 中的文件进行处理。

> test: 表示对哪类文件进行处理
>
> use: 表示要使用的loader
>
> exclude：表示不会处理哪些文件
>
> include: 表示哪些文件会去处理

```javascript
//webpack.config.js
module.exports = {
  	mode: "development",	
    module: {
        rules: [
            {
                test: /\.jsx?$/,
                use: ['babel-loader'],
                exclude: /node_modules/ //排除 node_modules 目录
            }
        ]
    }
}

```

有的时候还需要对于 babel 进行配置，可以在`.babelrc`中进行配置，也可以在 `webpack.config.js` 中进行配置, 如果在 webpack 中配置，则如下:

```javascript
module.exports = {
    // mode: 'development',
    module: {
        rules: [
            {
                test: /\.jsx?$/,
                use: {
                    loader: 'babel-loader',
                    options: { // options 表示对于这个 loader 进行的配置
                        presets: ["@babel/preset-env"],
                        plugins: [
                            [
                                "@babel/plugin-transform-runtime",
                                {
                                    "corejs": 3
                                }
                            ]
                        ]
                    }
                },
                exclude: /node_modules/
            }
        ]
    }
}

```

之后使用 `npx webpack`就会按照 development 并且配置的 loader 进行编译了，可以看到已经被编译成了低版本的代码。

参考:

1. [带你深度解锁Webpack系列(基础篇)](https://juejin.im/post/6844904079219490830)