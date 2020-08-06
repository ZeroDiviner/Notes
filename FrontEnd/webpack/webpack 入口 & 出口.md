# Webpack 入口和出口

## 单个入口写法

```javascript
const config = {
  entry: './path/to/my/entry/file.js'
};

module.exports = config;

// 或者

const config = {
  entry: {
    main: './path/to/my/entry/file.js'
  }
};
```

可以看到entry 可以接受一个字符串表示主文件入口或者接收一个对象。

当 entry 接收一个数组作为参数的时候，则表示将创建多个主入口



## 传入对象的语法

```javascript
const config = {
  entry: {
    app: './src/app.js',
    vendors: './src/vendors.js'
  }
};
// 这里表示分离应用程序(app)和第三方库(vendors)的入口
```

上面代码表示webpack 从 `app.js` 和 `vendors.js` 开始创建依赖图(dependency graph)。这些依赖图是彼此完全分离、互相独立的



## 多个页面应用程序

```javascript
const config = {
  entry: {
    pageOne: './src/pageOne/index.js',
    pageTwo: './src/pageTwo/index.js',
    pageThree: './src/pageThree/index.js'
  }
};
```

上面表示 webpack 需要**创建三个完全独立的依赖图**。



## 出口写法

output 属性的最低要求是一个对象，包含两个部分：

* filename: 表示输出文件的名字
* Path: 表示输出的位置

```javascript
const config = {
  output: {
    filename: 'bundle.js',
    path: '/home/proj/public/assets'
  }
};

module.exports = config;
```

这个配置会将一个单独的 `bundle.js`文件输出到`/home/proj/public/assets`下



## 多个入口写法

当有多个入口的时候，output 因为还是一个单独的对象，因此需要使用**占位符**

```javascript
{
  entry: {
    app: './src/app.js',
    search: './src/search.js'
  },
  output: {
    filename: '[name].js',
    path: __dirname + '/dist'
  }
}
```

