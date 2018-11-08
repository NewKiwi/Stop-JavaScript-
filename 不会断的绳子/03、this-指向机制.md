以下参考《JavaScript高级程序设计》和《你不知道的JavaScript》上卷
## 前言
what:我认为this关键字是JavaScript中的一种指向机制，和作用域有所关联，但指向的不一定是定义时所在的作用域，通常来说，this的指向不是定义时确定的，而是运行时基于函数的执行环境确定的（或者说只取决于函数的调用方式）。

## this如此复杂，为什么要用this
Why:this 提供了一种更优雅的方式来隐式“传递”一个对象引用，因此可以将API设计得更加简洁并且易于复用。

例子，在不同的上下文对象中重复使用函数 identify() 和 speak()：
如果不用this,则需要显式传入一个上下文对象
```javascript
function identify(context) {
  return context.name.toUpperCase();
}
function speak(context) {
  var greeting = "Hello, I'm " + identify( context ); 
  console.log( greeting );
}
var me = {
  name: "Kyle"
};
var you = {
  name: "Reader"
};

identify( you ); // READER
speak( me ); //hello, 我是 KYLE
```

而使用this:
```javascript
function identify() {
  return this.name.toUpperCase();
}
function speak() {
  var greeting = "Hello, I'm " + identify.call( this );
  console.log( greeting );
}
var me = {
  name: "Kyle"
};
var you = {
  name: "Reader"
};
identify.call( me ); // KYLE
identify.call( you ); // READER
speak.call( me ); // Hello, 我是 KYLE 
speak.call( you ); // Hello, 我是 READER
```
随着你的使用模式越来越复杂，显式传递上下文对象会让代码变得越来越混乱，使用 this 则不会这样。(而且在对象和原中，函数可以自动引用合适的上下文对象非常重要。)

## 对this的错误理解
对this的错误理解有两种：
- this指向函数对象
- this指向函数作用域（函数作用域是无法用javascript访问的，即使作用域中可见的标示符都是它的属性。它只存在于引擎内部，this 在任何情况下都不指向函数的词法作用域）

### 函数引用自身引发的思考
如何引用自身：
- 具名函数，用标示符来引用

因为this并不指向函数对象本身，用this无法引用自身。
匿名函数无法不具有名称，无法引用自身。使用 arguments. callee 来引用当前正在运行的函数对象。这是唯一一种可以从匿名函数对象内部引用自身的方法。然而，更好的方式是避免使用匿名函数，至少在需要自引用时使用具名函数(表达式)。arguments.callee 已经被弃用，不应该再使用它。

##this什么时候产生？
前面说过，当用执行环境来理解作用域时，每一个执行环境都有一个变量对象与之相连。当调用函数时，会取得活动对象的两个属性:this和arguments。而对这两个变量的搜索，只会在活动对象中搜索。而this的指向由函数的调用方法确定。（为什么不是函数的调用栈。因为this的指向可以动态改变）

### 由只在活动对象中搜索this引发的思考
```javascript
var name = "The Window";
var object = {
    name : "My Object",
    getNameFunc : function(){
        return function(){
            return this.name;
        };
  }
};
alert(object.getNameFunc()()); //"The Window"(在非严格模式下)
```
只能在当前活动对象中搜索this和arguments,因此永远不可能直接访问外部函数中的这两个变量。

但可以通过闭包来保持对外部函数活动对象的引用，即使用了词法作用域的概念
```javascript
var name = "The Window";
var object = {
    name : "My Object",
    getNameFunc : function(){
      var that = this;
        return function(){
            return that.name;
        };
  }
};
alert(object.getNameFunc()());//"My Object"
```
或者使用bind
```javascript
var name = "The Window";
var object = {
    name : "My Object",
    getNameFunc : function(){
      return function(){
          return this.name;
      }.bind(this);
  }
};
alert(object.getNameFunc()());//"My Object"
```
或者使用箭头函数：
```javascript
var name = "The Window";
var object = {
    name : "My Object",
    getNameFunc : function(){
        return () => {
            return this.name;
        };
  }
};
alert(object.getNameFunc()());//"My Object"
```
这也是箭头函数出现的原因之一，放弃了所有普通this绑定的规则，取而代之的是用当前的词法作用域覆盖了this本来的值。（我理解有两种：1、“this什么时候产生”章节中，把“只能在当前活动对象中搜索this和arguments”的行为改写了，可以通过作用域链搜索this 2、箭头函数表示直接使用外部活动对象this的值改写内部this的值，可是调用函数时才出现this和arguments，所以我更倾向于第一种，需要更多资料来确定）

MDN中关于箭头函数的描述是：
- 箭头函数不会创建自己的this,它只会从自己的作用域链的上一层继承this。（所以函数执行时并不会创建this，可是这个继承？是否作用域链查询）
- 不绑定Arguments对象，arguments只是引用了封闭作用域内的arguments。

### 箭头函数
What:箭头函数的写法通常被当作单调乏味且冗长(挖苦)的 function 关键字的简写。

#### 箭头函数的缺点
- 它们是匿名而非具名的。见（不会断的绳子-和作用域紧紧相接的闭包）
- 不能用作构造函数

#### call和apply调用箭头函数
MDN:由于 箭头函数没有自己的this指针，通过 call() 或 apply() 方法调用一个函数时，只能传递参数（不能绑定this），他们的第一个参数会被忽略。（这种现象对于bind方法同样成立）
```javascript
var adder = {
  base : 1,
    
  add : function(a) {
    var f = v => v + this.base;
    return f(a);
  },

  addThruCall: function(a) {
    var f = v => v + this.base;
    var b = {
      base : 2
    };
            
    return f.call(b, a);
  }
};

console.log(adder.add(1));         // 输出 2
console.log(adder.addThruCall(1)); // 仍然输出 2
```