##subclassing  

两周前，ES6 添加了 [新的类系统](https://hacks.mozilla.org/2015/07/es6-in-depth-classes/) 来解决对象构造函数的反锁问题，展示了用它写出如下的代码：  

	class Circle {
	    constructor(radius) {
	        this.radius = radius;
	        Circle.circlesMade++;
	    };

	    static draw(circle, canvas) {
	        // Canvas drawing code
	    };

	    static get circlesMade() {
	        return !this._count ? 0 : this._count;
	    };
	    static set circlesMade(val) {
	        this._count = val;
	    };

	    area() {
	        return Math.pow(this.radius, 2) * Math.PI;
	    };

	    get radius() {
	        return this._radius;
	    };
	    set radius(radius) {
	        if (!Number.isInteger(radius))
	            throw new Error("Circle radius must be an integer.");
	        this._radius = radius;
	    };
	}

不幸的是，正如某些人提出的一样，没时间来聊聊关于 ES6 类的其他强大之处。像其他类系统(例如 C++ 或 Java)一样，ES6 允许继承，此类使用别的类作为基类，通过添加特性来扩展它。让我们更进一步看看这个新特性。  

在此之前，有必要花点时间回顾下属性继承跟动态原型链。  

###Javascript 继承  

当创建对象时，我们有机会改变其属性，但同时会继承来自其他对象的属性。Javascript 程序员对 Object.create API 应该都很熟悉：  

    var proto = {
        value: 4,
        method() { return 14; }
    }

    var obj = Object.create(proto);

    obj.value; // 4
    obj.method(); // 14  
    
更进一步，当我们给 obj 属性赋值，如果该属性跟 proto 的一致，obj 的属性将会改变 proto 属性的值。  

    obj.value = 5;
    obj.value; // 5
    proto.value; // 4  
    
###基础子类  

得知上面的问题后，我们该知道如何对类创建的对象保持其原型链。构造类的时候，创建了对应的 constructor 函数，其包含了类的所有静态方法。同时创建了该函数的 prototype 属性，包含了所有实例方法。要创建继承自该类的子类，只需让其继承自父类。同样的，需要将 prototype 对象继承自父类的 prototype，来获得实例方法。  

描述太过冗长。来个活生生的例子，用新语法保持继承类的原型链，然后在不断扩展到看着优雅舒服。  

继续上个例子，假设有 Shape 父类：  

    class Shape {
        get color() {
            return this._color;
        }
        set color(c) {
            this._color = parseColorAsRGB(c);
            this.markChanged();  // repaint the canvas later
        }
    }

当写下类似上面的代码，我们遇到了之前 static 属性存在的问题：无法改变之前定义在其原型上的函数。当然你可以通过  Object.setPrototypeOf, 这种办法相比直接通过目标 prototype 构造函数，性能低下且更难优化。  

    class Circle {
        // As above
    }

    // Hook up the instance properties
    Object.setPrototypeOf(Circle.prototype, Shape.prototype);

    // Hook up the static properties
    Object.setPrototypeOf(Circle, Shape);
    
太丑了。有了类的语法，我们可以把所有类相关的逻辑放到一起，而不是用其他连接方式来操作。Java，ruby，以及其他面向对象语言都有定义子类继承的语法，Js 也应该有，我们用关键字 extends 表示继承：  

    class Circle extends Shape {
        // As above
    }  
    
你可以在 extends 关键字后面跟随任何拥有 prototype 的构造函数，例如：  

+ 另一个类  

+ 遗留系统的类似于类的函数  

+ 普通函数  

+ 包含函数或者类的变量  

+ 对象属性的获取  

+ 函数调用  

如果你不想实例继承自 Object.prototype 的话，甚至可以在后面跟 null。  

###超级属性  

所以我们可以构造子类，可以继承属性，有时甚至可以覆盖被继承的方法。但如果我们需要绕过覆盖呢？  

假设由于某些原因，我们需要编写一个 Circle 的子类来处理圆形的缩放。我们需要写类似下面的代码：  

    class ScalableCircle extends Circle {
        get radius() {
            return this.scalingFactor * super.radius;
        }
        set radius() {
            throw new Error("ScalableCircle radius is constant." +
                            "Set scaling factor instead.");
        }

        // Code to handle scalingFactor
    }
    
注意到 radius 的 getter 使用了 super.radius。通过 super 关键字来绕过自身的属性，以及可能覆盖的属性找到继承父类的属性。  

父类属性访问器(super[expr] 同样可行)可以再任何函数方法里使用。此时该方法拿到原始的对象，该访问器绑定到对象方法最初被定义时的状态。这意味着将该方法赋值给局部变量也不会改变 super 访问器的行为：  

    var obj = {
        toString() {
            return "MyObject: " + super.toString();
        }
    }

    obj.toString(); // MyObject: [object Object]
    var a = obj.toString;
    a(); // MyObject: [object Object]  
    
###子类内建命令  

你可能想要为 Javascript 内建指令编写扩展。内建数据结构为语言提供很多强大的功能，也够早了许多新类型，使得这些功能变得更实用、更基础，例如子类。假设你想写个带版本控制的数组(我知道，相信我。)你应该提供能够改变数据并提交，以及回滚提交等功能。下面是使用 Array 子类的一种实现：  

    class VersionedArray extends Array {
        constructor() {
            super();
            this.history = [[]];
        }
        commit() {
            // Save changes to history.
            this.history.push(this.slice());
        }
        revert() {
            this.splice(0, this.length, this.history[this.history.length - 1]);
        }
    }
    
VersionedArray 的实例对象保存了一些重要的属性。它们完善了 map，filter 跟 sort，属于数组有诚意的实例。  Array.isArray() 将视它们为数组，甚至它们的 length 会自动得到更新。更深远的，返回数组的函数(像 Array.prototype.slice()) 将返回 VersionedArray！  

###派生类构造函数  

你可能注意到了，在最后一个例子构造函数里的 super() 函数，究竟发生了什么？  

传统的类模型中，构造函数用来初始化该类实例对象的任何内部状态。每个子类负责初始化根子类相关的状态。将调用串起来，这样子类就可以共享被扩展类的同样的初始化代码。  

这次我们还是使用 super 关键字来调用父构造函数，然而这次 super 是个函数。注意这种语法仅支持在子类的构造函数中使用。利用 super 关键字，将 Shape 类重写：  

    class Shape {
        constructor(color) {
            this._color = color;
        }
    }

    class Circle extends Shape {
        constructor(color, radius) {
            super(color);

            this.radius = radius;
        }

        // As from above
    }  
    
Javascript 里，我们倾向于写构造函数来操作 this 对象，插入的属性以及初始化的内部状态。通常当使用 new 调用构造函数，或者在对象的 prototype 属性上使用 Object.create()的同时，this 对象也会被的创建。然而某些内建指令拥有不同的内建对象布局，比如数组在内存中的储存的方式跟普通对象就不太一样。由于想要获取子类内建指令，我们需要使基类的构造函数指定 this 对象。如果是内建的，我们将得到对象的布局，但如果是普通构造函数，我们只能得到默认的 this 对象。  

可能最奇怪的答案莫过于 this 跟子类构造器的绑定方式了。除非运行基类的构造函数，且允许它来指定 this 值，不然我们将得不到 this 值。结果是如果还未调用 super 构造函数之前，获取 this 的值将得到 ReferenceError。  

上篇文章我们了解到你可以省略构造函数，下面的语法可以使得衍生类的构造函数被省略：  

    constructor(...args) {
        super(...args);
    }  
    
一些时候，构造函数并不需要 this 对象。取而代之，它们构造某些对象，初始化并直接返回。这样情况下就不需要调用 super。父类的构造函数是否被调用，并不影响任何直接返回对象的构造函数。  

###new.target  

使用基类来指定 this 对象的另一个副作用是有些时候基类压根儿不知道该指定哪种类型的对象。假设你正在写一个框架库，想要一个集合类，但是子类中有些是数组，有些却是 maps。那么当执行集合的构造函数时，你没办法知道究竟初始化的是哪种类型的集合！  

既然我们能使用子类，而当执行内部构造函数时，在内部已经能得到原始类的原型。没有它，我们将无法根据合适的实例方法构造对象。为了解决集合这个奇怪的例子，新加入了语法，好让信息暴露给Javascript 代码。我们增加了新的元属性 new.target ，它直接与调用 new 获得的构造函数通信。用 new 的方式调用函数将设置 new.target 为被调用的函数，而调用 super 则会转发 new.target  值。

这不太好理解，所以看个例子：  

```
class foo {
    constructor() {
        return new.target;
    }
}

class bar extends foo {
    // This is included explicitly for clarity. It is not necessary
    // to get these results.
    constructor() {
        super();
    }
}

// foo directly invoked, so new.target is foo
new foo(); // foo

// 1) bar directly invoked, so new.target is bar
// 2) bar invokes foo via super(), so new.target is still bar
new bar(); // bar
```

这样我们就解决了上面集合的问题，因为集合的构造函数可以检查 new.target 的直系来源，也就能够决定使用哪个类型来构造。  

new.target 可以在任何函数里使用，只要不是通过 new 构造的函数，它将被设置为 undefined。  

####两个领域的最佳选择  

