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
#### 3. 执行机制
- 初始阶段
	- 读取配置文件 `webpack.config.js`, 例化一个 `Compiler` 对象
- 编译构建阶段
	- 从 Entry 出发，针对每个模块 (每个文件就是一个模块) 调用对应 Loader 进行翻译
	- 翻译完后，利用 Parser（比如 Babel 解析器）把代码转换为 AST（抽象语法树）, 指这个模块的AST
	- 通过遍历 AST，找出这个模块依赖了哪些其他模块，然后递归这个步骤，最终画出一张完整的模块依赖图
- 生成输出阶段
	- 根据依赖图，把各个模块 (Module) 组装成一个个代码块（Chunk），然后再把这些 Chunk 转换成最终的文件（Assets）输出到硬盘上
#### 4. 性能优化
###### 优化构建速度 (npm run dev) (提升开发体验)
- 开启持久化缓存 
	- `cache: { type: 'filesystem' }`  
	- 每次打包都有大量代码是没有修改的，完全可以把第一次翻译好的 AST 和编译结果存到硬盘里，下次直接拿来用, 二次构建速度倍速提升 (webpack5)
- 开启多线程打包
	- 将 `thread-loader` 放在极其耗时的 loader（比如 `babel-loader`）之前
	- 由于 Node.js 是单线程的，面对几千个 JS 文件的 Babel 转换，它只能一个个排队干活。我们可以利用电脑的多核 CPU，让多个线程同时去翻译
	- 开启多线程本身也耗时, 因此仅适合大型体积文件
- 缩小 Loader 的处理范围
	- 配置 `include` 和 `exclude` 
	- 告诉 Webpack 哪些文件需要翻译，哪些绝对不需要翻译, 比如 `exclude: /node_modules/` 代表无需翻译
###### 优化产物体积 (提升首屏加载速度)
- 代码分割 (最重要)
	- 痛点
		- 如果不做分割，Webpack 会把所有的业务代码和第三方库（Vue、ECharts、Lodash）全部塞进一个巨大的 `app.js` 里。只要你改了一句业务代码，整个 `app.js` 的 Hash 值就会变，用户下次访问就必须重新下载好几 MB 的代码
	- 方案
		- 使用内置的 `SplitChunksPlugin`, 配置 `optimization.splitChunks`
		- `chunks: 'all'` 默认只拆分异步的(懒加载), 开启同步异步都进行分割
		- 将经常变动的业务代码，和几乎不怎么变动的第三方依赖 (node_modules)分成两组
		- 配合 Output 里的 `filename: '[name].[contenthash].js'` 来使用, 打包出两个文件：`app.[hash1].js` (你的业务代码) 和 `chunk-vendors.[hash2].js` (第三方库). 只有业务代码的内容变了，所以 `app.js` 的 hash 会变；而 `node_modules` 里的 Vue 源码没变，所以 `chunk-vendors.js` 的 hash 依然保持不变
	- 结果
		- 用户再次访问时，浏览器会直接从本地强缓存中瞬间读取庞大的 `chunk-vendors.js`，只需要花极短的时间下载几十 KB 的新业务代码
- 路由懒加载
	- 结合 Vue/React 的异步组件，使用 ES6 的动态导入语法 `import('./About.vue')`。Webpack 遇到这个语法，就会自动把 `About.vue` 单独打包成一个 Chunk，只有当用户真正点击跳转到该页面时，浏览器才会去发起网络请求下载它
- 抽离与压缩 CSS
	- 痛点: 默认情况下，CSS 是被打包进 JS 里的，运行时再动态生成 `<style>` 标签插入页面。这不仅会让 JS 变大，还会导致页面闪烁（因为要等 JS 执行完才有样式）。
	- 解决
		- 用 **`MiniCssExtractPlugin`** 把 CSS 强行从 JS 里抽离出来，变成独立的 `.css` 文件。
	    - 用 **`CssMinimizerWebpackPlugin`** 对 CSS 代码进行极度压缩（去掉空格、注释）。
- 摇树优化
	- 配置`optimization.usedExports: true` 并结合`package.json` 中 `"sideEffects": false` 利用 ES Module 的静态特性，剔除掉没有被使用的死代码。
#### 5. webpack5 新特性
- 模块联邦
	- 痛点: 之前两个项目想要共享一个组件, 只能发布到npm上然后分别安装, 组件每次更新, 两个项目都要重新打包发布
	- 模块联邦: 模块联邦允许不同的 Webpack 构建在“运行时”互相动态共享代码。项目 A 可以直接通过 URL 远程加载项目 B 跑在服务器上的特定共享组件，就像引入本地文件一样丝滑。它让前端的“微服务架构”变得极其简单
- 持久化缓存
	- 想要提升二次打包速度, 之前需要借助第三方插件, 现在只需配置 `cache: { type: 'filesystem' }`
- 资源模块
	- 痛点: 以前处理图片、字体等文件，我们必须配置 `file-loader`、`url-loader`、`raw-loader`，不仅配置长，还容易冲突
	- Webpack 5 内置了资源处理能力。你只需要写 `type: 'asset/resource'` 或者 `type: 'asset/inline'`，Webpack 就能自动帮你把小图片转成 base64，大图片输出成物理文件，彻底淘汰了那堆旧 Loade
- 更智能的tree shaking
	- webpack4只能识别顶层未使用的导出
	- Webpack 5 支持了嵌套的 Tree Shaking。如果一个对象导出了属性 A 和属性 B，但在深层嵌套中只有 A 被用到了，Webpack 5 现在能精准地把 B 给摇掉
#### 6. webpack 手脚架
- 脚手架（比如 `Vue CLI`、`Create React App`）本质上是一套‘预设好最佳实践’(`plugin`, `loader` 等) 的 Webpack/Vite 包装盒
- 即便手脚架配置好, 依然要做代码分割 (antd, reactflow,vant) , 按需引入 (antd, vant) 等配置
# Vite
#### 1. 冷启动
- 定义
	- vite是 `no-bundle` 架构, 利用原生 `ES Module` (浏览器能看懂 `import`) , 先启动服务, 再按需编译
- 机制
	- 当浏览器去访问这个本地地址时，浏览器自己会解析文件里的 `import` 语句，并向 Vite 服务器发起 HTTP 请求去要这个文件
	- Vite 服务器收到请求后，找到对应的源文件（比如 `App.vue` 或 `Button.tsx`），在服务端把它光速编译成普通的 JS 文件，然后立刻丢给浏览器
- 结果
	- Vite 把解析模块依赖的脏活累活，交给了浏览器去干。你访问哪个页面，它就只编译那个页面的文件（真正的按需加载），所以无论项目多大，冷启动永远是秒开
- 对比webpack
	- 打包，构建依赖图, 再启动服务, 所以慢
#### 2. 依赖预构建 (Pre-bundling)
- 背景
	- 虽然让浏览器自行去解析并http发请求, 但如果引入了如 `lodash` 等大型库浏览器不断解析发请求会卡死
- 解决
	- 引入预构建机制, 第一次启动vite时, 把你引入的那些庞大的第三方依赖（如 React、Vue、Lodash），预先打包成一个单独的 ESM 文件存放在 `.vite/deps` 缓存目录里
	- Vite 用来做预构建的工具是 `esbuild`。`esbuild` 是用 Go 语言编写的, 比普通的打包工具快10倍以上
#### 3. 热更新 (HMR)
- 你改了哪个文件，Vite 就只重新编译那一个文件，并通过 WebSocket 让浏览器去重新请求它。它的时间复杂度是 O(1)，和项目的总大小完全无关。所以一万个模块的项目，热更新依然是毫秒级。
- 对比webpack: 虽然也是发送对应文件的补丁, 但在计算的时间复杂度上是不一样的, 当某个文件修改时, 需要重新计算在依赖图的影响, 重新重新计算 Hash，重新生成这个模块所在 Chunk 的补丁，然后再推送给浏览器。会随着项目越来越大而变慢
#### 4. 生产环境: Rollup 打包
- vite只在开发环境中使用, 生产环境使用Rollup
- 因为在真实的线上生产环境中，用户的网络是不可控的（不像本地开发时网络延迟为 0）。如果我们在生产环境依然用未打包的 ESM，浏览器遇到多层级的 `import` 就会发起大量的串行 HTTP 请求（瀑布流），这会导致首屏加载极慢。
- 所以在生产环境，Vite 选择了底层架构非常成熟、天生对 ESM 极度友好、且 Tree-Shaking 能力极强的 Rollup 来进行代码分割和打包，以获得最佳的线上性能。
#### 5. vite配置
- `resolve.alias` : 会配置 `@` 指向 `src` 目录
- `server.proxy` : 反向代理, 解决跨域问题
	- `target: env.VITE_API_URL`
	- 通过 Vite 导出的 `loadEnv` 工具函数，手动去解析 `.env` 文件，才能在配置文件里动态读取代理地址和公用路径
- `Plugins` 插件
	- 比如Vue 项目引入 `@vitejs/plugin-vue`，React 引入 `@vitejs/plugin-react`
	- 配置 `unplugin-auto-import`（自动导入 Vue/React 的 API 如 `ref`, `useState`）和 `unplugin-vue-components`（自动按需导入 Element-Plus 或 Vant 等组件库）。这不仅能极大减少冗余代码，还能完美配合 Tree Shaking。”
- `build` 生产环境构建优化
	- `rollupOptions.manualChunks`: 代码分割, 把庞大的第三方库（如 Vue 核心、ECharts、Lodash）单独剥离出来，避免一个 vendor.js 过大，极致利用浏览器长缓存
	- `esbuild.drop`: 在生产环境打包时，自动剥离掉所有的 `console.log` 和 `debugger`，既减小体积又防泄漏
	- `rollupOptions.output.assetFileNames`: 静态资源分类输出, 把打包后的图片、CSS、JS 分门别类地放到单独的文件夹（如 `img/`, `css/`）里，让产物目录干净清爽