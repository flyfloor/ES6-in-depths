##Arrow functions  

箭头在 Javascript 早起便已存在。第一个 Javascript 指南建议将 Javascript 代码用 scripts 标签包裹，写在 HTML 注释里。这样对于不支持 Js 的浏览器，Javascript 代码就不会被认作是 text 了：  

    <script language="javascript">
    <!--
      document.bgColor = "brown";  // red
    // -->
    </script>
    
老浏览器会看到不支持的 tag 和注释；只有支持的浏览器才能执行 Js 代码。  

为了支持这项老的把戏，浏览器的 Javascript 引擎会将 ` <!-- ` 开头当做单行注释。这项技术一直存在于 Javascript 语言中，直到今天，不仅仅在内联的 `<script>` 标签里。甚至在 NodeJs 里也是如此。  

因此，箭头开始的注释在 ES6 里第一次被标准化。然此箭头非彼箭头。  

箭头 --> 同时表示 单行注释。诡异的是，HTML 标签里 --> 之前的部分是注释，而在 Js 里，箭头后面的部分是注释。  

更奇怪的是，当且仅当箭头在行首时才算是注释。假如出现在别的地方， --> 是 Js 的运算符，表示“跳到” 运算符！  

    function countdown(n) {
      while (n --> 0)  // "n goes to zero"
        alert(n);
      blastoff();
    }  
    
[这段代码确实能运行](http://codepen.io/anon/pen/oXZaBY?editors=001)。循环从 n 到 0. 这并不是 ES6 的新特性， 然而熟悉的特性，带有一点误导的性质。你能觉察究竟发生了什么？通常的，答案[在这里](http://stackoverflow.com/questions/1642028/what-is-the-name-of-the-operator)。  

确实存在 小于或等于运算符，<=。你可能在代码里找到更多的箭头，然而停下来观察，会发现有个箭头没有用到。  

* <!-- `single-line comment`  
*  --> `“goes to” operator`    
* <=  `less than or equal to`  
* =>	???  

我们先来看看函数。    

###到处都存在函数表达式  

在你需要函数时，你可以在任何运行的代码中写下该函数体。  

例如，你要告诉浏览器，当用户点击某个按钮做某件事，代码如下：  

    $("#confetti-btn").click(  
    
jQuery 的 .click() 方法有一个参数：一个函数。毫无疑问，你可以这样写：  

    $("#confetti-btn").click(function (event) {
      playTrumpet();
      fireConfettiCannon();
    });  
    
这段代码太稀松平常了。所以在 Javascript 流行前，如果这样写是很奇怪的，毕竟其他语言并不支持该特性。当然1958年， Lisp 拥有了类似的函数表达式，也成为 lambda 函数。然而类似 C++，Python， c# 等语言里 许多年都没有这样的语法。  

现在这四种语言都有了 lambda。新语言通常都会内建 lambda 语法。 Javascript 应该感谢 -- 那些构建库时毫无畏惧的大量使用 lambda的早期 Js 程序员，是他们是的这项特性风靡全球。  

稍显遗憾的，上述语言中，Js 的 lambda 是最啰嗦的。 

    // A very simple function in six languages.
    function (a) { return a > 0; } // JS
    [](int a) { return a > 0; }  // C++
    (lambda (a) (> a 0))  ;; Lisp
    lambda a: a > 0  # Python
    a => a > 0  // C#
    a -> a > 0  // Java  
    
###你箭袋里的“新箭”  

ES6 提供了编写函数的新语法。  

    // ES5
    var selected = allJobs.filter(function (job) {
      return job.isSelected();
    });

    // ES6
    var selected = allJobs.filter(job => job.isSelected());  
    
当你需要单个参数的函数时，arrow function 的语法是 `Identifier => Expression`。这样就省去写 function, return, 括号，花括号及分号。  

(我个人对此特性心存感激。对我来说，不用输入 function 非常重要，因为我经常输入`functoin`，而且需要不断地去纠正它。)  

含有多个参数的函数(或者不含任何参数，可变参数及默认参数，或者 destructuring 参数)你只需要将变量们用括号括起来。  

    // ES5
    var total = values.reduce(function (a, b) {
      return a + b;
    }, 0);

    // ES6
    var total = values.reduce((a, b) => a + b, 0);  

我认为它看起来很漂亮。  

arrow functions 与其他库的函数工具搭配的也非常好，例如 [Underscore.js](http://underscorejs.org/) 和 [Immutable](https://facebook.github.io/immutable-js/)。事实上， [Immutable’s documentation](https://facebook.github.io/immutable-js/docs/#/) 的例子完全是用 ES6 写的， 很多函数都是用 arrow function 写的。  

那些不怎么函数式的设置呢？arrow function 可以包含一段表达式代码块，而不仅仅是一句表达式。翻回前面的例子：  

    // ES5
    $("#confetti-btn").click(function (event) {
      playTrumpet();
      fireConfettiCannon();
    });  
    
ES6 里这么写：  

    // ES6
    $("#confetti-btn").click(event => {
      playTrumpet();
      fireConfettiCannon();
    });  
    
另外，使用 [Promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) 变得更戏剧性， `}).then(function (result) {` 这行代码也可以省掉。  

注意到包含代码块的 arrow function 并没有显式的返回值。所以需要一个 return 表达式。  

当使用 arrow function 创建 对象字面量，总是急着用圆括号将对象括起来，不然会出现 bug：  

    // create a new empty object for each puppy to play with
    var chewToys = puppies.map(puppy => {});   // BUG!
    var chewToys = puppies.map(puppy => ({})); // ok  
    
为什么呢？ 原因在于，不幸的是空对象 {} 跟 空的代码块 {} 看起来完全一样。ES6 将箭头紧跟遇到的 { 的代码当做代码块处理。 所以 `puppy => {}` 被翻译为未作任何操作的箭头函数，而最终返回 undefined。  

更易获得是，包含 {key: value} 的对象看起来像是包含了标签语句的代码块 -- 起码你的 Javascript 引擎是这么认为的。 幸运的是 { 是唯一会产生疑惑的字符，那么用圆括号将对象字面量包裹起来是唯一你需要记住的把戏。  

###别忘了`This`  

