---
title: 代码拆分 - 使用 require.ensure
sort: 33
contributors:
  - pksjce
  - rahulcs
  - johnstew
---

在这一节，我们会讨论 webpack 如何使用 `require.ensure()` 进行代码拆分。

W> `require.ensure` 是 webpack 特有的, 查看 [`import()`](/guides/code-splitting-import) 了解关于此的一个 ECMAScript 提案。

## `require.ensure()`

webpack 在构建时，会静态地解析代码中的 `require.ensure()`，在其中 callback 中的任何代码，包括被 `require()` 的代码，将被分离到一个新的 chunk 当中。这个新的 chunk 会被生成为异步的 bundle，由 webpack 通过 `jsonp` 来按需加载。

语法如下：

```javascript
require.ensure(dependencies: String[], callback: function(require), chunkName: String)
```

#### 依赖(dependencies)
这是一个字符串数组，通过这个参数，在所有的回调函数的代码被执行前，我们可以将所有需要用到的模块进行声明。

#### 回调函数(callback)
当所有的依赖都加载完成后，webpack 会执行这个回调函数。实际上，回调函数将 `require` 函数作为一个参数传递。因此，我们可以在回调函数体(function body)内进一步 `require()` 在执行时所需要的那些模块。

#### chunkName
`chunkName` 是用来提供给特定的 `require.ensure()` 来作为创建的 chunk 的名称。通过向不同 `require.ensure()` 的调用提供相同的 `chunkName`，我们可以将代码合并到相同的 chunk 中，做到只产生一个 bundle 来让浏览器加载。

## 示例

让我们考虑下面的文件结构：

```bash
.
├── dist
├── js
│   ├── a.js
│   ├── b.js
│   └── entry.js
└── webpack.config.js
```

**entry.js**

```javascript
require('./a');
require.ensure([], function(require){
    require('./b');
});
```

**a.js**

```javascript
console.log('***** I AM a *****');
```

**b.js**

```javascript
console.log('***** I AM b *****');
```

**webpack.config.js**

```javascript
var path = require('path');

module.exports = function(env) {
    return {
        entry: './js/entry.js',
        output: {
            filename: 'bundle.js',
            path: path.resolve(__dirname, 'dist')
        }
    }
}
```
通过执行这个项目的 webpack 构建，我们发现 webpack 创建了 2 个新的 bundle，`bundle.js` 和 `0.bundle.js`。

`entry.js` 和 `a.js` 被打包进 `bundle.js`。

`b.js` 被打包进 `0.bundle.js`。

W> `require.ensure` 内部依赖于 `Promises`。 如果你在旧的浏览器中使用 `require.ensure` 请记得去 shim `Promise` [es6-promise polyfill](https://github.com/stefanpenner/es6-promise)。

**更多示例**
* https://github.com/webpack/webpack/tree/master/examples/code-splitting
* https://github.com/webpack/webpack/tree/master/examples/named-chunks – illustrates the use of `chunkName`

## `require.ensure()` 的陷阱

### 空数组作为参数

```javascript
require.ensure([], function(require){
    require('./a.js');
});
```

上面代码保证了拆分点被创建，而且 `a.js` 被 webpack 单独打包。

### 依赖作为参数

```javascript
require.ensure(['./a.js'], function(require) {
    require('./b.js');
});
```

上面代码中，`a.js` 和 `b.js` 都被打包到一起，而且从主 bundle 中拆分出来。但只有 `b.js` 的内容被执行。`a.js` 的内容仅仅是可被使用，但并没有被执行。想要执行 `a.js`，我们必须以同步的方式引用它，如 `require('./a.js')`，来让它的代码被执行。

***

> 原文：https://webpack.js.org/guides/code-splitting-require/
