##Modules  

当我在2007年组建 Mozilla Javascript 团队时，好笑的是当时典型的 Js 程序只有一行。  

两年后谷歌地图被推出，在这之前不久，Javascript 主要被用来做表单的验证，而且可以确定的是`<input onchange=>` 仍旧只有一行。  

一切都变了，Javascript 的发展速度让人咋舌，Js 社区开发出许多用于大规模应用的工具。你需要的最基础的功能之一便是模块系统。模块系统是一种帮你穿梭在众多文件和目录中 - 让你按需获取 - 且极具效率的方式。于是自然的，Javascript 有模块系统。实际上有多个。也有几个包管理器，根据层级依赖关系安装拷贝的工具。你可能会认为 ES6 带来的模块语法有点晚了。  

那么，今天会揭晓 ES6 是否在原有系统上加入些什么，而在此基础上是否又构造了什么未来特性及工具。首先，让我们深入研究下 ES6 的模块。  

###Module 的基础  

ES6 的模块是指一个包含 Js 的文件。没有 module 关键字；模块像一个脚本一样被读取。不过有两点不同。  

+ ES6 模块会自动开启严格模式，即便你没写 "use strict"。  

+ 你可以在模块内使用 import 跟 export。  

首先来看 export。默认所有在模块内的声明，对模块来说都是局部的。假如你想让模块的某些声明被其他模块使用，那就需要用到 export 特性。做到这点有几种方式，最简单的方式为在前面加上 export 关键字。  

```
// kittydar.js - Find the locations of all the cats in an image.
// (Heather Arthur wrote this library for real)
// (but she didn't use modules, because it was 2013)

export function detectCats(canvas, options) {
  var kittydar = new Kittydar(options);
  return kittydar.detectCats(canvas);
}

export class Kittydar {
  ... several methods doing image processing ...
}

// This helper function isn't exported.
function resizeCanvas() {
  ...
}
...
```  

你可以 export 任何顶层的 function，class，var，let 或者 const。  

你需要做的只是写一个 module! 你压根不需要把所有东西放入一个 [IIFE](https://en.wikipedia.org/wiki/Immediately-invoked_function_expression)(自动执行的函数体) 或者回调里。只需要声明你需要的东西。由于是模块，所有声明属于模块的范围，而不被其他模块所见。Export 让声明变为模块的公共 API。  

除 export 以外，其他代码跟从前一样，你可以使用类似 Object 跟 Array 等。如果模块在浏览器里执行，你还可以使用 document 跟 XMLHTTPRequest。  

在另一个文件，我们可以导入 detectCats() 函数：  

```
// demo.js - Kittydar demo program

import {detectCats} from "kittydar.js";

function go() {
    var canvas = document.getElementById("catpix");
    var cats = detectCats(canvas);
    drawRectangles(canvas, cats);
}
```  

想要导入模块的多个对象，可以这样写：

    import {detectCats, Kittydar} from "kittydar.js";  
    
当执行包含 import 声明语句的模块时，引用的模块会在这之前最早被导入，然后模块体会根据依赖关系做深度优先遍历，避免跳过任何被执行的代码导致的循环。  

以上便是模块最基础的概念。足够简单。  

###Export 清单  

与其一个个导出，不如列一个导出名字的清单，用括号包裹起来：  

```
export {detectCats, Kittydar};

// no `export` keyword required here
function detectCats(canvas, options) { ... }
class Kittydar { ... }
```
export 语句并非需要在文件的第一行；它可以在模块文件的任何最外层作用域里。可以有多个 export 清单，或者清单跟单个 export 混合的方式，只要别出现重复导出。  

###重命名导入导出  

有些时候导入的模块名称可能会跟别的名字冲突，这时，ES6 允许你在导入模块时对其重命名：  

```
// suburbia.js

// Both these modules export something named `flip`.
// To import them both, we must rename at least one.
import {flip as flipOmelet} from "eggs.js";
import {flip as flipHouse} from "real-estate.js";
...
```

导出模块同样也支持重命名。偶尔会出现一个模块拥有别名，对这种情况来说就非常方便：  

```
// unlicensed_nuclear_accelerator.js - media streaming without drm
// (not a real library, but maybe it should be)

function v1() { ... }
function v2() { ... }

export {
  v1 as streamV1,
  v2 as streamV2,
  v2 as streamLatestVersion
};
```

###默认导出  

新标准被设计成很容易跟已有 CommonJs 及 AMD 模块交互的。假设有一个 Node 项目，而你执行了 `npm install lodash`。你可以直接从 Lodash 导入模块：  

```
import {each, map} from "lodash";

each([3, 2, 1], x => console.log(x));
```

或许你早已习惯了 _.each 这种方式来写。或者你想看到 _ 作为一个函数出现，见这里 [that’s a useful thing to do in Lodash](https://lodash.com/docs#_)。  

你可以使用略微不同的语法：import 模块而不带花括号：  

    import _ from "lodash";  
    
这缩写类似于 `import { default as _ } from "lodash"`; 所有 CommonJs 及 AMD 模块在 ES6 里存在默认的 export，这根当你 require() 那个模块是同样的效果 - 这就是 exports 对象。  

ES6 模块被设计成可以导出多个东西，然而对于 CommonJs 模块，默认导出的是所有你能得到的。举个例子，写到这里时，据我所知最著名的 [color](https://github.com/Marak/colors.js) 仍不提供任何 ES6 的支持。但你可以把它正确的导入进来：  

```
// ES6 equivalent of `var colors = require("colors/safe");`
import colors from "colors/safe";
```  

如果你需要给自己的 ES6 模块设置默认导出值，非常容易。跟其他的导出方式一样，并无神奇之处，区别仅在于它被命名为“默认的”。你可以按以下语法来写：  

```
let myObject = {
  field1: value1,
  field2: value2
};
export {myObject as default};
```  

或者用缩写更好：  

```
export default {
  field1: value1,
  field2: value2
};
```  

关键字 export default 可以跟任何值：函数、类、对象字面量设置你自己命名的。  

###模块对象  

抱歉太长了。但 Javascript 并不特殊：出于某写原因，所有语言的模块系统都有一大堆独立琐碎又无趣的特性。幸运的是，只剩一个东西要讲。好吧，其实是两个。  

    import * as cows from "cows";  
    
当使用 import *，实际上导入的是模块命名空间对象。它的属性都属于模块的导出。所以当 “cows” 模块导出一个叫 moo() 的函数，这边导入 “cows”的话，你可以写：  cows.moo()。  

###聚集的模块  

有时候包的主模块并不仅仅是导入其他模块， 并以统一的方式导出它们。为了简化这类的代码，这个例子囊括了各种导入导出方式：  

```
// world-foods.js - good stuff from all over

// import "sri-lanka" and re-export some of its exports
export {Tea, Cinnamon} from "sri-lanka";

// import "equatorial-guinea" and re-export some of its exports
export {Coffee, Cocoa} from "equatorial-guinea";

// import "singapore" and export ALL of its exports
export * from "singapore";
```  

每个 export-from 语句类似于 import-from 语句后面跟了个 export。与真的导入不同的是，这并不会添加一条重复导出到作用域。所以如果你计划在 world-foods.js 里使用 Tea 模块，最好别用上面这种缩写方式，你会发现 Tea 根本不在 world-foods 里。  

如果 “singapore” 的导出与别的导出冲突，将得到一个错误，所以请谨慎使用 `export *`。  

终于讲完了语法！到了有意思的部分。  

###import 究竟做了什么？  

什么也没做，你相信么？  