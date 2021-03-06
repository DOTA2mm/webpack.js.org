---
title: 创建 Library
contributors:
    - pksjce
    - johnstew
    - simon04
---
webpack 是一个用来打包应用程序(application)和 library 的代码的工具。如果你是一个 JavaScript library 的作者，并且想要流水线化(streamline)你的打包逻辑，那么这篇文档将会帮助到你。

## 创建一个 library

假设你正在写一个名为 `webpack-numbers` 的 library，可以将数字 1 到 5 转换为文本表示，或者反之可以将文本表示数字。它使用 ES6 模块实现，看起来像这样：

__src/index.js__
```javascript
import _ from 'lodash';
import numRef from './ref.json';

export function numToWord(num) {
    return _.reduce(numRef, (accum, ref) => {
        return ref.num === num ? ref.word : accum;
    }, '');
};

export function wordToNum(word) {
    return _.reduce(numRef, (accum, ref) => {
        return ref.word === word && word.toLowerCase() ? ref.num : accum;
    }, -1);
};
```
该 library 的使用方式如下：

```javascript
import * as webpackNumbers from 'webpack-numbers';
...
webpackNumbers.wordToNum('Two') // 输出 2
...

// 使用 CommonJS 模块引入

var webpackNumbers = require('webpack-numbers');
...
webpackNumbers.numToWord(3); // 输出 Three
...
```

```html
// 或者使用 script 标签引入

<html>
...
<script src="https://unpkg.com/webpack-numbers"></script>
<script>
    ...
    /* webpackNumbers 是一个全局变量 */
    webpackNumbers.wordToNum('Five') //输出 5
    ...
</script>
</html>
```
完整的 library 配置和相关代码请参阅 [webpack library 示例](https://github.com/kalcifer/webpack-library-example)。

## 配置 webpack

现在需要打包这个 library，同时要完成以下要求：
  - 不要打包 lodash，而是 require 用户加载好的 lodash。
  - library 的名字是 `webpack-numbers`，其变量名是 `webpackNumbers`。
  - library 可以用两种方式来引入：`import webpackNumbers from 'webpack-numbers'` 或者 `require('webpack-numbers')`。
  - 当 library 通过 `script` 标签引入的时候，可以通过全局变量 `webpackNumbers` 来使用。
  - library 可以在 Node.js 中使用。

### 增加 webpack

增加基本的 webpack 配置。

__webpack.config.js__

```javascript
var path = require('path');

module.exports = {
    entry: './src/index.js',
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: 'webpack-numbers.js'
    }
};

```

以上代码是打包该 library 的基本配置。

### 增加 `externals`

现在，如果执行 `webpack`，你会发现输出了一个非常巨大的文件。进一步观察该文件，你会发现 lodash 和你的代码被一起打包了。然而对于你的 library 本身来说，并不需要打包 `lodash`。因此你可能会想将该外部扩展(external)的控制权交给你的用户。

这一点可以通过配置 `externals` 来实现：

__webpack.config.js__

```javascript
module.exports = {
    ...
    externals: {
        "lodash": {
            commonjs: "lodash",
            commonjs2: "lodash",
            amd: "lodash",
            root: "_"
        }
    }
    ...
};
```

这意味着你的 library 需要一个名为 `lodash` 的依赖，这个依赖在用户的环境中必须可用。

### 增加 `libraryTarget`

为了让此 library 能够被广泛使用，你需要让它兼容不同的环境，例如 CommonJS，AMD，Node.js 或者作为一个全局变量。

为了让你的代码能够被重用，需要在 webpack 配置中增加一个 `library` 属性。

__webpack.config.js__

```javascript
module.exports = {
    ...
    output: {
        ...
        library: 'webpackNumbers'
    }
    ...
};
```

这能让你的 library 被引入后，可以通过全局变量来使用。

为了让 library 可以兼容其他环境，还需要在配置中增加 `libraryTarget` 属性。

__webpack.config.js__

```javascript
module.exports = {
    ...
    output: {
        ...
        library: 'webpackNumbers',
        libraryTarget: 'umd' // 其他可取值 - amd, commonjs, commonjs2, commonjs-module, this, var
    }
    ...
};
```

如果设置了 `library` 但没设置 `libraryTarget`，则`libraryTarget` 默认为 `var`，详见[配置文档](/configuration/output)。

### 最终步骤

[使用 webpack 打包你的生产代码](/guides/production-build)。

在 `package.json` 中指定主文件(main file)为你生成的文件路径。

__package.json__

```javascript
{
    ...
    "main": "dist/webpack-numbers.js",
    "module": "src/index.js", // 增加标准的模块，参照：https://github.com/dherman/defense-of-dot-js/blob/master/proposal.md#typical-usage
    ...
}
```

__Important Note:__ `module` will point to a module that has ES2015 module syntax but otherwise only syntax features that browser/node supports.

Now you can [publish it as an npm package](https://docs.npmjs.com/getting-started/publishing-npm-packages) and find it at [unpkg.com](https://unpkg.com/#/) to distribute it to your users.
__重要提示：__ `module` 将指向一个含有 ES2015 模块语法的模块，但是只有在支持此语法功能的浏览器或 Node.js 版本中才可用。

现在你可以[将其作为一个 npm 包来发布](https://docs.npmjs.com/getting-started/publishing-npm-packages)，并且在 [unpkg.com](https://unpkg.com/#/) 找到它并分发给你的用户。

***

> 原文：https://webpack.js.org/guides/author-libraries/
