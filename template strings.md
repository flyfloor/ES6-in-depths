##template strings  

###"基础"  

ES6 引入了新的字面量字符语法，叫做 template strings。它看起来像是传统的字符串，区别在于用 **`** 取代了**'** 或者 **"**。简单的例子: 

    context.fillText(`Ceci n'est pas une chaîne.`, x, y);  
    
然而被叫做 ```template strings```，而不是```冗长的除了重音符不同的字符串```。原因在于 template strings 给 Javascript 带来了更简单的字符串修改方式。这种方式看起来很漂亮，很方便的修改字符串的值。  

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
 
 * 如果你需要在 template strings 里使用重音符，须用反斜杠进行转义： **`\``** 与 **`** 等价。  
 
 * 同样的，要使用 **$**, 也需要转义：**`write \${** 或者 **\$`**。  
 
 与不同字符串不同的是，template strings 可以是多行：  
 
     $("#warning").html(`
         <h1>Watch out!</h1>
          <p>Unauthorized hockeying can result in penalties
          of up to ${maxPenalty} minutes.</p>
    `);
    
