##其他参数及默认参数  

###其他参数  

在创建 API 时常常会需要一个可变参数的函数，该函数接受一系列的参数。例如：String.prototype.concat 方法接受多个字符串参数。ES6 提供一种新的方式来传递可变参数。  

我们写个简单的函数 `containsAll`，用来检查字符串是否包含一系列的子字符串。例如： containsAll("banana", "b", "nan") 会返回 true， 而 containsAll("banana", "c", "nan") 返回  false。  

一般我们是这样写的：  

    function containsAll(haystack) {
      for (var i = 1; i < arguments.length; i++) {
        var needle = arguments[i];
        if (haystack.indexOf(needle) === -1) {
          return false;
        }
      }
      return true;
    }  
    
这种实现利用了 arguments， 它包含了函数接收的所有参数，类似数组的对象。然而这段代码可读性并不理想。首先，函数的参数列表仅仅有 `haystack` 一个参数，所以第一眼看并不像可以传递多个参数。另外，对参数操作时也必须从 index 为1，而不是0来迭代，因为第一个参数是 haystack。其他参数解决了这些顾虑。下面是 ES6 实现的 `containsAll`：  

    function containsAll(haystack, ...needles) {
      for (var needle of needles) {
        if (haystack.indexOf(needle) === -1) {
          return false;
        }
      }
      return true;
    } 
    
 这个版本引入了 ...needles 的语法。让我们看看当执行 `containsAll("banana", "b", "nan")` 它是如何被调用的。haystack 被 `banana` 填充，needles 前跟的省略号暗示这部分属于其他参数。 其他参数被塞到 needles 这个数组里。这里的值为 `["b", "nan"]`。函数调用其他一切正常(注意到这里用到了 ES6 的 for-of 循环)。  
 
 只有最后一个参数可以作为其他参数，而前面的参数会被当做普通参数赋值。假如没有指定其他参数，那么其他参数为空数组，而不是 undefined。  
 
###默认参数  

通常不是所有参数都需要调用时赋值，一些可能会有默认值。Javascript 没被赋值的参数默认值为 undefined。ES6  提供一种方式来指定参数的任意默认值。  

例子：  

    function animalSentence(animals2="tigers", animals3="bears") {
        return `Lions and ${animals2} and ${animals3}! Oh my!`;
    }  
    
针对每个参数， `=` 表示如果调用者未对参数赋值时的默认值。所以，`animalSentence()` 会返回 `"Lions and tigers and bears! Oh my!", animalSentence("elephants") returns "Lions and elephants and bears! Oh my!", and animalSentence("elephants", "whales") returns "Lions and elephants and whales! Oh my!"`。  

