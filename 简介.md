##深入 ES6  - 简介
>原文出自 [ES6 in depths](https://hacks.mozilla.org/2015/04/es6-in-depth-an-introduction/), 作者 [Jason Orendorff](https://blog.mozilla.org/jorendorff/)
  
padding  

###ECMAScript 的局限  

以W3C 为首的 ECMA 组织，长期致力于 Javascript 的标准化， 其定义的ECMAScript 目前包括以下内容:

+ 语言语法 - 解析规则， 关键字，表达式，定义，运算等。  
+ 类型 - boolean, number, string, object等。  
+ 原型继承  
+ 一些内建对象和函数的标准库 - JSON, Math, Array 方法, 对象自省方法等。  

除此之外，ECMAScript 并未定义任何与 HTML, CSS 相关的东西，或是 Web API, 例如 DOM, 这些实际上都存在于别的标准中。 ECMAScript 不仅涵盖了浏览器端 Javascript,  对非浏览器环境的 Javascript, 例如 nodejs, 也有很好的覆盖。  

###新标准  

上周，ECMAScript 规范的最终草案， 第六版，被提交到 Ecma General Assembly 做最后的 review, 这意味着:  
这个夏天，**我们将得到新标准的 Javascript 语言**(翻译至此时，ES6 已经正式发布)。  

这是个重大新闻，毕竟 Js 新标准不会每天都出。上个版本，ES5 已是2009年的事了。那时起，ES 标准制定委员会致力于 ES6 的设计，直至今日。  

ES6 是 Javascript 最重要的一次升级。你以往的 Js 代码仍旧可以工作， 因为它在设计上做了非常好的向下兼容。 事实上许多浏览器早已支持一部分的 ES6 特性，而且支持也越来越好。这意味着你的 JS 代码早已跑在实现部分 ES6 特性的浏览器上啦! 如果你并未发现任何兼容性问题，你可能永远不会遇到。  

###细数过往  

过去的 ECMAscript 标准有 1，2，3，5版本。  

怎么没有ES4？事实上这个版本还真有过，并且被倾注了很多心血，最终由于过于理想而难以实现。(例如，它曾经拥有极其复杂的，包含诸多术语及类型接口的静态类型系统)。  

ES4 是颇受争议的。标准委员会最终停止 ES4 的设计后，他们决定发布相对保守的 ES5, 并将重心放在一些重要新特性的设计上。这次妥协被称作“和谐”, 也是为了 ES5 规范里包含了下面两句话: 
> ECMAScript 是一个极其活跃的语言，关于语言本身的变革仍在继续。未来新版本将持续进行技术增强。  

好，诺言静待实现。  

###兑现诺言  

2009年的 ES5 引入了 Object.create(), Object.defineProperty(), getters 和 setters, 严格模式， JSON 对象等。我用过了所有这些特性，而且很喜欢 ES5 为这个语言所做的一切，但这些特性对于写 Js 代码很难起到戏剧性的影响。最重要的革新，在我看来，或许是 Array 的 .map(), .filter() 方法吧。  

ES6 则是截然不同的。它是经年之作，就像一座收藏品的宝藏，源源不断的新语言、库特性，是Js 最具影响力的升级。新特性跨度从广受欢迎的基础特性，例如 arrow functions 和 字符串修改， 到炸裂的例如 proxies、generators 等新概念。  

ES6 将改变你写 JS 的方式。  

本系列将详细介绍如何使用 ES6。  

首先第一章将从最经典的“缺失特性”，也是我期待了好多年的特性开讲。所以欢迎加入 ES6 迭代器和 for-of 循环 之旅。