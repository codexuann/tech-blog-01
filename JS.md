
#### 数据类型基础
- 数据类型
	 - 基本数据类型
		 - 老: Number, String, Boolean, undifined, null
		 - 新: Symbol, BigInt
		 - 存在栈内存中, b = a 是深拷贝
		 - undefined和null
			 - undefined多为被动, 如变量未定义, 变量未初始化未赋值
				 - typeof undefined === 'undefined'
			 - null多为主动, 如对象为空, 或者显式释放对象
				 - typeof null === 'object'
	- 引用数据类型
		- Object (Function, Array, Date 都是对象) 
		- 存在堆内存中, b = a 是浅拷贝, 只复制引用地址
- `==`类型转换 
	- !的优先级高于== 所以先将!后面的转成布尔值再取反
	- 对象在在布尔上下文中始终为ture(空数组也是对象) (除了null)
	- 对象与原始值比较时, 会尝试调用自身toString或valueOf转化成原始值
	- 字符串或者布尔值在比较时, 会先转成数字再比较
	- `null==undifined`, 除此之外, null, undefined, NaN不等于任何值(NaN连自身都不等)
	- `===` 不会类型转换
- 判断数据类型
	- typeof
		- 基本数据类型判断准确, 可以用来判断number, string, boolean, undefined, symbol. 特例: null会判为object
		- 引用类型均判断为object 特例: function可以判断
		- 写法: 返回值均为小写字符串 
		  `typeof 'hello' === 'string'`
	- instanceof
		- 引用类型可以精确判断, 如Object, Function, Array
		- 基本类型不可以判断 (因为基本类型没有属性, 所以没有`__proto__`)
		- 写法: 返回布尔类型, 另外类型大写不加引号
		  `[1,2,3] instanceof Array === ture`
	- Object.prototype.toString.call()
		- 最精准, 可以判断任何数据类型
		- 写法, 返回字符串
		 `Object.prototype.toString.call(123) === '[object Number]`

#### Number与Math
- Math静态方法
	- Math.abs(-3), Math.max(1,2,3), Math.pow(2,3), Math.sqrt(16)
	- Math.ceil(3.2), Math.floor(3.2), Math.round(3.2)
	- Math.random()
```js
	// 生成[min,max]的随机整数
	function getRandom(min,max) {
		let num = Math.random()
		num = Math.floor(num*(max-min+1))+min
		return num 
	}
```
- 数字<->字符串
	- 数字->字符串
		- num.toString() 十进制转换
		  num.toString(2) 二进制, num.toString(16) 十六进制
		- num.toFixed(2)
			- 保留两位小数, 小数多则四舍五入, 小数少则添0
			- 返回的是String
	- 字符串->数字
		- Number('3.14')
		- parseInt('123'), parseFloat('3.14px') // 较Number更宽容

#### String
- 基本方法
	- 查询
		- str.indexOf('a'), str.lastIndexof('a'), str.includes('ab')
	- 替换
		- str.replaceAll('你好', 'Hello'), str.toUppercase(), str.toLowerCase()
	- 分割
		- str.slice(1,4) 截取`[1,4)`
		  str.slice(1,-1) 从第一截取, 去掉倒数第一个
		- str.substr(1,3) 从1开始截取3个
		- 如果只填一个参数, 默认从索引位置切到最后
- 字符串 <-> 数组
	- 字符串->数组
		- str.split(',') //按特定字符切, 返回数组  `let [int,dec] = str.split('.')`
		  str.split('') //每一个字符都切开
		- `let arr = Array.from(str)`
		- `let arr = [...str]`
	- 数组->字符串
		- `arr.join('')` // 用什么符号连起来, 如果不传参则默认逗号

#### Array
- 基本方法
	- 高级
		- forEach, map, filter, reduce, some, every, find, findIndex
	- 改变原数组
		- 增删
			- push, pop, unshift, shift
			- splice(start, deleteCount, item1, item2...)
		- 排序
			- arr.sort((a,b) => a-b) 升序
			- arr.sort((a,b) => b-a) 降序
		- 反转 arr.reverse()
	- 查询
		- indexOf, lastIndexOf, includes
	- 返回新数组
		- 切割 `arr.slice(2,5)` //不传参数则是数组浅拷贝
		- 连接 `arr.concat([2,3,4])`
- for in 和 for of
	- for in
		- 遍历对象的键名与原型链上所有可枚举属性键名
		- 可以用 `if(child.hasOwnProperty(key))`过滤原型链上属性
		- 适合遍历对象
	- for of 
		- 遍历对象的键值
		- 对象必须拥有`[Symbol.iterator]`属性(接口)
			- `[Symbol.iterator]`是一个函数返回迭代器对象, 对象中必须有next方法
			- 拥有该借口的类型有Array, String, Map, Set, arguments对象, NodeList对象
			- 普通的对象没有迭代器接口
		- 适合遍历有迭代器接口的对象
- 伪数组
	- 必须满足的条件 : 按索引存储数据, 拥有length属性
		- `const faker = {0:'a', 1:'b', 2:'c', length: 3}`
	- 不能改变长度, 不能使用数组原生方法
	- 常见的原生伪数组: arguments对象(函数中所有参数集合, 函数参数的...args中的args是真数组), NodeList对象(querySelectorAll('div'))
	- 最佳使用方案: 数组->伪数组
		- `let arr = Array.from(fakeArr)`
		- `let arr = [...fakeArr]`
		- `let arr = Array.prototype.slice.call(fakeArr)`
		- `let arr = Array.prototype.concat.apply([], fakeArr)`
- 深浅拷贝
	- 浅拷贝
		- 对象
			- `let newObj = Object.assign({}, oldObj)`
			- `let newObj = {...oldObj}`
		- 数组
			- `let newArr = [...oldArr]`
			- `let newArr = oldArr.slice()`
			- `let newArr = oldArr.concat([])`
	- 深拷贝
		- 手写递归
		- 适合简单场景对数组/对象的深拷贝: `JSON.parse(JSON.stringifu(oldObj))`
		- 第三方库: `import _ from 'ladash'; let newObj = _.cloneDeep(oldObj)`

#### Set, Map
- set
	- 特点: 哈希表, 且成员的值是唯一的
	- 场景: 数组去重, 跨速查找
	- 用法: `new Set(), set.add(value), set.delete(key), set.has(value), set.size `
- map
	- 特点: 哈希表, 各种类型的值（包括对象、函数、DOM节点）都可以当作键
	- 用法: `new Map(), map.set(key, value), map.delete(key), map.has(key), map.get(key), map.size`
- weakSet/weakMap
	- 把一个对象存入普通set/map中, 即使在所有地方都删除了这个对象, 垃圾回收仍不会回收这个对象, 因为其被set/map引用着
	- weakSet/weakMap是弱引用, 如果一个地方只被 WeakMap 引用，而没有其他地方引用它，垃圾回收机制会无视 WeakMap 的引用，直接把这个对象回收掉，WeakMap 里对应的这一项也会自动消失
	- 特点: 
		- key只能是对象, 不能是数字或者字符串
		- 不能遍历, 没有forEach, 没有size. 因为里面东西可能随时会回收无法计数
- 遍历
	- `for([key,value] of map) {}`
	- `map.forEach((value,key)=>{})`
#### Function
- 作用域
	- 声明类型: var, let, const, function
	- 作用域类型
		- 全局作用域: 在最外层定义 所有声明类型
		- 函数作用域: 在函数内定义 所有声明类型
		- 块级作用域: 在`{}`定义的变量 
			- var声明的无块级作用域
			- 严格模式下function有块级作用域, 非严格模式下会很奇怪
			- 除去定义Object的{}
	- 作用域链
		- 当在一个函数里使用变量时, 会沿着作用域链层层往上找, 直到找到函数定义
		- 词法作用域: 作用域链由函数声明时的位置决定, 而非函数调用时的位置
	- 变量提升
		- js执行前引擎会扫描将声明的变量提升到当前作用域顶部
		- fuction声明的变量既声明名字也声明赋值, 可以在定义前直接调用
			- function fn() {} 而非 const fn = function() {}
		- var声明的变量只声明名字不声明赋值, 值是undefined. 可以对变量多次声明
		- let和const的变量会提升, 但在声明前存在"暂时性死区", 声明前访问会报错. 同时不能重复声明
	- for循环
		- var a = 0: var没有块级作用域, 所以a在函数作用域或全局作用域中, 所有a共用一个地址
		- let a = 0: let有块级作用域, 所以每一次循环的a都是一个新的地址, 相互不干涉
- 闭包
	- 定义: 一个函数和其词法作用域(作用域链)的环境绑在一起, 使其能够访问外部作用域的内部变量, 即使该函数在其外部作用域之外执行
	- 内存驻留/数据私有: 
		- 当函数执行完, 正常情况下的局部变量会被GC垃圾回收, 但如果函数的内部函数被返回到了外部, 且内部函数引用了外部函数的变量, GC不会回收这些局部变量, 形成闭包
		- 函数也是一个对象, 每次调用外层函数会生成一个新的内存环境, 所以执行多次外层函数内部变量不会发生冲突
	- 应用: 数据私有化(计数器), 柯里化, 防抖节流保存定时器状态
	- 缺陷: 内存泄露, 调试困难, 性能干扰
	- 消除闭包
		- 手动将引用变量赋值为null
		- 使用weakmap来持有对象引用, 不会阻止垃圾回收
- 箭头函数
	- this静态绑定
	- 没有prototype属性
	- 没有arguments对象, 但可以在参数中写...args
- 构造函数
	- 函数名首字母必须大写
	- this给对象设置属性 `this.name = name`
	- 原型让所有对象共用一个方法, 节省内存
	 `People.prototype.sing = function() {}`

#### Object
- Object静态方法
	- Object.keys(obj) 返回键名数组
	- Object.values(obj) 返回键值数组
	- Object.entries(obj) 返回键值对二维数组
	- Object.assign(targetObj, ...sourceObj) 将后面对象属性浅拷贝到前一个对象中
	- Object.create(fn.prototype) 
	  const obj={} 的`__proto__`默认指向Object.prototype, 该方法可以改变对象原型链指向
- 原型与原型链
	- 每一个构造函数都有prototype属性, 包含了该构造函数的所有实例共享的属性和方法
	- 每一个实例对象都有`__proto__`指针, 指向构造函数的prototype属性
	- 原型链查找机制
		- 访问一个对象的属性或方法时, 如果不存在, 就沿着原型链向上查找, 直到找到
		- `User.prototype` -> `Object.prototype` -> `null`
	- `obj.__proto__ === Object.getPrototypeOf(obj)`
- this的绑定方式(优先级从高到低)
	- 箭头函数
		- 固定绑定在作用域外的对象上
		- 巧记 : 对象中普通的定义箭头函数绑定在windows/undifined上; function(settimeout(箭头函数))绑定在该对象上
	- new绑定
		- 新建对象实例, this绑定在对象的实例上
		- 绑定流程
			- `typeof fn === 'function'`
			  判断是否是函数(普通对象或箭头函数不可以new)
			- `const obj=Object.create(fn.prototype)`  
			  绑定方法: 创建对象并将原型链指向函数的原型
			- `fn.call(obj, ...args)`
			  绑定属性: 执行构造函数并将this绑定到新对象
			- 返回对象: 构造函数没有手动返回对象, 则返回这个新对象
	- 显示绑定
		- 由call, apply, bind指定的对象
		- call. apply, bind
			- func.call(obj,a,b)
			- func.apply(obj,【a, b】)
			- func.bind(obj, a, b)
			- call和apply立即执行该函数, 只是把this换掉
			- bind返回了新的函数可以随时调用
	- 隐式绑定
		- 对象调用函数 obj.func() 函数中this指向调用它的对象
	- 默认绑定
		- 独立调用函数时, 非严格模式指向window，严格模式指向undefined
		- settimeout中的回调函数/对象中用箭头函数定义的方法/对象中函数命名给了新值再调用, 都属于独立调用函数, 没有任何对象调用
- es6 class
```js
	// 父类
	class Animal {
	  constructor(name) {
	    this.name = name;
	  }
	  move() {
	    console.log(`${this.name} 动了`);
	  }
	}
	
	// 子类
	class Dog extends Animal {
	  constructor(name, breed) {
	    // 必须先调用 super()，否则就没有 `this`
	    // 这相当于以前的 Animal.call(this, name)
	    super(name); 
	    this._breed = breed;
	  }
	  get bread() {
		  return this._breed
	  }
	  set bread(val){
		  this._bread = val
	  }
	  bark() {
	    console.log("汪汪!");
	  }
	}
	
	const d = new Dog("旺财", "柯基"); 
	d.move(); // 继承自 Animal 
	d.bark(); // 自己的方法
	console.log(d.bread) //触发get
	d.bread = "哈士奇" //触发set

```

#### Promise

#### es6新特性