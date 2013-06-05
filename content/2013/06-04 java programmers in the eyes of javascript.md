# JavaScript的模块化：封装（闭包），继承（原型）

- date: 2013-06-04
- category: Work
- tags: java, javascript

----

随着用户体验被越来越重视，前端的 JavaScript 越来越流行。看起来随随便便的 JavaScript 其实也可以像 Java 一样进行封装继承，只是看起来没那么明显而已。今天看了这篇文章，讲得非常好，有种顿悟的感觉，下面简单记录一下并加入一些自己的观点，更详细的看[原文][1]吧。

> 我们试图在页面上维护一个计数器对象 ticker ，这个对象维护一个数值 n 。随着用户的操作，我们可以增加一次计数（将数值 n 加上 1 ），但不能减少 n 或直接改变 n 。而且，我们需要时不时查看这个数值。

## 门户大开的 JSON 风格模块化

	var ticker = {
	    n:0,
	    tick:function(){
	        this.n++;
	    },
	};

这种方式书写自然，而且确实有效，我们需要增加一次计数时，就调用 `ticker.tick()` 方法，需要查询次数时，就访问 `ticker.n` 变量。但是，模块的使用者被允许自由地改变 n ，比如调用 `ticker.n–` 或者 `ticker.n=-1`。。我们并没有对 `ticker` 进行封装， `n` 和 `tick()` 看上去是 `ticker` 的“成员”，但是它们的可访问性和 `ticker` 一样，都是全局性的（如果 `ticker` 是全局变量的话）。在封装性上，这种模块化的方式比下面这种更加可笑的方式，只好那么一点点（虽然对有些简单的应用来说，这一点点也足够了）。

	var ticker = {};
	var tickerN = 0;
	var tickerTick = function(){
	    tickerN++;
	}

	tickerTick();

值得注意的是，在 `tick()` 中，我访问的是 `this.n` ——这并不是因为 `n` 是 `ticker` 的成员，而是因为调用 `tick()` 的是 `ticker` 。事实上这里写成 `ticker.n` 会更好，因为如果调用 `tick()` 的不是 `ticker` ，而是其他什么东西，比如：

	var func = ticker.tick;
	func();

这时，调用 `tick()` 的其实是 `window` ，而函数执行时会试图访问 `window.n` 而出错。

> JSON风格的模块化其实只是定义了一个模块的组织方式，并没有进行封装，它的成员其实都是全局变量。

## 作用域链和闭包

	var config = {
	    nStart:100,
	    step:2
	}

	function ticker(config){
	    var n = config.nStart;
	    function tick(){
	        n += config.step;
	    }
	}
	console.log(ticker.n); // ->undefined

JavaScript 中只有函数具有作用域，即在函数体外无法访问函数内部的变量。上面的例子，从 `tick()` 到 `ticker()` 再到全局，这就是 JavaScript 中的“作用域链”。

可是还有问题，那就是——怎么调用 `tick()` ？ `ticker()` 的作用域将 `tick()` 也掩盖了起来。解决方法有两种：

- 1）将需要调用方法作为返回值，正如我们将递增 n 的方法作为 `ticker()` 的返回值；
- 2）设定外层作用域的变量，正如我们在 `ticker()` 中设置 `getN` 。

```
var getN;
function ticker(config){
    var n = config.nStart;
    getN = function(){
        return n;
    };
    return function(){
        n += config.step;
    };
}

var tick = ticker({nStart:100,step:2});
tick();
console.log(getN()); // ->102
```

这时，变量 `n` 就处在“闭包”之中，在 `ticker()` 外部无法直接访问它，但是却可以通过两个方法来观察或操纵它。

在本节第一段代码中， `ticker()` 方法执行之后， `n` 和 `tick()` 就被销毁了，直到下一次调用该函数时再创建；但是在第二段代码中， `ticker()` 执行之后， `n` 不会被销毁，因为 `tick()` 和 `getN()` 可能访问它或改变它，浏览器会负责维持n。我对“闭包”的理解就是：**用以保证 `n` 这种处在函数作用域内，函数执行结束后仍需维持，可能被通过其他方式访问的变量 不被销毁的机制。**

可是，如果我需要维持两个具有相同功能的对象 `ticker1` 和 `ticker2` ，那该怎么办？ `ticker()` 只有一个，总不能再写一遍吧？

## new 运算符与构造函数

如果通过 `new` 运算符调用一个函数，就会创建一个新的对象，并使用该对象调用这个函数。在我的理解中，下面的代码中 `t1` 和 `t2` 的构造过程是一样的。

	function myClass(){}
	var t1 = new myClass();
	var t2 = {};
	t2.func = myClass;
	t2.func();
	t2.func = undefined;

`t1` 和 `t2` 都是新构造的对象， `myClass()` 就是构造函数了。类似的， `ticker()` 可以重新写成。

```
function TICKER(config){
    var n = config.nStart;
    this.getN = function(){
        return n;
    };
    this.tick = function(){
        n += config.step;
    }
}

var ticker1 = new TICKER({nStart:100,step:2});
ticker1.tick();
console.log(ticker1.getN()); // ->102
var ticker2 = new TICKER({nStart:20,step:3});
ticker2.tick();
ticker2.tick();
console.log(ticker2.getN()); // ->26
```
习惯上，构造函数采用大写。注意， `TICKER()` 仍然是个函数，而不是个纯粹的对象（之所以说“纯粹”，是因为函数实际上也是对象， `TICKER()` 是函数对象），闭包依旧有效，我们无法访问 `ticker1.n` 。

## 原型 prototype 与继承

上面这个 `TICKER()` 还是有缺陷，那就是， `ticker1.tick()` 和 `ticker2.tick()` 是互相独立的！请看，每使用 `new` 运算符调用 `TICKER()` ，就会生成一个新的对象并生成一个新的函数绑定在这个新的对象上，每构造一个新的对象，浏览器就要开辟一块空间，存储 `tick()` 本身和 `tick()` 中的变量，这不是我们所期望的。我们期望 `ticker1.tick` 和 `ticker2.tick` 指向同一个函数对象。

**JavaScript 中，除了 Object 对象，其他对象都有一个 prototype 属性，这个属性指向另一个对象。这“另一个对象”依旧有其原型对象，并形成原型链，最终指向 Object 对象。在某个对象上调用某方法时，如果发现这个对象没有指定的方法，那就在原型链上一次查找这个方法，直到 Object 对象。**

函数也是对象，因此函数也有原型对象。当一个函数被声明出来时（也就是当函数对象被定义出来时），就会生成一个新的对象，这个对象的 prototype 属性指向 Object 对象，而且这个对象的 constructor 属性指向函数对象。

通过构造函数构造出的新对象，其原型指向构造函数的原型对象。所以我们可以在构造函数的原型对象上添加函数，这些函数就不是依赖于 `ticker1` 或 `ticker2` ，而是依赖于 `TICKER` 了。

为了访问闭包中的内容，对象必须有一些简洁的依赖于实例的方法，来访问闭包中的内容，然后在其 `prototype` 上定义复杂的公有方法来实现逻辑。实际上，例子中的 `tick()` 方法就已经足够简洁了，我们还是把它放回到 `TICKER` 中吧。下面实现一个复杂些的方法 `tickTimes()` ，它将允许调用者指定调用 `tick()` 的次数。

```
function TICKER(config){
    var n = config.nStart;
    this.getN = function(){
        return n;
    };
    this.tick = function(){
        n += config.step;
    };
}
TICKER.prototype.tickTimes = function(n){
    while(n>0){
        this.tick();
        n--;
    }
};
var ticker1 = new TICKER({nStart:100,step:2});
ticker1.tick();
console.log(ticker1.getN()); // ->102
var ticker2 = new TICKER({nStart:20,step:3});
ticker2.tickTimes(2);
console.log(ticker2.getN()); // ->26
```
这个 `TICKER` 就很好了。它封装了 `n` ，从对象外部无法直接改变它，而复杂的函数 `tickTimes()` 被定义在原型上，这个函数通过调用实例的小函数来操作对象中的数据。

所以，为了维持对象的封装性，我的建议是，将对数据的操作解耦为尽可能小的单元函数，在构造函数中定义为依赖于实例的（很多地方也称之为“私有”的），而将复杂的逻辑实现在原型上（即“公有”的）。

最后再说一些关于继承的话。实际上，当我们在原型上定义函数时，我们就已经用到了继承！ JavaScript 中的继承比 C++ 中的更……呃……简单，或者说简陋。在 C++ 中，我们可能会定义一个 animal 类表示动物，然后再定义 bird 类继承 animal 类表示鸟类，但我想讨论的不是这样的继承（虽然这样的继承在 JavaScript 中也可以实现）；我想讨论的继承在 C++ 中将是，定义一个 animal 类，然后实例化了一个 myAnimal 对象。对，这在 C++ 里就是实例化，但在 JavaScript 中是作为继承来对待的。

JavaScript 并不支持类，浏览器只管当前有哪些对象，而不会额外费心思地去管，这些对象是什么 class 的，应该具有怎样的结构。在我们的例子中， TICKER() 是个函数对象，我们可以对其赋值（TICKER=1），将其删掉（TICKER=undefined），但是正因为当前有 ticker1 和 ticker2 两个对象是通过 new 运算符调用它而来的， TICKER() 就充当了构造函数的作用，而 TICKER.prototype 对象，也就充当了类的作用。


[1]: http://www.w3c.com.cn/?p=1320589