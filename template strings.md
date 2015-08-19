##template strings  

###"基础"  

ES6 引入了新的字面量字符语法，叫做 template strings。它看起来像是传统的字符串，区别在于用 **`** 取代了**'** 或者 **"**。简单的例子: 

    context.fillText(`Ceci n'est pas une chaîne.`, x, y);  
    
然而被叫做 ```template strings```，而不是```冗长的除了引号不同的字符串```。原因在于 template strings 给 Javascript 带来了更简单的字符串修改方式。这种方式看起来很漂亮，很方便的修改字符串的值。  

使用 template strings 有很多种方式，我最喜欢下面这种:  

    function authorize(user, action) {
      if (!user.hasPrivilege(action)) {
        throw new Error(
          `User ${user.name} is not authorized to do ${action}.`);
      }
    }
    
 ${user.name} 和 ${action} 被称作 template 占位符。Javascript 会将 user.name 和 action 的值插入字符串。最后生成类似```User jorendorff is not authorized to do hockey```。  
 
 目前为止，功能仅限于比 **+** 运算符长得好看点，以下是这种用法你需要知道的细节:  
 
 * template 占位符可以是任何 Javascript 表达式，函数调用，算数操作等等(如果你愿意，甚至可以在 template string 里嵌套 template string, 我称它为 template 盗梦空间。)  
 
 * 如果占位符的值不是字符串，它将被转义为字符串。例如，如果 action 是一个对象，.toString() 方法将会被调用。  
 
 * 如果你需要在 template strings 里使用引号，须用反斜杠进行转义： **`\``** 与 **`** 等价。  
 
 * 同样的，要使用 **$**, 也需要转义：**`write \${** 或者 **\$`**。  
 
 与不同字符串不同的是，template strings 可以是多行：  
 
     $("#warning").html(`
         <h1>Watch out!</h1>
          <p>Unauthorized hockeying can result in penalties
          of up to ${maxPenalty} minutes.</p>
    `);
    
template strings 里的所有空白，包括换行跟缩进，都会按一字不落的输出。  

警告: 以下内容可能有些许不适。请适当酌情阅读，或许去喝杯咖啡放松下。  

###未来的引号  

让我们讨论下 template strings 做不了什么。  

* 它不会自动转义特殊字符。为了避免跨域脚本工具，对于不信任的数据仍需要你的关注，就像处理普通字符串一样。  

* 它再用来做国际化时，不适那么容易。template strings 并没有针对不同语言做数字、时间的处理，更不用说单复数了。  

* 它并不是[Mustache](https://mustache.github.io/) 或者 [Nunjacks](https://mozilla.github.io/nunjucks/) 之类模板引擎的替代品。  
原因在于 template strings 并不包含例如通过对数组迭代生成 HTML 的 table 结构、甚至是条件判断等语法(你可能觉得可以用 template 的层层嵌套做这件事，但这听起来就像是个笑话)。  

ES6 提供了一种隐晦的使用 template strings 的方式，来供 Js 开发者和库设计者打破上述限制。这个特性被称作 tagged templates。  

tagged templates 的语法非常简单。只需要在引号前加上额外的*tag*。上面提到html 内容转义，那么这个*tag* 就是 **SaferHTML**， 将需要限制的字符一一列出: 自动转义特殊字符。  

注意到 **SaferHTML** 并不是 ES6 的标注库。所以我们需要自己实现：  

    var message = 
        SaferHTML`<p>${bonk.sender} has sent you a bonk.</p>`;  
        
这里 SaferHTML 是个标示符，还可以用例如 SaferHTML.excape, 甚至是方法调用，例如：`SaferHTML.escape({unicodeControlCharacters: false})`(更准确点说，任何 ES6 [ MemberExpression or CallExpression](https://people.mozilla.org/~jorendorff/es6-draft.html#sec-left-hand-side-expressions) 可以称作是一个*tag*)。  

未 tagged 的 template strings 是简单字符串的连接。而带 tagged 的 templates 实际上是函数调用。  

上面代码等同于：  

    var message =
      SaferHTML(templateData, bonk.sender);
      
 