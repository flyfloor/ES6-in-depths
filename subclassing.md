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
    
