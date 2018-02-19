# 打包文件详解

理解 webpack 最好的方式是**观察实验**。通过调整不同的配置参数，观察生成的打包文件如何变化。这样就能切实掌握不同参数、不同插件的真实功用。

假如有两个文件 `app.js` 和 `bar.js`，内容如下：

```js
/** app.js */
import bar from './bar'
bar()

/** bar.js */
export default function bar() {
  console.log('bar running...')
}
```

配套的 `webpack.conf.js` 如下：

```js
/** webpack.config.js */
const path = require('path')

module.exports = {
  entry: {
    main: './app.js'
  },
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'bundle.js'
  }
}
```

经 webpack 打包后，会生成如下文件（删除前导注释）：

```js
/** bundle.js */
(function(modules) { // webpackBootstrap
	// The module cache
	var installedModules = {};
	// The require function
	function __webpack_require__(moduleId) {
		// Check if module is in cache
		if(installedModules[moduleId]) {
			return installedModules[moduleId].exports;
		}
		// Create a new module (and put it into the cache)
		var module = installedModules[moduleId] = {
			i: moduleId,
			l: false,
			exports: {}
		};
		// Execute the module function
		modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);
		// Flag the module as loaded
		module.l = true;
		// Return the exports of the module
		return module.exports;
	}
	// expose the modules object (__webpack_modules__)
	__webpack_require__.m = modules;
	// expose the module cache
	__webpack_require__.c = installedModules;
	// define getter function for harmony exports
	__webpack_require__.d = function(exports, name, getter) {
		if(!__webpack_require__.o(exports, name)) {
			Object.defineProperty(exports, name, {
				configurable: false,
				enumerable: true,
				get: getter
			});
		}
	};
	// getDefaultExport function for compatibility with non-harmony modules
	__webpack_require__.n = function(module) {
		var getter = module && module.__esModule ?
			function getDefault() { return module['default']; } :
			function getModuleExports() { return module; };
		__webpack_require__.d(getter, 'a', getter);
		return getter;
	};
	// Object.prototype.hasOwnProperty.call
	__webpack_require__.o = function(object, property) { return Object.prototype.hasOwnProperty.call(object, property); };
	// __webpack_public_path__
	__webpack_require__.p = "";
	// Load entry module and return exports
	return __webpack_require__(__webpack_require__.s = 0);
})
/************************************************************************/
([
/* 0 */
(function(module, __webpack_exports__, __webpack_require__) {
"use strict";
Object.defineProperty(__webpack_exports__, "__esModule", { value: true });
var __WEBPACK_IMPORTED_MODULE_0__bar__ = __webpack_require__(1);

Object(__WEBPACK_IMPORTED_MODULE_0__bar__["a" /* default */])()
}),
/* 1 */
(function(module, __webpack_exports__, __webpack_require__) {
"use strict";
/* harmony export (immutable) */ __webpack_exports__["a"] = bar;
function bar() {
  console.log('bar running...')
}
})
]);
```

该 bundle 文件定义了一个立即执行匿名函数，形参是 `modules`，实参是一个函数数组，元素分别为处理后的 `app.js` 和 `bar.js`。

函数数组的每个函数，形参有三个，分别是 `module`, `__webpack_exports__` 和 `__webpack_require__` 。从名称可以推断，`__webpack_exports__` 用于导出模块，`__webpack_require__` 用于引入模块。

## `__webpack_require__()`

下面来观察 `__webpack_require__` 是如何定义的。

```js
// 模块缓存区
var installedModules = {}

function __webpack_require__(moduleId) {
  // 检查模块是否在缓存中
  if(installedModules[moduleId]) {
    return installedModules[moduleId].exports
  }

  // 创建新模块，并放入缓存中（此时模块尚未加载）
  var module = installedModules[moduleId] = {
    i: moduleId,
    l: false,
    exports: {}
  }

  // 真正执行模块的函数
  modules[moduleId].call(module.exports, module, module.exports, __webpack_require__)

  // 将模块标定为已加载
  module.l = true

  // 返回模块的输出值
  return module.exports
}
```

定义完 `__webpack_require__` 主要功能后，接下来定义几个变量：

```js
// 暴露模块数组 (__webpack_modules__)
__webpack_require__.m = modules

// 暴露模块缓存
__webpack_require__.c = installedModules

// 为 harmony 输出定义 getter 函数
__webpack_require__.d = function(exports, name, getter) {
  if(!__webpack_require__.o(exports, name)) {
    Object.defineProperty(exports, name, {
      configurable: false,
      enumerable: true,
      get: getter
    })
  }
}

// getDefaultExport function for compatibility with non-harmony modules
__webpack_require__.n = function(module) {
  var getter = module && module.__esModule ?
    function getDefault() { return module['default'] } :
    function getModuleExports() { return module };
  __webpack_require__.d(getter, 'a', getter)
  return getter
}

// Object.prototype.hasOwnProperty.call
__webpack_require__.o = function(object, property) { 
  return Object.prototype.hasOwnProperty.call(object, property)
}

// __webpack_public_path__
__webpack_require__.p = ""
```

最后，引入第一个模块，并执行：

```js
// 加载入口模块，并返回其输出值
return __webpack_require__(__webpack_require__.s = 0)
```

### 总结

对于简单的单输入单输出配置，webpack 会将各代码分别封装为函数，并把原始的 ES6 模块语句转译为 `__webpack_require__` 和 `__webpack_exports__` 。

封装为函数，就可以保证各模块的变量都在自己的作用域，不会污染全局作用域。

## 多入口配置

如果增加一个新的模块 `foo.js`，内容如下：

```js
/** foo.js */
export default 42
```

同时修改 `app.js` 和 `bar.js`，各自引入 `foo.js`：

```js
/** app.js */
import bar from './bar';
import foo from './foo'

bar()
console.log(foo)

/** bar.js */
import foo from './foo'

export default function bar() {
  console.log('bar running...')
  console.log(foo)
}
```

并且将配置文件修改为多输入多输出模式：

```js
/** webpack.config.js */
const path = require('path')

module.exports = {
  entry: {
    main: './app.js',
    vendor: './foo.js'
  },
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].js'
  }
}
```

此时，运行 webpack ，会得到两个输出文件 `main.js` 和 `vendor.js`。

先看看 webpack 构建的输出内容：

```sh
Hash: 26b478e45d80d75c14b4
Version: webpack 3.10.0
Time: 70ms
    Asset     Size  Chunks             Chunk Names
  main.js  3.51 kB    0, 1  [emitted]  main
vendor.js  2.66 kB       1  [emitted]  vendor
   [0] ./foo.js 17 bytes {0} {1} [built]
   [1] ./app.js 72 bytes {0} [built]
   [2] ./bar.js 114 bytes {0} [built]
```

再看看 `main.js` 内容：

```js
(function(modules) { // webpackBootstrap
  // ...
  return __webpack_require__(__webpack_require__.s = 1);
})
/************************************************************************/
 ([
/* 0 */
(function(module, __webpack_exports__, __webpack_require__) {
  "use strict";
  Object.defineProperty(__webpack_exports__, "__esModule", { value: true });
  __webpack_exports__["default"] = (42);
}),

/* 1 */
(function(module, __webpack_exports__, __webpack_require__) {
  "use strict";
  Object.defineProperty(__webpack_exports__, "__esModule", { value: true });
  var __WEBPACK_IMPORTED_MODULE_0__bar__ = __webpack_require__(2);
  var __WEBPACK_IMPORTED_MODULE_1__foo__ = __webpack_require__(0);
  Object(__WEBPACK_IMPORTED_MODULE_0__bar__["a" /* default */])()
  console.log(__WEBPACK_IMPORTED_MODULE_1__foo__["default"])
}),

/* 2 */
(function(module, __webpack_exports__, __webpack_require__) {
  "use strict";
  __webpack_exports__["a"] = bar;
  var __WEBPACK_IMPORTED_MODULE_0__foo__ = __webpack_require__(0);
  function bar() {
    console.log('bar running...')
    console.log(__WEBPACK_IMPORTED_MODULE_0__foo__["default"])
  }
})
 ]);
```

可见，`main.js` 包含了 `foo.js`, `main.js` 和 `bar.js` 三个模块的内容。

接着看看 `vendor.js` 的内容：

```js
(function(modules) { // webpackBootstrap
	// ...
	// Load entry module and return exports
	return __webpack_require__(__webpack_require__.s = 0);
})
/************************************************************************/
([
/* 0 */
(function(module, __webpack_exports__, __webpack_require__) {
  "use strict";
  Object.defineProperty(__webpack_exports__, "__esModule", { value: true });
  __webpack_exports__["default"] = (42);
})
]);
```

可以看到 `vendor.js` 包含 `foo.js` 模块内容。

小结一下，虽然实现了多个入口打包，但是打包生成的 bundle 有重复代码（`foo.js` 的内容被两次打包），这造成了资源浪费。

## 提取公共资源

使用插件 **CommonsChunkPlugin** 可以把重复代码提取出来，当作一个功用模块。

修改 `webpack.config.js` ，在其中增加 `plugins` 字段，具体内容如下：

```js
// ...
plugins: [
  new webpack.optimize.CommonsChunkPlugin({
    name: 'commons',
    filename: 'commons.js'
  })
]
// ...
```

执行 webpack ，在命令行窗口输出如下内容：

```sh
Hash: c46701fafad8b4ac9960
Version: webpack 3.11.0
Time: 91ms
     Asset       Size  Chunks             Chunk Names
   main.js  881 bytes       0  [emitted]  main
 vendor.js   25 bytes       1  [emitted]  vendor
commons.js    4.09 kB       2  [emitted]  commons
   [0] ./foo.js 17 bytes {2} [built]
   [1] ./app.js 72 bytes {0} [built]
   [2] ./bar.js 114 bytes {0} [built]
```

我们先研究一下 `commons.js` 代码：

```js
(function(modules) { // webpackBootstrap
	// install a JSONP callback for chunk loading
	var parentJsonpFunction = window["webpackJsonp"];
	window["webpackJsonp"] = function webpackJsonpCallback(chunkIds, moreModules, executeModules) {
		// add "moreModules" to the modules object,
		// then flag all "chunkIds" as loaded and fire callback
		var moduleId, chunkId, i = 0, resolves = [], result;
		for(;i < chunkIds.length; i++) {
			chunkId = chunkIds[i];
			if(installedChunks[chunkId]) {
				resolves.push(installedChunks[chunkId][0]);
			}
			installedChunks[chunkId] = 0;
		}
		for(moduleId in moreModules) {
			if(Object.prototype.hasOwnProperty.call(moreModules, moduleId)) {
				modules[moduleId] = moreModules[moduleId];
			}
		}
		if(parentJsonpFunction) parentJsonpFunction(chunkIds, moreModules, executeModules);
		while(resolves.length) {
			resolves.shift()();
		}
		if(executeModules) {
			for(i=0; i < executeModules.length; i++) {
				result = __webpack_require__(__webpack_require__.s = executeModules[i]);
			}
		}
		return result;
	}
  
	// The module cache
	var installedModules = {}
  
	// objects to store loaded and loading chunks
	var installedChunks = {
		2: 0
	}
  
	// The require function
	function __webpack_require__(moduleId) {
		// Check if module is in cache
		if(installedModules[moduleId]) {
			return installedModules[moduleId].exports;
		}
		// Create a new module (and put it into the cache)
		var module = installedModules[moduleId] = {
			i: moduleId,
			l: false,
			exports: {}
		};

		// Execute the module function
		modules[moduleId].call(module.exports, module, module.exports, __webpack_require__)

		// Flag the module as loaded
		module.l = true

		// Return the exports of the module
		return module.exports
	}

	// expose the modules object (__webpack_modules__)
	__webpack_require__.m = modules

	// expose the module cache
	__webpack_require__.c = installedModules

	// define getter function for harmony exports
	__webpack_require__.d = function(exports, name, getter) {
		if(!__webpack_require__.o(exports, name)) {
			Object.defineProperty(exports, name, {
				configurable: false,
				enumerable: true,
				get: getter
			});
		}
	};

	// getDefaultExport function for compatibility with non-harmony modules
	__webpack_require__.n = function(module) {
		var getter = module && module.__esModule ?
			function getDefault() { return module['default']; } :
			function getModuleExports() { return module; };
		__webpack_require__.d(getter, 'a', getter);
		return getter;
	}
  
	// Object.prototype.hasOwnProperty.call
	__webpack_require__.o = function(object, property) { return Object.prototype.hasOwnProperty.call(object, property); }
  
	// __webpack_public_path__
	__webpack_require__.p = ""
  
	// on error function for async loading
	__webpack_require__.oe = function(err) { console.error(err); throw err; };
})
/************************************************************************/
([
/* 0 */
(function(module, __webpack_exports__, __webpack_require__) {
  "use strict";
  Object.defineProperty(__webpack_exports__, "__esModule", { value: true });
  __webpack_exports__["default"] = (42);
})
]);
```

`commons.js` 新增了一个函数 `webpackJsonpCallback`：

```js
// install a JSONP callback for chunk loading
var parentJsonpFunction = window["webpackJsonp"];
window["webpackJsonp"] = function webpackJsonpCallback(chunkIds, moreModules, executeModules) {
  // add "moreModules" to the modules object,
  // then flag all "chunkIds" as loaded and fire callback
  var moduleId, chunkId, i = 0, resolves = [], result;
  for(;i < chunkIds.length; i++) {
    chunkId = chunkIds[i]
    if(installedChunks[chunkId]) {
      resolves.push(installedChunks[chunkId][0])
    }
    installedChunks[chunkId] = 0
  }
  for(moduleId in moreModules) {
    if(Object.prototype.hasOwnProperty.call(moreModules, moduleId)) {
      modules[moduleId] = moreModules[moduleId];
    }
  }
  if(parentJsonpFunction) parentJsonpFunction(chunkIds, moreModules, executeModules);
  while(resolves.length) {
    resolves.shift()();
  }
  if(executeModules) {
    for(i=0; i < executeModules.length; i++) {
      result = __webpack_require__(__webpack_require__.s = executeModules[i]);
    }
  }
  return result;
}

// objects to store loaded and loading chunks
var installedChunks = {
  2: 0
}
```

在 `main.js` 中有 `webpackJsonp` 的执行代码：

```js
webpackJsonp([0],[
  /* 0 */,

  /* 1 */
  (function(module, __webpack_exports__, __webpack_require__) {
    "use strict"
    Object.defineProperty(__webpack_exports__, "__esModule", { value: true })
    var __WEBPACK_IMPORTED_MODULE_0__bar__ = __webpack_require__(2)
    var __WEBPACK_IMPORTED_MODULE_1__foo__ = __webpack_require__(0)

    Object(__WEBPACK_IMPORTED_MODULE_0__bar__["a" /* default */])()
    console.log(__WEBPACK_IMPORTED_MODULE_1__foo__["default"])
  }),

  /* 2 */
  (function(module, __webpack_exports__, __webpack_require__) {
    "use strict";
    __webpack_exports__["a"] = bar;
    var __WEBPACK_IMPORTED_MODULE_0__foo__ = __webpack_require__(0);
    function bar() {
        console.log('bar running...')
        console.log(__WEBPACK_IMPORTED_MODULE_0__foo__["default"])
    }
  })
],[1]);
```

可以看到，`webpackJsonp` 有三个实参，类型均为数组，第一个数组是公共代码的索引值（`chunkIds`），第二个数组是具体定义的代码（`moreModules`），第三个数组是需要执行的 chunk 索引值（`executeModules`）。

`vendor.js` 中的代码更简单：

```js
webpackJsonp([1],[],[0])
```

小结一下，通过使用 `CommonsChunkPlugin`，webpack 将 webpack 运行时和公共代码提取到一个单独文件，并暴露出一个全局变量 `webpackJsonp` ，其他的函数只需调用 `webpackJsonp` 即可。如果各函数有自己的独特模块，也可以出入到 `moreModules` 中。

## REF

- [CommonsChunkPlugin][commons-chunk]

[commons-chunk]: https://webpack.js.org/plugins/commons-chunk-plugin/