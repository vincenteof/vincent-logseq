- 同步代码，异步 chunk，以及 module federation 相关
-
- webpackJsonpCallback
- ```javascript
  // install a JSONP callback for chunk loading
  var webpackJsonpCallback = (parentChunkLoadingFunction, data) => {
    var [chunkIds, moreModules, runtime] = data
    // add "moreModules" to the modules object,
    // then flag all "chunkIds" as loaded and fire callback
    var moduleId,
      chunkId,
      i = 0
    if (chunkIds.some((id) => installedChunks[id] !== 0)) {
      for (moduleId in moreModules) {
        if (__webpack_require__.o(moreModules, moduleId)) {
          __webpack_require__.m[moduleId] = moreModules[moduleId]
        }
      }
      if (runtime) var result = runtime(__webpack_require__)
    }
    if (parentChunkLoadingFunction) parentChunkLoadingFunction(data)
    for (; i < chunkIds.length; i++) {
      chunkId = chunkIds[i]
      if (
        __webpack_require__.o(installedChunks, chunkId) &&
        installedChunks[chunkId]
      ) {
        installedChunks[chunkId][0]()
      }
      installedChunks[chunkId] = 0
    }
    return __webpack_require__.O(result)
  }
  ```
- 这个函数在我们 demo 的入口里是这样被调用的
- ```javascript
  self['webpackJsonp_webpack-runtime-analysis'] =
    self['webpackJsonp_webpack-runtime-analysis'] ||
    [].push([
      ['basic'],
      {
        /* more modules */
      },
      (__webpack_require__) => {
        // webpackRuntimeModules
        var __webpack_exec__ = (moduleId) =>
          __webpack_require__((__webpack_require__.s = moduleId))
        var __webpack_exports__ = __webpack_exec__('./src/basic/index.js')
      },
    ])
  ```
- `parentChunkLoadingFunction` 是直接在劫持的时候被 bind 的，我们传入的数组对象对应了 data。
- `installedChunks` 里记录了每个 chunk 的加载状态，为 0 说明加载已完成。
- `__webpack_require__.o` 就是 hasOwnProperty，`__webpack_require__.m` 纪录了所有的 modules  的定义，它等同于 `__webpack_modules__`，提供给 `__webpack_require__` 去根据 id 找对应的定义。10-14 行代码会将 moreModules 里的 module 定义直接加入到 `__webpack_modules__`。
- 这边的 moreModules 类似于
- ```javascript
  {
    './src/basic/a.js': (
      __unused_webpack_module,
      __webpack_exports__,
      __webpack_require__
    ) => {
      __webpack_require__.r(__webpack_exports__)
      __webpack_require__.d(__webpack_exports__, {
        default: () => helloWord,
      })
      function helloWord() {
        console.log('hello word')
      }
    },
  
    './src/basic/index.js': (
      __unused_webpack_module,
      __webpack_exports__,
      __webpack_require__
    ) => {
      __webpack_require__.r(__webpack_exports__)
      var _a__WEBPACK_IMPORTED_MODULE_0__ =
        __webpack_require__('./src/basic/a.js')
  
      ;(0, _a__WEBPACK_IMPORTED_MODULE_0__['default'])()
      __webpack_require__
        .e('src_basic_b_js')
        .then(__webpack_require__.bind(__webpack_require__, './src/basic/b.js'))
        .then((x) => {
          console.log(x.default())
        })
    },
  }
  
  ```
- 这个 case 里提供的 runtime 参数就是把整个流程跑起来，直接对入口文件做了一次 `__webpack_require__`。猜测这个 `__webpack_require__.s` 标记了整个依赖图的入口。
- 然后就开始了递归 require 的过程，先是同步的 module，从 index 到 a 都是同步代码，b 文件因为是动态引入的，被单独分了一个 chunk，并且通过 `__webpack_require__.e` 去异步加载。
- ```javascript
  /* webpack/runtime/ensure chunk */
  (() => {
    __webpack_require__.f = {}
    // This file contains only the entry chunk.
    // The chunk loading function for additional chunks
    __webpack_require__.e = (chunkId) => {
      return Promise.all(
        Object.keys(__webpack_require__.f).reduce((promises, key) => {
          __webpack_require__.f[key](chunkId, promises)
          return promises
        }, [])
      )
    }
  })()
  ```
- `__webpack_require__.e` 具体的执行流程依赖于 `__webpack_require__.f`。这样设计可能是为了拓展性，只要把需要的操作函数放在 `__webpack_require__.f`，`__webpack_require__.e` 自动会将他们 pipe 起来。
- 目前这个 case 里只生成了一个基本的用于异步 chunk 加载的函数
- ```javascript
  __webpack_require__.f.j = (chunkId, promises) => {
    // JSONP chunk loading for javascript
    var installedChunkData = __webpack_require__.o(installedChunks, chunkId)
      ? installedChunks[chunkId]
      : undefined
    if (installedChunkData !== 0) {
      // 0 means "already installed".
  
      // a Promise means "currently loading".
      if (installedChunkData) {
        promises.push(installedChunkData[2])
      } else {
        if ('runtime' != chunkId) {
          // setup Promise in chunk cache
          var promise = new Promise(
            (resolve, reject) =>
              (installedChunkData = installedChunks[chunkId] = [resolve, reject])
          )
          promises.push((installedChunkData[2] = promise))
  
          // start chunk loading
          var url = __webpack_require__.p + __webpack_require__.u(chunkId)
          // create error before stack unwound to get useful stacktrace later
          var error = new Error()
          var loadingEnded = (event) => {
            if (__webpack_require__.o(installedChunks, chunkId)) {
              installedChunkData = installedChunks[chunkId]
              if (installedChunkData !== 0) installedChunks[chunkId] = undefined
              if (installedChunkData) {
                var errorType =
                  event && (event.type === 'load' ? 'missing' : event.type)
                var realSrc = event && event.target && event.target.src
                error.message =
                  'Loading chunk ' +
                  chunkId +
                  ' failed.\n(' +
                  errorType +
                  ': ' +
                  realSrc +
                  ')'
                error.name = 'ChunkLoadError'
                error.type = errorType
                error.request = realSrc
                installedChunkData[1](error)
              }
            }
          }
          __webpack_require__.l(url, loadingEnded, 'chunk-' + chunkId, chunkId)
        } else installedChunks[chunkId] = 0
      }
    }
  }
  ```
- `__webpack_require__.p` 应该是 publicPath，`__webpack_require__.u` 则用于根据 chunkId 得到对应的文件名。
- `__webpack_require__.l` 就是实际上通过动态插入 script 标签去完成异步 chunk 的加载。
- ```javascript
  ;/* webpack/runtime/load script */
  (() => {
    var inProgress = {}
    var dataWebpackPrefix = 'webpack-runtime-analysis:'
    // loadScript function to load a script via script tag
    __webpack_require__.l = (url, done, key, chunkId) => {
      if (inProgress[url]) {
        inProgress[url].push(done)
        return
      }
      var script, needAttach
      if (key !== undefined) {
        var scripts = document.getElementsByTagName('script')
        for (var i = 0; i < scripts.length; i++) {
          var s = scripts[i]
          if (
            s.getAttribute('src') == url ||
            s.getAttribute('data-webpack') == dataWebpackPrefix + key
          ) {
            script = s
            break
          }
        }
      }
      if (!script) {
        needAttach = true
        script = document.createElement('script')
  
        script.charset = 'utf-8'
        script.timeout = 120
        if (__webpack_require__.nc) {
          script.setAttribute('nonce', __webpack_require__.nc)
        }
        script.setAttribute('data-webpack', dataWebpackPrefix + key)
        script.src = url
      }
      inProgress[url] = [done]
      var onScriptComplete = (prev, event) => {
        // avoid mem leaks in IE.
        script.onerror = script.onload = null
        clearTimeout(timeout)
        var doneFns = inProgress[url]
        delete inProgress[url]
        script.parentNode && script.parentNode.removeChild(script)
        doneFns && doneFns.forEach((fn) => fn(event))
        if (prev) return prev(event)
      }
      var timeout = setTimeout(
        onScriptComplete.bind(null, undefined, {
          type: 'timeout',
          target: script,
        }),
        120000
      )
      script.onerror = onScriptComplete.bind(null, script.onerror)
      script.onload = onScriptComplete.bind(null, script.onload)
      needAttach && document.head.appendChild(script)
    }
  })()
  ```
- 这个函数找这个异步 chunk 是否在处理中，如果已经在处理中，只会把当前这次异步 import 的 callback 放到 inProgress 里。如果是首次那就创建 script 标签并初始化对应的 inProgress。这个 script 加载完成以后会被 remove 掉。
- 这里的 done 是由上面的 `__webpack_require__.f.j` 传入的。当加载发生错误的时候才会被实际调用。
- 需要区分 moduleId 与 chunkId，moduleId 是指每一个被 require 的 module 的 id。chunkId 是指每一个被 `self['webpackJsonp_webpack-runtime-analysis'] || []).push` 加载的 chunk 的 id，一般是指异步 chunk。`installedChunkData` 保存的是这些异步 chunk 的加载状态。
- 整个异步 chunk 的加载链路是这样的，异步的 chunk 会通过 `__webpack_require__.e` 去加载的，这个函数会设置 `installedChunkData` 的初识态，异步 chunk 的代码格式是 `self['webpackJsonp_webpack-runtime-analysis'] || []).push`，因此加载完成执行的时候就会触发 `webpackJsonpCallback` 的逻辑，这里回去处理成功的终态，如果不能正确加载执行的话则会进入 script 标签的回掉，处理失败的终态。
- 这个基本 case 里调用链路是这样的：
	- webpackJsonpCallback
		- __webpack_require__ (a.js)
		- __webpack_require__.e (b.js)
			- webpackJsonpCallback
			- __webpack_require__
- 上面的 `__webpack_require__.e ` 只是去异步加载 chunk，整个流程最终只是把对应的 module 定义放到 `__webpack_modules__` 里。那么它是何时被解析并被放到 module 缓存里的呢？观察上面主文件的代码
- ```javascript
  __webpack_require__
    .e('src_basic_b_js')
    .then(__webpack_require__.bind(__webpack_require__, './src/basic/b.js'))
    .then((x) => {
      console.log(x.default())
    })
  ```
- 在 e 的回调里执行了 require 的 bind 语句，这里 bind 就是异步 chunk 的 moduleId，也就是说在异步加载完成后马上做一次 require，后续 then 的参数对应的导出。此刻缓存也就完成了，后续再去引入这个异步 chunk 都会走缓存了。
-
-
-
-
-
-
- 上面的 case 是一个基本场景，我们可以看下加入 mf 已经会是什么情况。观察打包出来的主文件，和基本场景非常类似，但是 moreModules 里多了一个 mf 模块的异步引入
- ```javascript
  {
    'webpack/container/reference/mfRemote': (
      module,
      __unused_webpack_exports,
      __webpack_require__
    ) => {
      'use strict'
      var __webpack_error__ = new Error()
      module.exports = new Promise((resolve, reject) => {
        if (typeof mfRemote !== 'undefined') return resolve()
        __webpack_require__.l(
          'http://localhost:3001/remoteEntry.js',
          (event) => {
            if (typeof mfRemote !== 'undefined') return resolve()
            var errorType =
              event && (event.type === 'load' ? 'missing' : event.type)
            var realSrc = event && event.target && event.target.src
            __webpack_error__.message =
              'Loading script failed.\n(' + errorType + ': ' + realSrc + ')'
            __webpack_error__.name = 'ScriptExternalLoadError'
            __webpack_error__.type = errorType
            __webpack_error__.request = realSrc
            reject(__webpack_error__)
          },
          'mfRemote'
        )
      }).then(() => mfRemote)
    },
  }
  ```
- 这里用到的 `__webpack_require__.l` 和上面异步加载 chunk 用到的 方法是一样的，在这种打包模式下成功以后全局会有一个 mfRemote 的变量，就是远程模块的导出内容。
- 因此需要看下远程应用的 remoteEntry 里到底有什么东西。因为 remote 貌似不支持 runtime chunk 分离，所以 remoteEntry 里有很多 runtime 的函数，除此以外就是导出的远程 module。类似下面这种
- ```javascript
  ;(() => {
    var exports = __webpack_exports__
    var moduleMap = {
      './sayHi': () => {
        return __webpack_require__
          .e('src_mfRemote_sayHi_js')
          .then(() => () => __webpack_require__('./src/mfRemote/sayHi.js'))
      },
    }
    var get = (module, getScope) => {
      __webpack_require__.R = getScope
      getScope = __webpack_require__.o(moduleMap, module)
        ? moduleMap[module]()
        : Promise.resolve().then(() => {
            throw new Error(
              'Module "' + module + '" does not exist in container.'
            )
          })
      __webpack_require__.R = undefined
      return getScope
    }
    var init = (shareScope, initScope) => {
      if (!__webpack_require__.S) return
      var name = 'default'
      var oldScope = __webpack_require__.S[name]
      if (oldScope && oldScope !== shareScope)
        throw new Error(
          'Container initialization failed as it has already been initialized with a different share scope'
        )
      __webpack_require__.S[name] = shareScope
      return __webpack_require__.I(name, initScope)
    }
  
    __webpack_require__.d(exports, {
      get: () => get,
      init: () => init,
    })
  })()
  
  mfRemote = __webpack_exports__
  ```
- 可以看到当这个 chunk 被加载执行完成以后，mfRemote 这个全局变量已被赋值为具体的导出。
- `__webpack_require__.t`
- 在基本 case 里说到，通过给 `__webpack_require__.f` 里添加不同的 key，可以达到 pipe 的功能，在 mf 的场景下就要派用场了，这里多了一个 `__webpack_require__.f.remotes` 。
-
-