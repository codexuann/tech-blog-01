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