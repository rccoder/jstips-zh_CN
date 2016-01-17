![header](https://raw.githubusercontent.com/loverajoel/jstips/master/resources/jstips-header-blog.gif)

# JavaScript 技巧介绍

新的一年，新的项目. **每天一个JS小技巧**

**《Introducing JavaScript Tips》** 中文翻译版，原地址：[https://github.com/loverajoel/jstips](https://github.com/loverajoel/jstips)

> With great excitement, I introduce these short and useful daily JavaScript tips that will allow you to improve your code writing. With less than 2 minutes each day, you will be able to read about performance, conventions, hacks, interview questions and all the items that the future of this awesome language holds for us.

怀着兴奋的心情，我将介绍一些简短有用的JavaScript日常小技巧，这可能会帮助你提高你的代码编写水平。伴随着每天不到两分钟的阅读，你将会了解JavaScript的性能、约定、Hacks、面试问题。

> At midday, no matter if it is a weekend or a holiday, a tip will be posted and tweeted.

在每个午休时刻，无论是周末还是假期，这样的一条条JavaScript小技巧将会发布。

# 技巧列表

## #15 - Even simpler way of using indexOf as a contains clause

> 2016-01-15 by [@jhogoforbroke](https://twitter.com/jhogoforbroke)

JavaScript by default does not have a contains method. And for checking existence of a substring in string or item in array you may do this:

``` javascript
var someText = 'javascript rules';
if (someText.indexOf('javascript') !== -1) {
}

// or
if (someText.indexOf('javascript') >= 0) {
}
```

But let's look at these [Expressjs](https://github.com/strongloop/express) code snippets.

[examples/mvc/lib/boot.js](https://github.com/strongloop/express/blob/2f8ac6726fa20ab5b4a05c112c886752868ac8ce/examples/mvc/lib/boot.js#L26)

``` javascript
for (var key in obj) {
  // "reserved" exports
  if (~['name', 'prefix', 'engine', 'before'].indexOf(key)) continue;
```

[lib/utils.js](https://github.com/strongloop/express/blob/2f8ac6726fa20ab5b4a05c112c886752868ac8ce/lib/utils.js#L93)

``` javascript
exports.normalizeType = function(type){
  return ~type.indexOf('/')
    ? acceptParams(type)
    : { value: mime.lookup(type), params: {} };
};
```

[examples/web-service/index.js](https://github.com/strongloop/express/blob/2f8ac6726fa20ab5b4a05c112c886752868ac8ce/examples/web-service/index.js#L35)

``` javascript
// key is invalid
if (!~apiKeys.indexOf(key)) return next(error(401, 'invalid api key'));
```

The gotcha is the [bitwise operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Bitwise_Operators) **~**, "Bitwise operators perform their operations on such binary representations, but they return standard JavaScript numerical values."

It transforms -1 into 0, and 0 is false in javascript, so:

``` javascript
var someText = 'text';
!!~someText.indexOf('tex'); //sometext contains text - true
!~someText.indexOf('tex'); //sometext not contains text - false
~someText.indexOf('asd'); //sometext contains asd - false
~someText.indexOf('ext'); //sometext contains ext - true
```

### String.prototype.includes()

In ES6 was introduced the [includes() method](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/includes) and you can use to determine whether or not a string includes another string:

``` javascript
'something'.includes('thing'); // true
```

With ECMAScript 2016 (ES7) is even possible uses with Arrays, like indexOf:

``` javascript
!!~[1, 2, 3].indexOf(1); // true
[1, 2, 3].includes(1); // true
```

**Unfortunately, It's got support only in Chrome, Firefox, Safari 9 or above and Edge. Not IE11 or less.**

**It's better to using in controlled environments.**

## #14 - Fat Arrow Functions #ES6

> 2016-01-13 by [@pklinger](https://github.com/pklinger/)

Introduced as a new feature in ES6, fat arrow functions may come as a handy tool to write more code in less lines. The name comes from its syntax as `=>` is a 'fat arrow' compared to a thin arrow `->`. Some programmers might already know this type of functions from different languages such as Haskell as 'lambda expressions' respectively 'anonymous functions'. It is called anonymous, as these arrow functions do not have a descriptive function name.

### What are the benefits?

* Syntax: less LOC; no more typing `function` keyword over and over again
* Semantics: capturing the keyword `this` from the surrounding context

### Simple syntax example

Have a look at these two code snippets, which exactly do the same job. You will quickly understand what fat arrow functions do.

``` javascript
// general syntax for fat arrow functions
param => expression

// may also be written with parentheses
// parentheses are required on multiple params
(param1 [, param2]) => expression


// using functions
var arr = [5,3,2,9,1];
var arrFunc = arr.map(function(x) {
  return x * x;
});
console.log(arr)

// using fat arrow
var arr = [5,3,2,9,1];
var arrFunc = arr.map((x) => x*x);
console.log(arr)
```

As you may see, the fat arrow function in this case may save you time typing out the parentheses as well as the function and return keywords. I would advice you to always write parentheses around the parameter inputs as the parentheses will be needed for multiple input parameters such as in `(x,y) => x+y` anyways. It is just a way to cope with forgetting them in different use cases. But the code above would also work like this: `x => x*x`. So far these are only syntactical improvements, which lead to less LOC and better readability. 

### Lexically binding `this`

There is another good reason to use fat arrow functions. There is the issue with the context of `this`. With arrow functions, you will not worry about `.bind(this)` or setting `that = this` anymore, as fat arrow functions pick the context of `this` from the lexical surrounding. Have a look at the next [example] (https://jsfiddle.net/pklinger/rw94oc11/):

``` javascript

// globally defined this.i
this.i = 100;

var counterA = new CounterA();
var counterB = new CounterB();
var counterC = new CounterC();
var counterD = new CounterD();

// bad example 
function CounterA() {
  // CounterA's `this` instance (!! gets ignored here)
  this.i = 0;
  setInterval(function () {
    // `this` refers to global object, not to CounterA's `this`
    // therefore starts counting with 100, not with 0 (local this.i)
    this.i++;
    document.getElementById("counterA").innerHTML = this.i;
  }, 500);
}

// manually binding that = this
function CounterB() {
  this.i = 0;
  var that = this;
  setInterval(function() {
    that.i++;
    document.getElementById("counterB").innerHTML = that.i;
  }, 500);
}

// using .bind(this)
function CounterC() {
  this.i = 0;
  setInterval(function() {
    this.i++;
    document.getElementById("counterC").innerHTML = this.i;
  }.bind(this), 500);
}

// fat arrow function
function CounterD() {
  this.i = 0;
  setInterval(() => {
    this.i++;
    document.getElementById("counterD").innerHTML = this.i;
  }, 500);
}
```

Further information about fat arrow functions may be found at [MDN] (https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions). To see different syntax options visit [this site] (http://jsrocks.org/2014/10/arrow-functions-and-their-scope/).



## #13 - Tip to measure performance of a javascript block

2016-01-13 by [@manmadareddy](https://twitter.com/manmadareddy)

For quickly measuring performance of a javascript block, we can use the console functions like

[```console.time(label)```](https://developer.chrome.com/devtools/docs/console-api#consoletimelabel) and [```console.timeEnd(label)```](https://developer.chrome.com/devtools/docs/console-api#consoletimeendlabel)

``` javascript
console.time("Array initialize");
var arr = new Array(100),
    len = arr.length,
    i;

for (i = 0; i < len; i++) {
    arr[i] = new Object();
};
console.timeEnd("Array initialize"); // Outputs: Array initialize: 0.711ms
```

More info:

[Console object](https://github.com/DeveloperToolsWG/console-object),

[Javascript benchmarking](https://mathiasbynens.be/notes/javascript-benchmarking)

Demo: [jsfiddle](https://jsfiddle.net/meottb62/) - [codepen](http://codepen.io/anon/pen/JGJPoa) (outputs in browser console)

## #12 - Pseudomentatory parameters in ES6 functions #ES6

> 2016-01-12 by [Avraam Mavridis](https://github.com/AvraamMavridis)



In many programming languages the parameters of a function is by default mandatory and the developer has to explicitly define that a parameter is optional. In Javascript every parameter is optional, but we can enforce this behavior without messing the actual body of a function taking advantage of the [**es6's default values for parameters**] (http://exploringjs.com/es6/ch_parameter-handling.html#sec_parameter-default-values) feature.

``` javascript
const _err = function( message ){
  throw new Error( message );
}

const getSum = (a = _err('a is not defined'), b = _err('b is not defined')) => a + b

getSum( 10 ) // throws Error, b is not defined
getSum( undefined, 10 ) // throws Error, a is not defined
 ```

 `_err` is a function that immediately throws an Error. If no value is passed for one of the parameters, the default value is gonna be used, `_err` will be called and an Error will be throwed. You can see more examples for the **default parameters feature** on [Mozilla's Developer Network ](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Functions/default_parameters)

## #11 - Hoisting
> 2016-01-11 by [@squizzleflip](https://twitter.com/squizzleflip)

Understanding [hoisting](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/var#var_hoisting) will help you organize your function scope. Just remember, variable declaration and function definition are hoisted to the top. Variable definition is not, even if you declare and define a variable on the same line. Also, variable **declaration** is letting the system know that the variable exists while **definition** is assigning it a value.

​```javascript
function doTheThing() {
  // ReferenceError: notDeclared is not defined
  console.log(notDeclared);

  // Outputs: undefined
  console.log(definedLater);
  var definedLater;

  definedLater = 'I am defined!'
  // Outputs: 'I am defined!'
  console.log(definedLater)

  // Outputs: undefined
  console.log(definedSimulateneously);
  var definedSimulateneously = 'I am defined!'
  // Outputs: 'I am defined!'
  console.log(definedSimulateneously)

  // Outputs: 'I did it!'
  doSomethingElse();

  function doSomethingElse(){
    console.log('I did it!');
  }

  // TypeError: undefined is not a function
  functionVar();

  var functionVar = function(){
    console.log('I did it!');
  }
}
```

To make things easier to read, declare all of your variables at the top of your function scope so it is clear which scope the variables are coming from. Define your variables before you need to use them. Define your functions at the bottom of your scope to keep them out of your way.

## #10 - Check if a property is in a Object

> 2016-01-10 by [@loverajoel](https://www.twitter.com/loverajoel)

When you have to check if a property is present of an [object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Working_with_Objects), you probably are doing something like this:

``` javascript
var myObject = {
  name: '@tips_js'
};

if (myObject.name) { ... }

```

Thats ok, but you have to know that there are two native ways for this kind of thing, the [`in` operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/in) and [`Object.hasOwnProperty`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/hasOwnProperty), every object descended from `Object`, has available both ways.

### See the big Difference

``` javascript
var myObject = {
  name: '@tips_js'
};

myObject.hasOwnProperty('name'); // true
'name' in myObject; // true

myObject.hasOwnProperty('valueOf'); // false, valueOf is inherited from the prototype chain
'valueOf' in myObject; // true

```

Both differs in the depth how check the properties, in other words `hasOwnProperty` will only return true if key is available on that object directly, however `in` operator doesn't discriminate between properties created on an object and properties inherited from the prototype chain.

Here another example

``` javascript
var myFunc = function() {
  this.name = '@tips_js';
};
myFunc.prototype.age = '10 days';

var user = new myFunc();

user.hasOwnProperty('name'); // true
user.hasOwnProperty('age'); // false, because age is from the prototype chain
```

Check here the [live examples](https://jsbin.com/tecoqa/edit?js,console)!

Also recommends read [this discussion](https://github.com/loverajoel/jstips/issues/62) about common mistakes at checking properties' existence in objects

## #09 - Template Strings

> 2016-01-09 by [@JakeRawr](https://github.com/JakeRawr)

As of ES6, JS now has template strings as an alternative to the classic end quotes strings.

Ex:

Normal string

``` javascript
var firstName = 'Jake';
var lastName = 'Rawr';
console.log('My name is ' + firstName + ' ' + lastName);
// My name is Jake Rawr
```

Template String

``` javascript
var firstName = 'Jake';
var lastName = 'Rawr';
console.log(`My name is ${firstName} ${lastName}`);
// My name is Jake Rawr
```

You can do Multi-line strings without `\n` and simple logic (ie 2+3) inside `${}` in Template String.

You are also able to to modify the output of template strings using a function; they are called [Tagged template strings]

(https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/template_strings#Tagged_template_strings) for example usages of tagged template strings.

You may also want to [read](https://hacks.mozilla.org/2015/05/es6-in-depth-template-strings-2) to understand template strings more

## #08 - 将节点列表转化为数组 (Converting a Node List to an Array)

> 2016-01-08 by [@Tevko](https://twitter.com/tevko)


> The `querySelectorAll` method returns an array-like object called a node list. These data structures are referred to as "Array-like", because they appear as an array, but can not be used with array methods like `map` and `foreach`. Here's a quick, safe, and reusable way to convert a node list into an Array of DOM elements:

`querySelectorAll` 方法返回了一个类似数组的对象—— 节点列表(node list)。这个数据结构看起来是 ”类数组“的。因为他们是以一个数组的形式显示的，但是并不是使用数组的一些方法，比如 `map` 和 `foreach`。 这儿有一个快速，安全，可重读使用的方法去把一个节点列表转化为 DOM 元素的数组。

``` javascript
const nodelist = document.querySelectorAll('div');
const nodelistToArray = Array.apply(null, nodelist);

//later on ..

nodelistToArray.forEach(...);
nodelistToArray.map(...);
nodelistToArray.slice(...);

//etc...
```

> The `apply` method is used to pass an array of arguments to a function with a given `this` value. [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/apply) states that `apply` will take an array like object, which is exactly what `querySelectorAll` returns. Since we don't need to specify a value for `this` in the context of the function, we pass in `null` or `0`. The result is an actual array of DOM elements which contains all of the available array methods.

`apply` 方法被用来将数组参数传递给一个给定 `this` 的函数。 MDN 规定，`apply` 将得到一个类似对象的数组，他正是 `querySelectorAll` 返回的东西。因为我们不需要在函数的上下文中指定 `this` 这个值，所以我们用 `null` 或者 `0` 。其结果是一个包含所有可用的数组方法的 DOM 元素的数组。

> Or if you are using ES2015 you can use the [spread operator `...`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_operator)

如果你使用 ES2015，你可以使用 `...` 这个操作符。

``` js
const nodelist = [...document.querySelectorAll('div')]; // returns a real Array

//later on ..

nodelist.forEach(...);
nodelist.map(...);
nodelist.slice(...);

//etc...
```

## #07 - "use strict" 和 惰性 ("use strict" and get lazy)

> 2016-01-07 by [@nainslie](https://twitter.com/nat5an)


> Strict-mode JavaScript makes it easier for the developer to write "secure" JavaScript.

严格模式让开发者能够更加容易的写出 “安全” 的 JavaScript 代码。

> By default, JavaScript allows the programmer to be pretty careless, for example, by not requiring us to declare our variables with "var" when we first introduce them.  While this may seem like a convenience to the unseasoned developer, it's also the source of many errors when a variable name is misspelled or accidentally referred to out of its scope.

默认的情况下，JavaScript 允许程序员非常的不小心，举个例子，第一次使用的时候不需要使用 `var` 来申明一个变量。虽然这看起来方便了经验丰富的开发，但是事实上，当一个变量名拼写错误或者意外地超出了他的作用域也是很多错误的源头。

> Programmers like to make the computer do the boring stuff for us, and automatically check our work for mistakes. That's what the JavaScript "use strict" directive allows us to do, by turning our mistakes into JavaScript errors.

程序员们喜欢让电脑去代替他们做一些无聊的事，并且去自动的检查我们工作中的失误。这就是 JavaScript 中的严格模式指令允许我们去做的事，通过他可以将我们的失误转化为 JavaScript 中的错误。

> We add this directive either by adding it at the top of a js file:

我们在文件的顶部添加这样的指令

``` javascript
// Whole-script strict mode syntax
"use strict";
var v = "Hi!  I'm a strict mode script!";
```

> or inside a function:

或者在函数里面:

``` javascript
function f()
{
  // Function-level strict mode syntax
  'use strict';
  function nested() { return "And so am I!"; }
  return "Hi!  I'm a strict mode function!  " + nested();
}
function f2() { return "I'm not strict."; }
```

> By including this directive in a JavaScript file or function, we will direct the JavaScript engine to execute in strict mode which disables a bunch of behaviors that are usually undesirable in larger JavaScript projects.  Among other things, strict mode changes the following behaviors:

在 JavaScript 的文件或函数中加入这个指令，我们能直接让 JavaScript 的引擎在严格模式下禁用一系列的在大型项目中不可取的行为。除此之外，严格模式改变了以下的行为：

* > Variables can only be introduced when they are preceded with "var"
  
  变量引入的时间前面必须有 `var` 。
  
* > Attempting to write to readonly properties generates a noisy error
  
  试图写只读属性的时候会产生错误 。
  
* > You have to call constructors with the "new" keyword
  
  你必须用  `new` 关键词来调用构造函数。
  
* > "this" is not implicitly bound to the global object
  
  `this` 不绑定全局对象。
  
* > Very limited use of eval() allowed
  
  限制使用 `eval()` 。
  
* > Protects you from using reserved words or future reserved words as variable names
  
  阻止你使用保留字或者将来会用作保留字的单词作为变量名。

> Strict mode is great for new projects, but can be challenging to introduce into older projects that don't already use it in most places.  It also can be problematic if your build chain concatenates all your js files into one big file, as this may cause all files to execute in strict mode.

在一个新的项目中使用严格模式是非常友好的，但是在大多数地方引入旧项目往往是有些困难的。 如果你编译所有的 js 文件到一个大文件中极有可能是存在问题的，应为这很可能导致所有的文件都在严格模式下执行。

> It is not a statement, but a literal expression, ignored by earlier versions of JavaScript.

这不是一个申明，是一个文字的表示式，在早期版本的 JavaScript 中会被忽略。

> Strict mode is supported in:

严格模式在以下的环境中被支持：

* > Internet Explorer from version 10.
  
  10 以后的IE
  
* > Firefox from version 4.
  
  4 以后的 FireFox
  
* > Chrome from version 13.
  
  13 以后的 Chrome
  
* > Safari from version 5.1.
  
  5.1 以后的Safari
  
* > Opera from version 12.
  
  12 以后的Opera

[See MDN for a fuller description of strict mode](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode).

## #06 - 同时为一个数组或者元素写一个简单的方法(Writing a single method for arrays or single elements)

> 2016-01-06 by [@mattfxyz](https://twitter.com/mattfxyz)


> Rather than writing separate methods to handle an array and a single element parameter, write your functions so they can handle both. This is similar to how some of jQuery's functions work (`css` will modify everything matched by the selector).

不像编写一个单独的函数去处理数组或者单一元素，去尝试写一个能同时处理他们的方法。这和 `jQuery` 中的函数是如何工作是类似的。（`css` 将修改所有和他选择器匹配的东西）

> You just have to concat everything into an array first. `Array.concat` will accept an array or a single element.

你不得不去首先合并数组中的一切东西。 `Array.concat` 将会接收一个数组或者单一元素。

``` javascript
function printUpperCase(words) {
  var elements = [].concat(words);
  for (var i = 0; i < elements.length; i++) {
    console.log(elements[i].toUpperCase());
  }
}
```

> `printUpperCase` is now ready to accept a single node or an array of nodes as its parameter.

`printUpperCase` 现在接收一个单一的元素或者数组中的元素作为他的参数。

``` javascript
printUpperCase("cactus");
// => CACTUS
printUpperCase(["cactus", "bear", "potato"]);
// => CACTUS
//  BEAR
//  POTATO
```

## #05 - `undefined` 与 `null` 之间的不同之处(Differences between `undefined` and `null`)

> 2016-01-05 by [@loverajoel](https://twitter.com/loverajoel)

- > `undefined` means a variable has not been declared, or has been declared but has not yet been assigned a value
  
  `undefined` 是指一个变量没有被定义，或者是已经定义了但是还没有被赋值。
  
- > `null` is an assignment value that means "no value"
  
  `null` 是指一个赋值了的变量，给他赋的值是 “没有值”。
  
- > Javascript sets unassigned variables with a default value of `undefined`
  
  JavaScript给没有赋值的变量分配的默认值是`undefined`。
  
- > Javascript never sets a value to `null`. It is used by programmers to indicate that a `var` has no value.
  
  JavaScript 从来没有给任何一个变量赋值`null`。 他用于由程序员申明一个变量没有值。
  
- > `undefined` is not valid in JSON while `null` is
  
  `undefined` 在 JSON 中是不符合规范的，但 `null` 是可以的。
  
- > `undefined` typeof is `undefined`
  
  `undefined` 的类型就是`undefined`。
  
- > `null` typeof is an `object`
  
  `null` 的类型是 `object`。
  
- > Both are primitives
  
  他们两个都是基本类型
  
- > Both are [falsy](https://developer.mozilla.org/en-US/docs/Glossary/Falsy)
  
  他们两个转化为布尔后都是false。
  
  (`Boolean(undefined) // false`, `Boolean(null) // false`)
  
- > You can know if a variable is [undefined](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/undefined)
  
  假如一个变量是 `undefined` 的话你是可以知道他的。
  
  ``` javascript
  typeof variable === "undefined"
  ```
  
  > You can check if a variable is [null](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/null)
  
  你是可以检测一个变量是不是 `null` 。
  
  ``` javascript
  variable === null
  ```
  
- > The **equality** operator considers them equal, but the **identity** doesn't
  
  等于运算符 `==` 认为他们是相等的，但是身份运算符 `===` 认为他们不是相等的。
  
  ``` javascript
  null == undefined // true
  
  null === undefined // false
  ```

## #04 - 带重音的字符串排序(Sorting strings with accented characters)

> 2016-01-04 by [@loverajoel](https://twitter.com/loverajoel)



> Javascript has a native method **[sort](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/sort)** that allows sorting arrays. Doing a simple `array.sort()` will treat each array entry as a string and sort it alphabetically. Also you can provide your [own custom sorting](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/sort#Parameters) function.

JavaScript 拥有一个原生的 `sort` 排序方法开排序数组。使用简单的 `array.sort()` 会把数组中的每项当做一个字符串，然后按照字母的顺序进行排序。当然，你也可以使用你自己定义的排序函数。

``` javascript
['Shanghai', 'New York', 'Mumbai', 'Buenos Aires'].sort();
// ["Buenos Aires", "Mumbai", "New York", "Shanghai"]
```

> But when you try order an array of non ASCII characters like this `['é', 'a', 'ú', 'c']`, you will obtain a strange result `['c', 'e', 'á', 'ú']`. That happens because sort works only with english language.

但是当你排序不是ASCII字符的数组时，比如这样的 `['é', 'a', 'ú', 'c']` ， 你将会得到这样一个奇怪的结果 `['c', 'e', 'á', 'ú']`。这是因为排序仅仅适用于英语。

> See the next example:

看下面的例子：

``` javascript
// Spanish
['único','árbol', 'cosas', 'fútbol'].sort();
// ["cosas", "fútbol", "árbol", "único"] // bad order

// German
['Woche', 'wöchentlich', 'wäre', 'Wann'].sort();
// ["Wann", "Woche", "wäre", "wöchentlich"] // bad order
```

> Fortunately, there are two ways to overcome this behavior [localeCompare](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/localeCompare) and [Intl.Collator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Collator) provided by ECMAScript Internationalization API.

 幸运的是，这儿有两种由 ECMAScript 国际化 API 提供的方式可以克服这种行为——`localeCompare` 和 `Intl.Collator`。

> Both methods have their own custom parameters in order to configure it to work adequately.



> 这两种方法都有可以自定义的参数去配置他来使得他充分工作。

### 使用 `localeCompare()`(Using `localeCompare())`

``` javascript
['único','árbol', 'cosas', 'fútbol'].sort(function (a, b) {
  return a.localeCompare(b);
});
// ["árbol", "cosas", "fútbol", "único"]

['Woche', 'wöchentlich', 'wäre', 'Wann'].sort(function (a, b) {
  return a.localeCompare(b);
});
// ["Wann", "wäre", "Woche", "wöchentlich"]
```

### 使用`Intl.Collator()`(Using `Intl.Collator()`)

``` javascript
['único','árbol', 'cosas', 'fútbol'].sort(Intl.Collator().compare);
// ["árbol", "cosas", "fútbol", "único"]

['Woche', 'wöchentlich', 'wäre', 'Wann'].sort(Intl.Collator().compare);
// ["Wann", "wäre", "Woche", "wöchentlich"]
```

- > For each method you can customize the location
  
  每种方法你都可以自定义位置
  
- > According to [Firefox](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/localeCompare#Performance) Intl.Collator is faster when comparing large numbers of strings.
  
  当比较两个比较大的字符串的时候`Intl.Collator`是更快的。

> So when you are working with arrays of strings in a language other than English, remember to use this method to avoid unexpected sorting.

所以当你处理的数组中的字符串不是英文的时候，记得去使用这种方法来避免产生不期待的排序。

## #03 - 改善嵌套条件(Improve Nested Conditionals)

> 2016-01-03 by [AlbertoFuente](https://github.com/AlbertoFuente)



> How can we improve and make more efficient nested `if` statement in javascript.

我们如何在 `JavaScript` 中去改善，或者是使我们的`if`嵌套更加有效。

``` javascript
if (color) {
  if (color === 'black') {
    printBlackBackground();
  } else if (color === 'red') {
    printRedBackground();
  } else if (color === 'blue') {
    printBlueBackground();
  } else if (color === 'green') {
    printGreenBackground();
  } else {
    printYellowBackground();
  }
}
```

> One way to improve the nested `if` statement would be using the `switch` statement. Although it is less verbose and is more ordered, It's not recommended to use it because it's so difficult to debug errors, here's [why](https://toddmotto.com/deprecating-the-switch-statement-for-object-literals/).

一种去改善`if`嵌套的方法可能是使用`switch`语句。虽然这样做的确看起来更加的有序和简洁，但是并不建议去使用它，因为使用它是如此的难以调试错误，原因在这儿[why](https://toddmotto.com/deprecating-the-switch-statement-for-object-literals/)。

``` javascript
switch(color) {
  case 'black':
    printBlackBackground();
    break;
  case 'red':
    printRedBackground();
    break;
  case 'blue':
    printBlueBackground();
    break;
  case 'green':
    printGreenBackground();
    break;
  default:
    printYellowBackground();
}
```

> But what if we have a conditional with several checks in each statement? In this case, if we like to do less verbose and more ordered, we can use the conditional `switch`. If we pass `true` as parameter to the `switch` statement, It allows us to put a conditional in each case.

但是如果我们有有一个条件，然后在每个语句进行多项的检查。在这种情况下，如果我们喜欢让嵌套变得更加有序和简洁，我们可以使用这种条件`switch`。如果我们传递 `true` 参数给`switch` 语句，他将允许我们在每一种情况下都有条件。



``` javascript
switch(true) {
  case (typeof color === 'string' && color === 'black'):
    printBlackBackground();
    break;
  case (typeof color === 'string' && color === 'red'):
    printRedBackground();
    break;
  case (typeof color === 'string' && color === 'blue'):
    printBlueBackground();
    break;
  case (typeof color === 'string' && color === 'green'):
    printGreenBackground();
    break;
  case (typeof color === 'string' && color === 'yellow'):
    printYellowBackground();
    break;
}
```

> But we must always avoid having several checks in every condition, avoiding use of `switch` as far as possible and take into account that the most efficient way to do this is through an `object`.

但是我们必须避免在每个条件下都有多次的检查，尽可能的去避免使用`switch`，解决这个问题最有效的一个方法是通过对象。

``` javascript
var colorObj = {
  'black': printBlackBackground,
  'red': printRedBackground,
  'blue': printBlueBackground,
  'green': printGreenBackground,
  'yellow': printYellowBackground
};

if (color && colorObj.hasOwnProperty(color)) {
  colorObj[color]();
}
```

> Here you can find more information about [this](http://www.nicoespeon.com/en/2015/01/oop-revisited-switch-in-js/).

在[这儿](http://www.nicoespeon.com/en/2015/01/oop-revisited-switch-in-js/)你将找到更多与这个相关的资料。

## #02 - ReactJs - Keys in children components are important

> 2016-01-02  by [@loverajoel](https://twitter.com/loverajoel)



The [key](https://facebook.github.io/react/docs/multiple-components.html#dynamic-children) is an attribute that you must pass to all components created dynamically from an array. It's unique and constant id that React use for identify each component in the DOM and know that it's a different component and not the same one. Using keys will ensure that the child component is preserved and not recreated and prevent that weird things happens.

> Key is not really about performance, it's more about identity (which in turn leads to better performance). randomly assigned and changing values are not identity [Paul O’Shannessy](https://github.com/facebook/react/issues/1342#issuecomment-39230939)

- Use an existing unique value of the object.
- Define the keys in the parent components, not in child components

``` 
​```javascript
//bad
...
render() {
	<div key={{item.key}}>{{item.name}}</div>
}
...

//good
<MyComponent key={{item.key}}/>
​```
```

- [Using array index is a bad practice.](https://medium.com/@robinpokorny/index-as-a-key-is-an-anti-pattern-e0349aece318#.76co046o9)
- `random()` will not work

``` 
​```javascript
//bad
<MyComponent key={{Math.random()}}/>
​```
```

- You can create your own unique id, be sure that the method be fast and attach it to your object.
- When the amount of child are big or involve expensive components, use keys has performance improvements.
- [You must provide the key attribute for all children of ReactCSSTransitionGroup.](http://docs.reactjs-china.com/react/docs/animation.html)

## #1 - Angular之$digest与$apply对决(AngularJs: `$digest` vs `$apply`)

> 2016-01-01  by [@loverajoel](https://twitter.com/loverajoel)



> One of the most appreciated features of AngularJs is the two way data binding. In order to make this work AngularJs evaluates the changes between the model and the view through cycles(`$digest`). You need to understand this concept in order to understand how the framework works under the hood.

AngularJs 中最让人欣赏的莫过于数据的双向绑定。AngularJs 在模型和视图之间通过循环视图使得这一机制实现。如果你想去理解框架是如何去工作的话你需要了解这一概念。

> Angular evaluates each watcher whenever one event is fired, this is the known `$digest` cycle.Sometimes you have to force to run a new cycle manually and you must choose the correct option because this phase is one of the most influential in terms of performance.

Angular 中每当一个事件被触发，就形成一个已知的`$digest`  cycle。有时候你不得不去强制手动创建一个新的周期，这个时候你必须有正确的选择，因为这个阶段对性能的影响是非常之大的。

### `$apply`

> This core method lets you to start the digestion cycle explicitly, that means that all watchers are checked, the entire application starts the `$digest loop`. Internally after execute an optional function parameter, call internally to `$rootScope.$digest();`.

这个核心的方法让你去明确的开始`$digest`  cycle，这就意味着所有的watchers都会重新进行检查，整个应用启动`$digest loop`。 经过内部执行可选的函数参数，调用内部的`$rootScope.$digest();`。

### `$digest`

> In this case the `$digest` method starts the `$digest` cycle for the current scope and its children. You should notice that the parents scopes will not be checked and not be affected.

在这样的情况下，`$digest`方法从当前的作用域以及子作用域中启动 `$digest`  cycle。你应该能注意到父作用域将不会进行检查，当然也就没有影响。

### 建议(Recommendations)

- > Use `$apply` or `$digest` only when browser DOM events have triggered outside of AngularJS.
  
  仅仅当浏览器的 DOM 事件在 AngularJs 作用的外面时使用 `$apply` 或者 `$digest`。
  
- > Pass a function expression to `$apply`, this have a error handling mechanism and allow integrate changes in the digest cycle
  
  给 `$apply` 传递一个函数表达式的时候，有一个错误处理机制，这个机制允许集成 digest cycle 的变化。

``` 
​```javascript
$scope.$apply(() => {
	$scope.tip = 'Javascript Tip';
});
​```
```

- > If only needs update the current scope or its children use `$digest`, and prevent a new digest cycle for the whole application. The performance benefit it's self evident
  
  仅仅去更新现在的作用域或者子作用域的时候使用 `$digest` ，阻止整个应用产生新的 digest cycle 。这在性能上的好处是不言而喻的。
  
- > `$apply()` is hard process for the machine and can lead to performance issues when having a lot of binding.
  
  `$apply()` 是一个很难用的方法，当许多的绑定放在一起的时候可能会在性能上产生问题。
  
- > If you are using >AngularJS 1.2.X, use `$evalAsync` is a core method that will evaluate the expression during the current cycle or the next. This can improve your application's performance.
  
  如果你使用的 AngularJS 版本是大于 1.2.X的，使用核心方法`$evalAsync`在当前的周期或者下一个的时候。这将提升你的应用性能。

## #0 - 在数组中插入一个元素(Insert item inside an Array)

> 2015-12-29



> Inserting an item into an existing array is a daily common task. You can add elements to the end of an array using push, to the beginning using unshift, or the middle using splice.

在一个已经存在的数组中插入一个元素是我们每天都要做的事情，你可以用`push`操作把这个元素插入到数组的最后面，使用`unshift`操作把它插入到数据的最前面，或者使用`splice	`把它插入到数组的中间。

> But those are known methods, doesn't mean there isn't a more performant way, here we go...

这些都是我们熟知的方法，但是这并不意味这没有一个性能更好的方法， 现在，跟着我一起去看看...

> Adding an element at the end of the array is easy with push(), but there is a way more performant.

用`push`	方法能把一个元素非常方便的插入到数据的最后面，但是这个有一种性能上更好的方法去达到同样的目的。

``` javascript
var arr = [1,2,3,4,5];

arr.push(6);
arr[arr.length] = 6; // 这种方法在 OS X 10.11.1 的 Chrome 47.0.2526.106 上要比直接push要快43%（43% faster in Chrome 47.0.2526.106 on Mac OS X 10.11.1）
```

> Both methods modify the original array. Don't believe me? Check the [jsperf](http://jsperf.com/push-item-inside-an-array)

去试试利用这两种方法修改数据。不相信我，看看这个[jsperf](http://jsperf.com/push-item-inside-an-array)

> Now we are trying to add an item to the beginning of the array

现在我们去尝试在数据的最前面加入一个元素

``` javascript
var arr = [1,2,3,4,5];

arr.unshift(0);
[0].concat(arr); // 这种方法在 OS X 10.11.1 的 Chrome 47.0.2526.106 上要比unshift快98% （faster in Chrome 47.0.2526.106 on Mac OS X 10.11.1）
```

> Here is a little bit detail, unshift edits the original array, concat returns a new array. [jsperf](http://jsperf.com/unshift-item-inside-an-array)

这儿有一个小小的细节，`unshift`修改的是原始的数据，`concat`返回的是一个新的数组。[jsperf](http://jsperf.com/unshift-item-inside-an-array)

> Adding items at the middle of an array is easy with splice and is the most performant way to do it.

用`splice`去把一个元素插入到数组的中间不仅仅是最简单的方法，同样也是性能上最好的方法。

``` javascript
var items = ['one', 'two', 'three', 'four'];
items.splice(items.length / 2, 0, 'hello');
```

> I tried to run these tests in various Browsers and OS and the results were similar. I hope these tips will be useful for you and encourage to perform your own tests!

我试着去在不同的浏览器和操作系统上测试上面所说的东西，结果都是一致的。我希望这个技巧对你是有用的，同时我也鼓励你去尝试他。

### License

[![CC0](http://i.creativecommons.org/p/zero/1.0/88x31.png)](http://creativecommons.org/publicdomain/zero/1.0/)