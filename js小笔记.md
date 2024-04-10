https://wangdoc.com/javascript/

https://www.liaoxuefeng.com/wiki/1022910821149312

JavaScript 语言本身，虽然是一种解释型语言，但是在现代浏览器中，JavaScript 都是编译后运行。



### 变量

let const var



### 模块

ESM 文件通常使用 `.mjs` 扩展名，以便与 CommonJS 模块进行区分。但在Node.js 13.2.0及更高版本中，你也可以在 `.js` 文件中使用 ESM，只需在 `package.json` 中添加 `"type": "module"` 字段。

#### ES6与commnoJS对比

ES6 模块的设计思想是尽量的**静态化**，使得编译时就能确定模块的依赖关系，以及输入和输出的变量。CommonJS 和 AMD 模块，都只能在运行时确定这些东西。

**ES6模块系统：**

1. **静态分析：** ES6模块系统是静态的，这意味着在解析阶段就能够确定模块之间的依赖关系。这种静态分析的特性使得引擎能够在编译时优化模块的加载，而不需要在运行时进行解析。这也为引擎提供了更多的优化机会。
2. **单一实例：** ES6模块在一个程序生命周期内只会初始化一次。当你多次导入同一个模块时，实际上是在访问已经初始化的模块实例。这导致了动态绑定的特性，即当导出模块内部的值发生变化时，导入模块能够实时获取到这些变化。

**CommonJS模块系统：**

1. **运行时加载：** CommonJS模块系统是在运行时加载的，而不是在编译时。这意味着在运行时才能确定模块的依赖关系，模块的加载和执行是动态的过程。
2. **值的拷贝：** 在CommonJS中，`module.exports`和`exports`输出的是值的拷贝，而不是引用。这导致了模块之间的连接是静态的，导入模块得到的是导出模块的一个副本，而不是直接引用导出模块的变量。

**语法差异**

- **ES6模块:**
  - 使用 `import` 和 `export` 语法。
  - 可以使用 `export default` 导出一个默认值。

```js
javascriptCopy code
// ES6 模块
// moduleA.js
export const variableA = 42;
export function funcA() { /* ... */ }

// moduleB.js
import { variableA, funcA } from './moduleA';
```

- **CommonJS:**
  - 使用 `require` 函数和 `module.exports` 对象。
  - 没有直接支持默认导出，但可以通过 `module.exports = ...` 实现。

```js
javascriptCopy code
// CommonJS 模块
// moduleA.js

// 方法一 exports
exports.variableA = 42;
exports.funcA = function() { /* ... */ };

// 方法二 module.exports 推荐
let variableA = 42;
function funcA() {
  // ...
}

module.exports = {
  variableA,
  funcA
};

// moduleB.js
const { variableA, funcA } = require('./moduleA');
```



#### 严格模式

ES6 的模块自动采用严格模式，不管你有没有在模块头部加上`"use strict";`。

严格模式主要有以下限制。

- 变量必须声明后再使用
- 函数的参数不能有同名属性，否则报错
- 不能使用`with`语句
- 不能对只读属性赋值，否则报错
- 不能使用前缀 0 表示八进制数，否则报错
- 不能删除不可删除的属性，否则报错
- 不能删除变量`delete prop`，会报错，只能删除属性`delete global[prop]`
- `eval`不会在它的外层作用域引入变量
- `eval`和`arguments`不能被重新赋值
- `arguments`不会自动反映函数参数的变化
- 不能使用`arguments.callee`
- 不能使用`arguments.caller`
- 禁止`this`指向全局对象
- 不能使用`fn.caller`和`fn.arguments`获取函数调用的堆栈
- 增加了保留字（比如`protected`、`static`和`interface`）



#### export 命令

一个模块就是一个独立的文件。该文件内部的所有变量，外部无法获取。如果你希望外部能够读取模块内部的某个变量，就必须使用`export`关键字输出该变量。

``` js
// 写法一
export var firstName = 'Michael';
export var lastName = 'Jackson';
export var year = 1958;
// 写法二
var firstName = 'Michael';
var lastName = 'Jackson';
var year = 1958;
export { firstName, lastName, year };
```



`export`命令除了输出变量，还可以输出函数或类（class）。

```js
// 写法一
export function multiply(x, y) {
  return x * y;
};

function v1() { ... }
function v2() { ... }
function v3() { ... }
// 写法二
export {
  v1 as streamV1, //重命名了函数v1和v2的对外接口
  v2 as streamV2,
  v2 as streamLatestVersion,
	v3
};
```



`export`语句输出的接口，与其对应的值是动态绑定关系，即通过该接口，可以取到模块内部实时的值。

```js
export var foo = 'bar';
setTimeout(() => foo = 'baz', 500);
```

上面代码输出变量`foo`，值为`bar`，500 毫秒之后变成`baz`。这一点与 CommonJS 规范完全不同。CommonJS 模块输出的是值的缓存，不存在动态更新。



`export`命令可以出现在模块的任何位置，只要处于模块顶层就可以。如果处于块级作用域内，就会报错，下一节的`import`命令也是如此。这是因为处于条件代码块之中，就没法做静态优化了，违背了 ES6 模块的设计初衷。

```js
function foo() {
  export default 'bar' // SyntaxError
}
foo()
```



#### import 命令

使用`export`命令定义了模块的对外接口以后，其他 JS 文件就可以通过`import`命令加载这个模块。大括号里面的变量名，必须与被导入模块（`profile.js`）对外接口的名称相同。如果想为输入的变量重新取一个名字，`import`命令要使用`as`关键字，将输入的变量重命名。注意，`import`命令具有提升效果，会提升到整个模块的头部，首先执行。

```js
// main.js
import { firstName, lastName, year } from './profile.js';
import { lastName as surname } from './profile.js';

function setName(element) {
  element.textContent = firstName + ' ' + lastName;
}
```

`import`命令输入的变量都是只读的，因为它的本质是输入接口。但是，如果`a`是一个对象，改写`a`的属性是允许的。

```js
import {a} from './xxx.js'

a = {}; // Syntax Error : 'a' is read-only;


import {b} from './xxx.js'

b.foo = 'hello'; // 合法操作
```

由于`import`是静态执行，所以不能使用表达式和变量，这些只有在运行时才能得到结果的语法结构。

```js
// 报错
import { 'f' + 'oo' } from 'my_module';

// 报错
let module = 'my_module';
import { foo } from module;

// 报错
if (x === 1) {
  import { foo } from 'module1';
} else {
  import { foo } from 'module2';
}
```

`import`语句会执行所加载的模块，因此可以有下面的写法。

```js
import 'lodash';
```



#### export default 命令

从前面的例子可以看出，使用`import`命令的时候，用户需要知道所要加载的变量名或函数名，否则无法加载。为了给用户提供方便，让他们不用阅读文档就能加载模块，就要用到`export default`命令，为模块指定默认输出。其他模块加载该模块时，`import`命令可以为该匿名函数指定任意名字。需要注意的是，这时`import`命令后面，不使用大括号。

```js
// export-default.js
export default function () {
  console.log('foo');
}
// import-default.js
import customName from './export-default';
customName(); // 'foo'
```

`export default`命令用在非匿名函数前，也是可以的。

```js
// export-default.js
export default function foo() {
  console.log('foo');
}

// 或者写成

function foo() {
  console.log('foo');
}

export default foo;
```

上面代码中，`foo`函数的函数名`foo`，在模块外部是无效的。加载的时候，视同匿名函数加载。

显然，一个模块只能有一个默认输出，因此`export default`命令只能使用一次。所以，import命令后面才不用加大括号，因为只可能唯一对应`export default`命令。



本质上，`export default`就是输出一个叫做`default`的变量或方法，然后系统允许你为它取任意名字。
正是因为`export default`命令其实只是输出一个叫做`default`的变量，所以它后面不能跟变量声明语句。

```js
// 正确
export var a = 1;

// 正确
var a = 1;
export default a;

// 错误
export default var a = 1;

// 正确
export default 42;

// 报错
export 42;
```



#### import()

前面介绍过，`import`命令会被 JavaScript 引擎静态分析，先于模块内的其他语句执行（`import`命令叫做“连接” binding 其实更合适）。这样的设计，固然有利于编译器提高效率，但也导致无法在运行时加载模块。

`import()`函数，支持动态加载模块。

`import()`函数可以用在任何地方，不仅仅是模块，非模块的脚本也可以使用。它是运行时执行，也就是说，什么时候运行到这一句，就会加载指定的模块。另外，`import()`函数与所加载的模块没有静态连接关系，这点也是与`import`语句不相同。`import()`类似于 Node.js 的`require()`方法，区别主要是前者是异步加载，后者是同步加载。

`import()`返回一个 Promise 对象

```js
async function renderWidget() {
  const container = document.getElementById('widget');
  if (container !== null) {
    // 等同于
    // import("./widget").then(widget => {
    //   widget.render(container);
    // });
    const widget = await import('./widget.js');
    widget.render(container);
  }
}
```

```js
import('./myModule.js')
.then(({export1, export2}) => {
  // ...·
});
```



### 定时器

是一个异步

``` js
setTimeout(function () {
console.log( 'suprise' )
}，2000)

var timer = setInterval(function () {
  console.log('间隔2s输出一次')
}，2000)

setTimeout(function (){
  clearInterval(timer)
}，6000) //停止




```



### 解构赋值

**变量解构赋值**

```js

// 基本数组解构
let [a, b] = [1, 2];
console.log(a); // 输出 1
console.log(b); // 输出 2

// 跳过某些元素
let [c, , d] = [3, 4, 5];
console.log(c); // 输出 3
console.log(d); // 输出 5

```

**对象解构赋值**

```js

// 基本对象解构
let { x, y } = { x: 10, y: 20 };
console.log(x); // 输出 10
console.log(y); // 输出 20

// 指定新的变量名
let { a: var1, b: var2 } = { a: 'apple', b: 'banana' };
console.log(var1); // 输出 'apple'
console.log(var2); // 输出 'banana'

// 默认值
let { p = 50, q = 60 } = { p: 30 };
console.log(p); // 输出 30
console.log(q); // 输出 60
```

**函数参数解构赋值**

```js
javascriptCopy code
// 函数参数解构
function printCoordinates({ x, y }) {
  console.log(`x: ${x}, y: ${y}`);
}

printCoordinates({ x: 100, y: 200 }); // 输出 "x: 100, y: 200"
```



### **this**关键字总结

1. **全局上下文：** 当代码在全局作用域中执行时，`this` 指向全局对象，在浏览器中通常是 `window` 对象。

   ```js
   javascriptCopy code
   console.log(this); // 在全局上下文中，this 指向全局对象（在浏览器中通常是 window）
   ```

2. **函数上下文：** 在函数内部，`this` 的值取决于函数是如何被调用的。

   - **作为函数调用：** 当函数作为普通函数调用时，`this` 指向全局对象或者是 `undefined`（在严格模式下）。

     ```
     javascriptCopy code
     function exampleFunction() {
         console.log(this);
     }
     
     exampleFunction(); // 在非严格模式下，this 指向全局对象
     ```

   - **作为对象方法调用：** 当函数作为对象的方法调用时，`this` 指向调用该方法的对象。

     ```
     javascriptCopy code
     var obj = {
         method: function() {
             console.log(this);
         }
     };
     
     obj.method(); // this 指向 obj
     
     var obj = {
         birth: 1990,
         getAge: function () {
             var b = this.birth; // 1990
             var fn = function () {
                 return new Date().getFullYear() - this.birth; // this指向window或undefined
             };
             return fn();
         }
     };
     ```
   
   - **构造函数中：** 当函数作为构造函数使用（使用 `new` 关键字创建实例）时，`this` 指向新创建的对象。
   
     ```
     javascriptCopy code
     function Constructor() {
         this.property = 'value';
         console.log(this);
     }
     
     var instance = new Constructor(); // this 指向新创建的实例对象
     ```
   
   - **使用 `call`、`apply` 或 `bind` 方法：** 这些方法可以显式地设置函数内部的 `this` 值。

     ```
     javascriptCopy code
     function exampleFunction() {
         console.log(this);
     }
     
     var obj = { name: 'Object' };
     
     exampleFunction.call(obj); // 使用 call 方法设置 this 指向 obj
     ```
   
   - **箭头函数：** 箭头函数没有自己的 `this`，它继承父级上下文中的 `this`。
   
     ```
     javascriptCopy code
     var obj = {
         method: function() {
             var innerFunction = () => {
                 console.log(this);
             };
             innerFunction();
         }
     };
     
     obj.method(); // this 指向 obj，因为箭头函数继承了父级上下文的 this
     ```



### 异步处理

promise和async函数
https://www.liaoxuefeng.com/wiki/1022910821149312/1023024413276544

### AJAX

Asynchronous JavaScript and XML

有三种方法

**原生AJAX**

```js
const xhr = new XMLHttpRequest()
xhr.open('GET'，"http://wuyou.com/common/get?name=吴悠&age=18')
xhr,send()
xhr.onreadystatechange = function () {
		if (xhr.readyState == XMLHttpRequest.DONE && xhr.status == 200) {
      console.log(JSON.parse(xhr.responseText))
    }
}
```



``` js
xhr.open('POST'，'http://wuyou. com/common/post')
xhr.setRequestHeader('Content-Type','application/x-www-form-urlencoded' )
xhr.send('name=吴悠&age=18')
xhr.onreadystatechange = function () {
		if (xhr.readyState == XMLHttpRequest.DONE && xhr.status == 200) {
      console.log(JSON.parse(xhr.responseText))
    }
}
```

**axios**

``` js

(async () => {
  const ins = axios.create({
  	baseURL:'http://wuyou.com/common
  })
	const res1 = await ins.get('/get',{
    params: {
			name:吴悠'
			age: 18
    }
  })
 
	const res2 = await ins.post('/post',{
    name:'吴悠'，
		age: 18
  })
console.log( res2.data)
})()
```

**Fetch api**

``` js
async function get(url) {
    let resp = await fetch(url);
    return resp.json();
}

// 发送异步请求:
get('/api/categories').then(data => {
    let textarea = document.getElementById('fetch-response-text');
    textarea.value = JSON.stringify(data);
});

```













### DOM

DOM 是 JavaScript 操作网页的接口，全称为“**文档对象模型”（Document Object Model）**。它的作用是将网页转为一个 JavaScript 对象，从而可以用脚本进行各种操作（比如增删内容）。

浏览器会根据 DOM 模型，将结构化文档（比如 HTML 和 XML）解析成一系列的**节点**，再由这些节点组成一个树状结构（DOM Tree）。所有的节点和最终的树状结构，都有规范的对外接口。

#### 节点

DOM 的最小组成单位叫做节点（node）。文档的树形结构（DOM 树），就是由各种不同类型的节点组成。每个节点可以看作是文档树的一片叶子。

节点的类型有七种。

- `Document`：整个文档树的顶层节点
- `DocumentType`：`doctype`标签（比如`<!DOCTYPE html>`）
- `Element`：网页的各种HTML标签（比如`<body>`、`<a>`等）
- `Attr`：网页元素的属性（比如`class="right"`）
- `Text`：标签之间或标签包含的文本
- `Comment`：注释
- `DocumentFragment`：文档的片段

浏览器提供一个原生的节点对象`Node`，上面这七种节点都继承了`Node`，因此具有一些共同的属性和方法。

#### 节点树

一个文档的所有节点，按照所在的层级，可以抽象成一种树状结构。这种树状结构就是 DOM 树。它有一个顶层节点，下一层都是顶层节点的子节点，然后子节点又有自己的子节点，就这样层层衍生出一个金字塔结构，又像一棵树。

浏览器原生提供`document`节点，代表整个文档。



#### 常用的DOM方法

**获取节点**

``` js
// 返回ID为'test'的节点：
var test = document.getElementById('test');

// 先定位ID为'test-table'的节点，再返回其内部所有tr节点：
var trs = document.getElementById('test-table').getElementsByTagName('tr');

// 先定位ID为'test-div'的节点，再返回其内部所有class包含red的节点：
var reds = document.getElementById('test-div').getElementsByClassName('red');
```

**更新DOM**

一种是修改`innerHTML`属性，这个方式非常强大，不但可以修改一个DOM节点的文本内容，还可以直接通过HTML片段修改DOM节点内部的子树：

```js
// 获取<p id="p-id">...</p>
var p = document.getElementById('p-id');
// 设置文本为abc:
p.innerHTML = 'ABC'; // <p id="p-id">ABC</p>
// 设置HTML:
p.innerHTML = 'ABC <span style="color:red">RED</span> XYZ';
// <p>...</p>的内部结构已修改
```

第二种是修改`innerText`或`textContent`属性，这样可以自动对字符串进行HTML编码，保证无法设置任何HTML标签：

```js
// 获取<p id="p-id">...</p>
var p = document.getElementById('p-id');
// 设置文本:
p.innerText = '<script>alert("Hi")</script>';
// HTML被自动编码，无法设置一个<script>节点:
// <p id="p-id">&lt;script&gt;alert("Hi")&lt;/script&gt;</p>
```

两者的区别在于读取属性时，`innerText`不返回隐藏元素的文本，而`textContent`返回所有文本。另外注意IE<9不支持`textContent`。

**插入DOM**

有两个办法可以插入新的节点。一个是使用`appendChild`，把一个子节点添加到父节点的最后一个子节点。例如：

```html
<!-- HTML结构 -->
<p id="js">JavaScript</p>
<div id="list">
    <p id="java">Java</p>
    <p id="python">Python</p>
    <p id="scheme">Scheme</p>
</div>
```

把`<p id="js">JavaScript</p>`添加到`<div id="list">`的最后一项：

```js
var
    js = document.getElementById('js'),
    list = document.getElementById('list');
list.appendChild(js);
```

现在，HTML结构变成了这样：

```html
<!-- HTML结构 -->
<div id="list">
    <p id="java">Java</p>
    <p id="python">Python</p>
    <p id="scheme">Scheme</p>
    <p id="js">JavaScript</p>
</div>
```

因为我们插入的`js`节点已经存在于当前的文档树，因此这个节点首先会从原先的位置删除，再插入到新的位置。



更多的时候我们会从零创建一个新的节点，然后插入到指定位置：

```js
var
    list = document.getElementById('list'),
    haskell = document.createElement('p');
haskell.id = 'haskell';
haskell.innerText = 'Haskell';
list.appendChild(haskell);
```

这样我们就动态添加了一个新的节点：

```js
<!-- HTML结构 -->
<div id="list">
    <p id="java">Java</p>
    <p id="python">Python</p>
    <p id="scheme">Scheme</p>
    <p id="haskell">Haskell</p>
</div>
```





### 立即执行函数表达式（Immediately Invoked Function Expression，IIFE）

```js
(function() {
    // 函数体
    console.log("Hello, IIFE!");
})();


(function(message) {
    console.log(message);
})("Hello, IIFE with parameter!");

```

在这个例子中，外层的括号 `()` 将函数声明转换为函数表达式，然后后面的 `()` 立即调用了这个函数。

这种写法有几个作用：

1. **创建私有作用域：** IIFE 可以创建一个独立的作用域，避免变量污染全局作用域。
2. **封装变量：** 在 IIFE 中声明的变量不会被添加到全局对象中，这有助于避免命名冲突。
3. **模块化代码：** IIFE 可以被用作创建模块，通过返回值暴露出需要共享的接口。
4. **防止函数声明冲突：** 将函数声明转为表达式，可以避免函数声明与其他代码发生冲突的问题。

### 基础


JavaScript 是一种轻量级的脚本语言。所谓“脚本语言”（script language），指的是它不具备开发操作系统的能力，而是只用来编写控制其他大型应用程序（比如浏览器）的“脚本”。

JavaScript 也是一种**嵌入式（embedded）语言**。它本身提供的核心语法不算很多，只能用来做一些数学和逻辑运算。**JavaScript 本身不提供任何与 I/O（输入/输出）相关的 API，都要靠宿主环境（host）提供**，所以 JavaScript 只合适嵌入更大型的应用程序环境，去调用宿主环境提供的底层 API。

#### 变量

JavaScript 是一种动态类型语言，也就是说，变量的类型没有限制，变量可以随时更改类型。

```js
var a = 1;
a = 'hello';
```

如果使用`var`重新声明一个已经存在的变量，是无效的。

```js
var x = 1;
var x;
x // 1
```

JavaScript 引擎的工作方式是，先解析代码，获取所有被声明的变量，然后再一行一行地运行。这造成的结果，就是所有的变量的声明语句，都会被提升到代码的头部，这就叫做变量提升（hoisting）。



JavaScript 引擎的工作方式是，先解析代码，获取所有被声明的变量，然后再一行一行地运行。这造成的结果，就是所有的变量的声明语句，都会被提升到代码的头部，这就叫做变量提升（hoisting）。

```js
console.log(a);
var a = 1;
```

上面代码首先使用`console.log`方法，在控制台（console）显示变量`a`的值。这时变量`a`还没有声明和赋值，所以这是一种错误的做法，但是实际上不会报错。因为存在变量提升，真正运行的是下面的代码。

```js
var a;
console.log(a);
a = 1;
```

最后的结果是显示`undefined`，表示变量`a`已声明，但还未赋值。



#### 数据类型

- 数值（number）：整数和小数（比如`1`和`3.14`）。
- 字符串（string）：文本（比如`Hello World`）。
- 布尔值（boolean）：表示真伪的两个特殊值，即`true`（真）和`false`（假）。
- `undefined`：表示“未定义”或不存在，即由于目前没有定义，所以此处暂时没有任何值。
- `null`：表示空值，即此处的值为空。
- 对象（object）：各种值组成的集合。

通常，数值、字符串、布尔值这三种类型，合称为原始类型（primitive type）的值，即它们是最基本的数据类型，不能再细分了。对象则称为合成类型（complex type）的值，因为一个对象往往是多个原始类型的值的合成，可以看作是一个存放各种值的容器。至于`undefined`和`null`，一般将它们看成两个特殊值。

对象是最复杂的数据类型，又可以分成三个子类型。

- 狭义的对象（object）
- 数组（array）
- 函数（function）

#### 数值

#### 整数和浮点数

JavaScript 内部，所有数字都是以64位浮点数形式储存，即使整数也是如此。所以，`1`与`1.0`是相同的，是同一个数。

```
1 === 1.0 // true
```

这就是说，JavaScript 语言的底层根本没有整数，所有数字都是小数（64位浮点数）。容易造成混淆的是，某些运算只有整数才能完成，此时 JavaScript 会自动把64位浮点数，转成32位整数，然后再进行运算，参见《运算符》一章的“位运算”部分。

由于浮点数不是精确的值，所以涉及小数的比较和运算要特别小心。

```js
0.1 + 0.2 === 0.3
// false

0.3 / 0.1
// 2.9999999999999996

(0.3 - 0.2) === (0.2 - 0.1)
// false
```

#### 字符串

如果长字符串必须分成多行，可以在每一行的尾部使用反斜杠。

```
var longString = 'Long \
long \
long \
string';

longString
// "Long long long string"
```

上面代码表示，加了反斜杠以后，原来写在一行的字符串，可以分成多行书写。但是，输出的时候还是单行，效果与写在同一行完全一样。注意，反斜杠的后面必须是换行符，而不能有其他字符（比如空格），否则会报错。

字符串与数组

字符串可以被视为字符数组，因此可以使用数组的方括号运算符，用来返回某个位置的字符（位置编号从0开始）。

```
var s = 'hello';
s[0] // "h"
s[1] // "e"
s[4] // "o"

// 直接对字符串使用方括号运算符
'hello'[1] // "e"
```

如果方括号中的数字超过字符串的长度，或者方括号中根本不是数字，则返回`undefined`。

```
'abc'[3] // undefined
'abc'[-1] // undefined
'abc'['x'] // undefined
```

但是，字符串与数组的相似性仅此而已。实际上，无法改变字符串之中的单个字符。

```
var s = 'hello';

delete s[0];
s // "hello"

s[1] = 'a';
s // "hello"

s[5] = '!';
s // "hello"
```

上面代码表示，字符串内部的单个字符无法改变和增删，这些操作会默默地失败。

#### 对象

生成方法

对象（object）是 JavaScript 语言的核心概念，也是最重要的数据类型。

什么是对象？简单说，对象就是一组“键值对”（key-value）的集合，是一种无序的复合数据集合。

```
var obj = {
  foo: 'Hello',
  bar: 'World'
};
```

上面代码中，大括号就定义了一个对象，它被赋值给变量`obj`，所以变量`obj`就指向一个对象。该对象内部包含两个键值对（又称为两个“成员”），第一个键值对是`foo: 'Hello'`，其中`foo`是“键名”（成员的名称），字符串`Hello`是“键值”（成员的值）。键名与键值之间用冒号分隔。第二个键值对是`bar: 'World'`，`bar`是键名，`World`是键值。两个键值对之间用逗号分隔。

键名

对象的所有键名都是字符串（ES6 又引入了 Symbol 值也可以作为键名），所以加不加引号都可以。

```
var obj = {
  'foo': 'Hello',
  'bar': 'World'
};
```

如果键名是数值，会被自动转为字符串。

```
var obj = {
  1: 'a',
  3.2: 'b',
  1e2: true,
  1e-2: true,
  .234: true,
  0xFF: true
};

obj
// Object {
//   1: "a",
//   3.2: "b",
//   100: true,
//   0.01: true,
//   0.234: true,
//   255: true
// }

obj['100'] // true
```

上面代码中，对象`obj`的所有键名虽然看上去像数值，实际上都被自动转成了字符串。

如果键名不符合标识名的条件（比如第一个字符为数字，或者含有空格或运算符），且也不是数字，则必须加上引号，否则会报错。

```
// 报错
var obj = {
  1p: 'Hello World'
};

// 不报错
var obj = {
  '1p': 'Hello World',
  'h w': 'Hello World',
  'p+q': 'Hello World'
};
```

上面对象的三个键名，都不符合标识名的条件，所以必须加上引号。



对象采用大括号表示，这导致了一个问题：如果行首是一个大括号，它到底是表达式还是语句？

```
{ foo: 123 }
```

JavaScript 引擎读到上面这行代码，会发现可能有两种含义。第一种可能是，这是一个表达式，表示一个包含`foo`属性的对象；第二种可能是，这是一个语句，表示一个代码区块，里面有一个标签`foo`，指向表达式`123`。

为了避免这种歧义，JavaScript 引擎的做法是，如果遇到这种情况，无法确定是对象还是代码块，一律解释为代码块。

```
{ console.log(123) } // 123
```

如果要解释为对象，最好在大括号前加上圆括号。因为圆括号的里面，只能是表达式，所以确保大括号只能解释为对象。

```js
({ foo: 123 }) // 正确
({ console.log(123) }) // 报错
```



JavaScript 允许属性的“后绑定”，也就是说，你可以在任意时刻新增属性，没必要在定义对象的时候，就定义好属性。

```
var obj = { p: 1 };

// 等价于

var obj = {};
obj.p = 1;
```



### js 面向对象

JavaScript不区分类和实例的概念，而是通过原型（prototype）来实现面向对象编程。JavaScript的原型链和Java的Class区别就在，它没有“Class”的概念，所有对象都是实例，所谓继承关系不过是把一个对象的原型指向另一个对象而已。

Object.create()

```js
Object.create(proto[, propertiesObject])

//proto: 必需。表示新创建的对象的原型对象。可以为 null 或者一个对象。

//propertiesObject: 可选。包含一个或多个属性的对象，其中属性是通过 Object.defineProperty 方法添加到新创建的对象中的。这个参数通常用于定义新对象的属性。
```

该方法可以传入一个原型对象，并创建一个基于该原型的新对象，但是新对象什么属性都没有（若不填第二个参数），因此，我们可以编写一个函数来创建xiaoming。

```js
// 原型对象:
var Student = {
    name: 'Robot',
    height: 1.2,
    run: function () {
        console.log(this.name + ' is running...');
    }
};

function createStudent(name) {
    // 基于Student原型创建一个新对象:
    var s = Object.create(Student);
    // 初始化新对象:
    s.name = name;
    return s;
}

var xiaoming = createStudent('小明');
xiaoming.run(); // 小明 is running...
xiaoming.__proto__ === Student; // true
```



#### 构造函数

除了直接用`{ ... }`创建一个对象外，JavaScript还可以用一种构造函数的方法来创建对象。它的用法是，先定义一个构造函数：

```js
function Student(name) {
    this.name = name;
    this.hello = function () {
        alert('Hello, ' + this.name + '!');
    }
}
```

这确实是一个普通函数，但是在JavaScript中，可以用关键字`new`来调用这个函数，并返回一个对象：

```js
var xiaoming = new Student('小明');
xiaoming.name; // '小明'
xiaoming.hello(); // Hello, 小明!
```

*注意*，如果不写`new`，这就是一个普通函数，它返回`undefined`。但是，如果写了`new`，它就变成了一个构造函数，它绑定的`this`指向新创建的对象，并默认返回`this`，也就是说，不需要在最后写`return this;`。	

如果我们通过`new Student()`创建了很多对象，这些对象的`hello`函数实际上只需要共享同一个函数就可以了，这样可以节省很多内存。

要让创建的对象共享一个`hello`函数，根据对象的属性查找原则，我们只要把`hello`函数移动到`xiaoming`、`xiaohong`这些对象共同的原型上就可以了，也就是`Student.prototype`

```js
function Student(name) {
    this.name = name;
}

Student.prototype.hello = function () {
    alert('Hello, ' + this.name + '!');
};
```

新的关键字`class`从ES6开始正式被引入到JavaScript中。`class`的目的就是让定义类更简单。

我们先回顾用函数实现`Student`的方法：

```
function Student(name) {
    this.name = name;
}

Student.prototype.hello = function () {
    alert('Hello, ' + this.name + '!');
}
```

如果用新的`class`关键字来编写`Student`，可以这样写：

```
class Student {
    constructor(name) {
        this.name = name;
    }

    hello() {
        alert('Hello, ' + this.name + '!');
    }
}
```

比较一下就可以发现，`class`的定义包含了构造函数`constructor`和定义在原型对象上的函数`hello()`（注意没有`function`关键字），这样就避免了`Student.prototype.hello = function () {...}`这样分散的代码。

最后，创建一个`Student`对象代码和前面章节完全一样：

```
var xiaoming = new Student('小明');
xiaoming.hello();
```

#### class继承

用`class`定义对象的另一个巨大的好处是继承更方便了。想一想我们从`Student`派生一个`PrimaryStudent`需要编写的代码量。现在，原型继承的中间对象，原型对象的构造函数等等都不需要考虑了，直接通过`extends`来实现：

```js
class PrimaryStudent extends Student {
    constructor(name, grade) {
        super(name); // 记得用super调用父类的构造方法!
        this.grade = grade;
    }

    myGrade() {
        alert('I am at grade ' + this.grade);
    }
}

var test = PrimaryStudent(name,grade);
```

注意`PrimaryStudent`的定义也是class关键字实现的，而`extends`则表示原型链对象来自`Student`。子类的构造函数可能会与父类不太相同，例如，`PrimaryStudent`需要`name`和`grade`两个参数，并且需要通过`super(name)`来调用父类的构造函数，否则父类的`name`属性无法正常初始化。

`PrimaryStudent`已经自动获得了父类`Student`的`hello`方法，我们又在子类中定义了新的`myGrade`方法。

ES6引入的`class`和原有的JavaScript原型继承有什么区别呢？实际上它们没有任何区别，`class`的作用就是让JavaScript引擎去实现原来需要我们自己编写的原型链代码。简而言之，用`class`的好处就是极大地简化了原型链代码。
