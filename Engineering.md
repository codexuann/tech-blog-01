# CommonJS 与 ES Module
#### 1. 区别
- CommonJS (CJS) 是 Node.js 的传统规范, 而ES Module (ESM) 是语言层面的官方标准
- 语法差异
	- CJS: 使用 `require()` 导入，`module.exports` 或 `exports` 导出
	- ESM: 使用 `import` 导入，`export` 或 `export default` 导出
- 加载方式
	- CJS: 运行时加载, `require()` 是一个普通的同步函数调用。
		- 文件存在一个全局对象, 初始化是空的 `{}` 
		- 当 require某个文件时, 就将这个文件执行. 如果遇到 `module.exports` ,就把导出的值存在全局对象中, 键为文件名. 例如 `{"a.js":{num:2},...}`
		- 再次require时, 如果全局对象中有这个值, 就直接返回, 没有再执行上述步骤
	- ESM: 编译时输出, JS 引擎在代码真正执行前的编译阶段，就会分析出所有的 `import` 依赖关系，构建出一棵模块依赖树
- 值导出
	- CJS: 导出基本数据类型, 在模块内部修改这个值, 外部引用不会改变. 除非导出对象
	- ESM: 输出的值是动态只读引用, 外部模块不能修改内部的值
#### 2. 摇树优化 `Tree-shaking`
- 前提是 ES Module 规范, 因为其是静态的（编译时决定的）, 打包工具在不运行代码的情况下，就能通过生成 AST（抽象语法树），清晰地分析出整个项目的依赖关系图。它能准确地知道模块导出了哪些东西，而哪些东西在其他文件里被 `import` 并使用了
- 机制
	- 标记: 打包工具遍历 AST，把用到的导出打上标记，没用到的就不管
	- 清除: 到了代码压缩阶段 (terser工具等) , 没有被标记的无用代码就会被彻底物理删除
- 失效情况
	- 文件中有代码的副作用: 在执行过程中修改了全局状态
		- 如 `window.myConfig`, `Array.prototype.myMethod`
		- 即便没有使用这些代码, 打包工具也不会摇掉他们
	- 解决
		- 如果我们确定整个 npm 包或整个项目的代码都是纯净的、没有副作用的，我们可以直接在 `package.json` 中配置 `"sideEffects": false`, 引导打包工具进行激进的 Tree Shaking
# webpack
#### 1. 定义
- Webpack本质是静态模块打包工具. 项目里所有的资源（JS、CSS、图片、字体等）都是模块。它的终极目标，就是把这些错综复杂的模块，翻译并合并成浏览器可以直接高效运行的静态资源。
#### 2. 核心配置
- `Entry` 和 `Output`
	- 规定了 Webpack 从哪个文件开始抓取依赖，以及最后打包好的成品放在哪里。
- `Loder`
	- 因为 Webpack 本身只懂 JS 和 JSON，Loader 扮演了翻译官的角色，比如把 Vue 模板、Sass、图片转换成 JS 模块
	- 在 `module.rules` 中配置，每个规则定义 `test`（匹配的文件类型）和 `use` (使用的 loader 列表)
	- 常见: es6翻译成低版本js: `bable-loader`, 处理css: `css-loader`, 处理scss: `scss-loader`, 解析vue: `vue-loader`, 处理图片(复制到输出目录) : `file-loader`
- `Plugin`
	- 负责一切更宏大、更复杂的任务
	- 在 `plugins` 中使用, 传入 `new` 的插件实例
	- 常见
		- `HtmlWebpackPlugin`: 自动生成一个 HTML 文件，并将打包后的 bundle 自动注入其中
		- `CleanWebpackPlugin`: 每次打包前清理输出目录，避免旧文件残留
- `devServer`: 
	- 配置 `hot:true` 开启热模块替换与热更新 (HMR)
		- 开发时, 代码更新后, 浏览器在不刷新页面的情况下，在内存里把旧组件替换成新组件
		- 开启 HMR 后，Webpack 会在浏览器和本地服务之间建立一个 WebSocket 长连接。当你修改了一个组件的代码并保存，Webpack 只重新编译这一个组件，然后把这个“补丁”推送到浏览器。
	- 配置 `proxy` 解决跨域问题
		- 浏览器不直接请求真实的后端接口，而是把请求发给本地的 Node 服务器, 其作为代理将其转发给真实的服务器, 解决跨域限制
- `mode`
	- 区分 `development` 和 `production`，Webpack 根据模式自动开启相应的内置优化