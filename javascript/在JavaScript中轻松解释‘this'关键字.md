## 在JavaScript中轻松解释'this'关键字

[本文翻译自](https://dmitripavlutin.com/gentle-explanation-of-this-in-javascript/#4constructorinvocation)


### 1.神秘的‘this’

很多时候，这个关键词对我和许多开始JavaScript开发者来说都是一个谜。这是一个强大的功能，但需要努力去理解。

从诸如Java，PHP或其他标准语言的背景中，这被看作类方法中当前对象的一个​​实例：不多也不少。大多数情况下，它不能在方法之外使用，这种简单的方法不会造成混淆。

在JavaScript中，情况有所不同：这是一个函数的当前执行上下文(context)。该语言有4种函数调用类型：
 - function invocation(函数调用)：alert（'Hello World！'）
 - method invocation(方法调用)：console.log（'Hello World！'）
 - constructor invocation(构造函数调用)：new RegExp（'\\ d'）
 - indirect invocation(间接调用)：alert.call（未定义，'Hello World！'）

 每种调用类型都以自己的方式定义了上下文，因此这种行为与开发人员的期望略有不同。

而且 [strict mode](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode) 也会影响执行上下文。

理解这个关键字的关键在于清楚地了解函数调用以及它如何影响上下文。
本文主要关注调用解释，函数调用如何影响这一点，并演示识别上下文的常见缺陷。

开始之前，让我们先熟悉一下几个术语：
 - 函数的调用是执行构成函数主体的代码，或者只是调用函数。例如parseInt函数调用是parseInt（'15'）。
 - 调用的上下文是函数体内的值。例如，调用map.set（'key'，'value'）具有上下文映射。
 - 函数的范围是一组函数体内可访问的变量，对象和函数。

### 2.函数调用

**当函数对象的求值表达式后面跟着一个开括号（逗号分隔的参数表达式和一个右括号）时，函数调用被执行。** 例如parseInt（'18'）。

函数调用表达式不能是一个[property accessor](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Property_accessors) obj.myFunc（），它会创建一个方法调用。例如[1,5] .join（'，'）不是函数调用，而是方法调用。这个区别很重要。

函数调用的一个简单例子：
```javascript
function hello(name) {
  return 'Hello ' + name + '!';
}
// Function invocation
var message = hello('World');
console.log(message); // => 'Hello World!'
```
hello（'World'）是函数调用：hello表达式求值为一个函数对象，然后是一对带有'World'参数的括号。

更高级的例子是**IIFE** (immediately-invoked function expression)：
```javascript
var message = (function(name) {
   return 'Hello ' + name + '!';
})('World');
console.log(message) // => 'Hello World!'
```
IIFE也是一个函数调用：第一对括号（函数（名称）{...}）是一个表达式，其值为一个函数对象，然后是一对带有'World'参数（'World'）的括号。

 #### 2.1  函数调用中的‘this’
 > this是函数调用中的全局对象。

 全局对象由执行环境决定。在浏览器中，它是[window](https://developer.mozilla.org/en-US/docs/Web/API/Window)对象。
![](https://raw.githubusercontent.com/leegaox/notes/javascript/pics/2_1.png)
在函数调用中，执行上下文是全局对象。

让我们来检查下面的函数中的上下文：

```javascript
function sum(a, b) {
   console.log(this === window); // => true
   this.myNumber = 20; // add 'myNumber' property to global object
   return a + b;
}
// sum() is invoked as a function
// this in sum() is a global object (window)
sum(15, 16);     // => 31
window.myNumber; // => 20
```
在调用sum（15,16）时，JavaScript自动将其设置为全局对象，它在浏览器中是窗口。

当this在任何函数作用域（最顶端的作用域：全局执行上下文）之外使用时，它也引用全局对象：
```javascript
console.log(this === window); // => true
this.myString = 'Hello World!';
console.log(window.myString); // => 'Hello World!'
```

```javascript
<!-- In an html file -->
<script type="text/javascript">
   console.log(this === window); // => true
</script>
```

#### 2.2 函数调用，strict mode（严格模式）中的‘this’
> this 在严格模式下的函数调用中未定义（**undefined** ）

strict mode 是在[ECMAScript 5.1](http://www.ecma-international.org/ecma-262/5.1/#sec-10.1.1)中引入的，它是JavaScript的一个受限变体。它提供更好的安全性和更强的错误检查。

要启用严格模式，请将指令'use strict'放在函数体的顶部。

一旦启用，strict mode将影响执行上下文，从而使其在常规函数调用中未定义。执行上下文不再是全局对象，与上面的情况2.1相反。
![](https://raw.githubusercontent.com/leegaox/notes/javascript/pics/2_2.png)

以strict mode执行的函数示例：
```javascript
function multiply(a, b) {
  'use strict'; // enable the strict mode
  console.log(this === undefined); // => true
  return a * b;
}
// multiply() function invocation with strict mode enabled
// this in multiply() is undefined
multiply(2, 5); // => 10
```
当multiply（2，5）作为函数被调用时，**this**为**undefined**。

strict mode不仅在当前范围内有效，而且在内部范围内（对于在内部声明的所有函数）：
```javascript
function execute() {
   'use strict'; // activate the strict mode
   function concat(str1, str2) {
     // the strict mode is enabled too
     console.log(this === undefined); // => true
     return str1 + str2;
   }
   // concat() is invoked as a function in strict mode
   // this in concat() is undefined
   concat('Hello', ' World!'); // => "Hello World!"
}
execute();
```

'use strict'插入到执行主体的顶部，从而在其范围内启用strict mode。因为concat是在执行范围内声明的，所以它继承了strict mode。而调用concat（'Hello'，'World！'）使得this成为undefined。

一个JavaScript文件可能包含严格模式和非严格模式。因此，对于相同的调用类型，可以在单个脚本中具有不同的上下文行为：
```javascript
function nonStrictSum(a, b) {
  // non-strict mode
  console.log(this === window); // => true
  return a + b;
}
function strictSum(a, b) {
  'use strict';
  // strict mode is enabled
  console.log(this === undefined); // => true
  return a + b;
}
// nonStrictSum() is invoked as a function in non-strict mode
// this in nonStrictSum() is the window object
nonStrictSum(5, 6); // => 11
// strictSum() is invoked as a function in strict mode
// this in strictSum() is undefined
strictSum(8, 12); // => 20
```
#### 2.3 陷阱：内部函数中的this
**注意：** 函数调用的一个**常见陷阱**是*认为内部函数和外部函数中的相同*。

**正确地说，内部函数的上下文仅依赖于调用，而不依赖于外部函数的上下文。**

为了获得预期的this，请使用[.call()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/call)或[.apply()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/apply)来修改内部函数的上下文（见5.）或创建一个绑定函数（使用.bind（），见6.）。

以下示例计算两个数字的总和：

```javascript
var numbers = {
   numberA: 5,
   numberB: 10,
   sum: function() {
     console.log(this === numbers); // => true
     function calculate() {
       // this is window or undefined in strict mode
       console.log(this === numbers); // => false
       return this.numberA + this.numberB;
     }
     return calculate();
   }
};
numbers.sum(); // => NaN or throws TypeError in strict mode
```

**注意：** numbers.sum（）是一个对象的方法调用（参见3.）,所以sum的context是numbers对象。
calculate 函数是在sum内定义的，所以你可能期望在calculate（）中也有这个numbers 对象。

尽管如此，calculate（）是一个函数调用（但不是方法调用），它将this作为全局对象**window**（情况2.1）或者在 strict mode（案例2.2）中的 **undefined**。即使外部函数sum具有作为number对象的上下文context ，它在这里也没有影响。

numbers.sum（）的调用结果是NaN或在strict mode下引发错误TypeError: Cannot read property 'numberA' of undefined。绝对不是预期的结果5 + 10 = 15，都是因为caculate没有被正确调用。

**为了解决这个问题，calculate 函数应该使用与sum方法相同的上下文来执行，以便访问numberA和numberB属性。**

一种解决方案是通过调用calculate.call（this）（函数的间接调用，请参见第5节）手动更改calculate 的上下文到期望的上下文。:

```javascript
var numbers = {
   numberA: 5,
   numberB: 10,
   sum: function() {
     console.log(this === numbers); // => true
     function calculate() {
       console.log(this === numbers); // => true
       return this.numberA + this.numberB;
     }
     // use .call() method to modify the context
     return calculate.call(this);
   }
};
numbers.sum(); // => 15
```
calculate.call(this)像往常一样执行calculate 函数，但还会将上下文修改为指定为第一个参数的值。
现在this.numberA + this.numberB相当于numbers.numberA + numbers.numberB。该函数返回预期结果5 + 10 = 15。

### 3.方法调用
**方法是存储在对象属性中的函数。** 例如：
```javascript
var myObject = {
  // helloFunction is a method
  helloFunction: function() {
    return 'Hello World!';
  }
};
var message = myObject.helloFunction();
```

helloFunction是myObject的一种方法。要获取该方法，请使用**属性访问器**：myObject.helloFunction。

**当一个表达式以[property accessor](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Property_accessors)的形式出现即函数对象后面跟着一个开括号（，逗号分隔的参数表达式列表和一个右括号）时方法调用被执行。**

回顾前面的例子，myObject.helloFunction（）是对象myObject的helloFunction的方法调用。方法调用还有：[1,2] .join（'，'）或/s/.test('beautiful world'）。

区分**函数调用**（参见第2节）和**方法调用**很重要，因为它们是不同的类型。**主要区别**在于方法调用需要一个属性访问器形式来调用函数（obj.myFunc（）或obj ['myFunc']（）），而函数调用不会（myFunc（））

以下调用列表显示了如何区分这些类型：
```javascript
['Hello', 'World'].join(', '); // method invocation
({ ten: function() { return 10; } }).ten(); // method invocation
var obj = {};
obj.myFunction = function() {
  return new Date().toString();
};
obj.myFunction(); // method invocation

var otherFunction = obj.myFunction;
otherFunction();     // function invocation
parseFloat('16.60'); // function invocation
isNaN(0);            // function invocation
```
**理解函数调用和方法调用之间的区别有助于正确识别contex上下文。**

#### 3.1 方法调用中的this
>在方法调用中'this'是拥有该方法的对象。

当在一个对象上调用一个方法时，'this'成为对象本身。
![](https://raw.githubusercontent.com/leegaox/notes/javascript/pics/3_1.png)
让我们用一个增加数字的方法创建一个对象：
```javascript
var calc = {
  num: 0,
  increment: function() {
    console.log(this === calc); // => true
    this.num += 1;
    return this.num;
  }
};
// method invocation. this is calc
calc.increment(); // => 1
calc.increment(); // => 2
```

调用calc.increment（）使increment 函数的上下文成为calc对象。所以使用this.num来增加数字属性是行之有效的。

我们来看另一个案例。JavaScript对象从其**prototype**继承了一种方法。当在对象上调用继承的方法时，调用的上下文仍然是对象本身：
```javascript
var myDog = Object.create({
  sayName: function() {
     console.log(this === myDog); // => true
     return this.name;
  }
});
myDog.name = 'Milo';
// method invocation. this is myDog
myDog.sayName(); // => 'Milo'
```

[Object.create()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/create)创建一个新的对象myDog并设置prototype.myDog对象继承sayName方法。
当执行myDog.sayName()时，myDog是调用的上下文。

在ECMAScript 6 [class](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes)语法中，方法调用上下文也是实例本身：
```javascript
class Planet {
  constructor(name) {
    this.name = name;
  }
  getName() {
    console.log(this === earth); // => true
    return this.name;
  }
}
var earth = new Planet('Earth');
// method invocation. the context is earth
earth.getName(); // => 'Earth'
```

#### 3.2 陷阱：从它的对象分离方法

**注意：**来自对象的方法可以被提取到独立变量var alone = myObj.myMethod中。当单独调用该方法时，从原始对象分离alone()，您可能会认为‘this’是定义该方法的对象。

正确地说，如果在没有对象的情况下调用该方法，则会发生函数调用：其中，this是全局对象window或者在严格模式下的undefined（参见2.1和2.2）。

创建一个绑定函数 var alone = myObj.myMethod.bind(myObj)（using [.bind()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_objects/Function/bind)  , see 6.）修复上下文，使其成为拥有该方法的对象。

以下示例创建Animal构造函数并创建它的一个实例 - myCat。然后在1秒钟后setTimout（）记录myCat对象信息：
```javascript
function Animal(type, legs) {
  this.type = type;
  this.legs = legs;
  this.logInfo = function() {
    console.log(this === myCat); // => false
    console.log('The ' + this.type + ' has ' + this.legs + ' legs');
  }
}
var myCat = new Animal('Cat', 4);
// logs "The undefined has undefined legs"
// or throws a TypeError in strict mode
setTimeout(myCat.logInfo, 1000);
```

**注意：**你可能会认为setTimout会调用myCat.logInfo（），它应该记录有关myCat对象的信息。

**不幸的是，当作为参数传递时，该方法与其对象分离**：setTimout（myCat.logInfo）。以下情况等同：
```javascript
setTimout(myCat.logInfo);
// is equivalent to:
var extractedLogInfo = myCat.logInfo;
setTimout(extractedLogInfo);
```

当分离的logInfo作为函数调用时，这是全局对象，或者在严格模式下（但不是myCat对象）的undefined。所以对象信息不能正确记录。

一个函数可以使用[.bind()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_objects/Function/bind)方法绑定一个对象（见6.）。如果分离的方法与myCat对象绑定，则解决上下文问题：

```javascript
function Animal(type, legs) {
  this.type = type;
  this.legs = legs;
  this.logInfo = function() {
    console.log(this === myCat); // => true
    console.log('The ' + this.type + ' has ' + this.legs + ' legs');
  };
}
var myCat = new Animal('Cat', 4);
// logs "The Cat has 4 legs"
setTimeout(myCat.logInfo.bind(myCat), 1000);
```

myCat.logInfo.bind(myCat)返回一个完全像logInfo一样执行的新函数，但是将其作为myCat使用，即使在函数调用中也是如此。

### 4.构造函数调用
当[new](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/new)的关键字后跟一个表达式时，将执行构造函数调用即一个函数对象，一个左括号（逗号分隔的参数表达式列表和一个右括号）。例如：new RegExp（'\\ d'）。

本示例声明一个函数Country，然后将其作为构造函数调用它：
```javascript
function Country(name, traveled) {
   this.name = name ? name : 'United Kingdom';
   this.traveled = Boolean(traveled); // transform to a boolean
}
Country.prototype.travel = function() {
  this.traveled = true;
};
// Constructor invocation
var france = new Country('France', false);
// Constructor invocation
var unitedKingdom = new Country;

france.travel(); // Travel to France
```

new Country('France', false) 是Country函数的构造函数调用。执行结果是一个新的对象，其name属性是'France'。
**如果不带参数调用构造函数，那么可以省略括号对**：new Country。

ECMAScript 2015开始，JavaScript允许使用[class](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes)语法定义构造函数：
```javascript
class City {
  constructor(name, traveled) {
    this.name = name;
    this.traveled = false;
  }
  travel() {
    this.traveled = true;
  }
}
// Constructor invocation
var paris = new City('Paris', false);
paris.travel();
```
new City('Paris') 是一个构造函数调用。对象初始化由class中的特殊方法处理：[constructor](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/constructor)
，它有'this'作为新创建的对象。

构造函数调用将创建一个空的新对象，该对象从构造函数的prototype继承属性。构造函数的作用是初始化对象。
正如你可能已经知道的那样，这种类型的调用的上下文是创建的实例。这是下一章主题。

当一个属性访问器myObject.myFunction的前面是new关键字，JavaScript将执行构造函数调用，但不会执行方法调用。例如new myObject.myFunction()：**首先使用属性访问器提取函数**:extractedFunction = myObject.myFunction,**然后作为构造函数被调用来创建一个新的对象**: new extractedFunction().

#### 4.1 构造函数调用中的this
> this 是构造函数调用中新创建的对象

构造函数调用的上下文是新创建的对象。它用于使用来自构造函数参数的数据初始化对象，为属性设置初始值，附加事件处理程序等。
![](https://raw.githubusercontent.com/leegaox/notes/javascript/pics/4_1.png)

让我们来看看下面例子中的上下文：
```javascript
function Foo () {
  console.log(this instanceof Foo); // => true
  this.property = 'Default Value';
}
// Constructor invocation
var fooInstance = new Foo();
console.log(fooInstance.property);
```
new Foo()在上下文为fooInstance的情况下进行构造函数调用。在Foo内部，对象被初始化：this.property被分配一个默认值。

使用[class](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes)语法时会发生同样的脚本(在ES2015中可用)，只有初始化发生在构造函数方法中：
```javascript
class Bar {
  constructor() {
    console.log(this instanceof Bar); // => true
    this.property = 'Default Value';
  }
}
// Constructor invocation
var barInstance = new Bar();
console.log(barInstance.property); // => 'Default Value'
```
当new Bar()被执行时，avaScript创建一个空对象并使其成为构造函数方法的上下文。现在您可以使用此关键字将属性添加到对象：this.property ='Default Value'。

#### 4.2 陷阱：忘记 new
一些JavaScript函数不仅在作为构造函数调用时创建实例，还在作为函数调用时创建实例。例如 RegExp:
```javascript
var reg1 = new RegExp('\\w+');
var reg2 = RegExp('\\w+');

reg1 instanceof RegExp;      // => true
reg2 instanceof RegExp;      // => true
reg1.source === reg2.source; // => true
```

当执行new RegExp（'\\ w +'）和RegExp（'\\ w +'）时，JavaScript会创建等效的正则表达式对象。

**注意：**使用函数调用来创建对象是一个潜在的问题（不包括工厂模式），因为当new关键字丢失时，一些构造函数可能会忽略初始化对象的逻辑。

以下示例说明了这个问题：
```javascript
function Vehicle(type, wheelsCount) {
  this.type = type;
  this.wheelsCount = wheelsCount;
  return this;
}
// Function invocation
var car = Vehicle('Car', 4);
car.type;       // => 'Car'
car.wheelsCount // => 4
car === window  // => true
```
Vehicle是一个在上下文对象上设置type和wheelsCount属性的函数。当执行Vehicle('Car', 4)一个car对象被返回，它具有正确的属性：car.type是'Car'，car.wheelsCount是4。您可能认为它适用于创建和初始化新对象。

但是，this是函数调用中的window对象 (见 2.1.)，结果Vehicle('Car', 4)在window对象上设置熟悉-错误的情况。一个新的对象不会被创建。

**确保在需要构造函数调用时使用new运算符**：

```javascript
function Vehicle(type, wheelsCount) {
  if (!(this instanceof Vehicle)) {
    throw Error('Error: Incorrect invocation');
  }
  this.type = type;
  this.wheelsCount = wheelsCount;
  return this;
}
// Constructor invocation
var car = new Vehicle('Car', 4);
console.log(car.type) ;              // => 'Car'
console.log(car.wheelsCount)     ;   // => 4
console.log(car instanceof Vehicle); // => true

// Function invocation. Generates an error.
var brokenCar = Vehicle('Broken Car', 3);
```
new Vehicle('Car', 4)运行良好：由于new关键字存在于构造函数调用中，因此会创建并初始化一个新对象。

在构造函数中添加一个验证：this instanceof Vehicle，以确保执行上下文是正确的对象类型。如果this不是Vehicle对象，则会产生错误。每当Vehicle('Broken Car',3)执行（没有new）时抛出一个异常：Error: Incorrect invocation.（不正确的调用）。

### 间接调用
使用myFun.call()或myFun.apply()方法调用函数时将执行间接调用。

JavaScript中的函数是第一类对象，这意味着函数是一个对象。该对象的类型是[Function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function)。

从函数对象所具有的 [list of methods](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function#Methods)中,.call() and .apply() are used to invoke the function with a configurable context:
中文(简体)[.call()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/call)和[.apply()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/apply)用于使用可配置的上下文调用该函数：

  - 方法.call(thisArg [，arg1 [，arg2 [，...]]])接受第一个参数thisArg作为调用的上下文和一个参数列表arg1，arg2，...作为参数传递给被调用函数。
  - 方法.apply(thisArg, [arg1, arg2, ...]) 接受第一个参数thisArg作为调用的上下文和类似数组的对象[array-like object](http://2ality.com/2013/05/quirk-array-like-objects.html)[arg1, arg2, ...] 作为参数传递给被调用的函数。

以下示例演示了间接调用：
```javascript
function increment(number) {
  return ++number;
}
increment.call(undefined, 10);    // => 11
increment.apply(undefined, [10]); // => 11
```
increment.call() 和 increment.apply()都使用参数10调用increment函数。

两者之间的主要区别在于.call()接受参数列表，例如myFun.call(thisValue，'val1'，'val2'),但.apply()接受类似数组的对象中的值列表，例如myFunc.apply（thisValue，['val1'，'val2']）。

#### 5.1 间接调用中的this
> this 是间接调用.call()或者.apply()的第一个参数。

很明显，这是间接调用中传递给.call()或.apply()的第一个参数的值。
![](https://raw.githubusercontent.com/leegaox/notes/javascript/pics/5_1.png)

以下示例显示了间接调用上下文：
```javascript
var rabbit = { name: 'White Rabbit' };
function concatName(string) {
  console.log(this === rabbit); // => true
  return string + this.name;
}
// Indirect invocations
concatName.call(rabbit, 'Hello ');  // => 'Hello White Rabbit'
concatName.apply(rabbit, ['Bye ']); // => 'Bye White Rabbit'
```
当使用特定上下文执行函数时，间接调用很有用。例如，为了解决函数调用的上下文问题，它总是window，或者在严格模式下的undefined（见2.3）。它可以用来模拟对象上的方法调用（请参阅前面的代码示例）。

**另一个实际的例子是在ES5中创建类的层次结构来调用父构造函数：**
```javascript
function Runner(name) {
  console.log(this instanceof Rabbit); // => true
  this.name = name;
}
function Rabbit(name, countLegs) {
  console.log(this instanceof Rabbit); // => true
  // Indirect invocation. Call parent constructor.
  Runner.call(this, name);
  this.countLegs = countLegs;
}
var myRabbit = new Rabbit('White Rabbit', 4);
console.log('name: '+myRabbit.name +' countLegs: '+myRabbit.countLegs);// myRabbit:{ name: 'White Rabbit', countLegs: 4 }
```
在Rabbit内部的Runner.call(this，name)会间接调用父函数来初始化对象。

### 绑定函数
绑定函数是一个与对象连接的函数。通常它是使用.bind()方法从原始函数创建的。原始和绑定函数共享相同的代码和范围，但执行时会有不同的上下文。

方法.bind(thisArg [，arg1 [，arg2 [，...]]])接受第一个参数thisArg作为调用时绑定函数的上下文和一个可选的参数列表arg1，arg2，...作为参数传递给被调用的函数。它返回一个与thisArg绑定的新函数。

以下代码创建一个绑定函数并稍后调用它：
```javascript
function multiply(number) {
  'use strict';
  return this * number;
}
// create a bound function with context
var double = multiply.bind(2);
// invoke the bound function
console.log(double(3));  // => 6
console.log(double(10)); // => 20
```

multiply.bind(2)返回一个新的函数对象double，它与数字2绑定。multiply和double具有相同的代码和范围。

与.apply()和.call()方法（见5.）相反，它立即调用该函数,
.bind()方法只会返回一个新的函数，它应该在稍后使用预先配置的函数调用。

#### 6.1绑定函数中的this
> this 是调用绑定函数.bind()时的第一个参数

bind()的作用是创建一个新的函数，该函数将把上下文作为第一个参数传递给.bind()。这是一种功能强大的技术，可以使用预定义的值创建函数。
![](https://raw.githubusercontent.com/leegaox/notes/javascript/pics/6_1.png)

让我们来看看如何配置一个绑定函数：

```javascript
var numbers = {
  array: [3, 5, 10],
  getNumbers: function() {
    return this.array;
  }
};
// Create a bound function
var boundGetNumbers = numbers.getNumbers.bind(numbers);
console.log(boundGetNumbers()); // => [3, 5, 10]
// Extract method from object
var simpleGetNumbers = numbers.getNumbers;
console.log(simpleGetNumbers()); // => undefined or throws an error in strict mode
```
numbers.getNumbers.bind(numbers)返返回一个绑定numbers对象的函数boundGetNumbers。然后用this作为numbers对象调用boundGetNumbers（）并返回正确的数组对象。

函数numbers.getNumbers可以被提取到一个变量simpleGetNumbers中而无需绑定。在稍后的函数调用simpleGetNumbers()将this作为window或在严格模式下的undefined,但不是numbers对象 (see 3.2. Pitfall)。
在这种情况下，simpleGetNumbers（）将不会正确返回数组。

#### 6.2 紧密的上下文绑定

  .bind（）创建一个永久的上下文链接（permanent context link），并将始终保留它。当使用.call()或.apply()传入一个不同的上下文时，绑定函数不能更改其链接的上下文，或者即使反弹也没有任何效果。

  只有绑定函数的构造函数调用才能改变，但这不是推荐方法（对于构造函数调用使用普通函数，而不是绑定函数）。

  以下示例创建一个绑定函数，然后尝试更改其已经预定义的上下文：

```javascript
function getThis() {
  'use strict';
  return this;
}
var one = getThis.bind(1);
// Bound function invocation
one(); // => 1
// Use bound function with .apply() and .call()
one.call(2);  // => 1
one.apply(2); // => 1
// Bind again
one.bind(2)(); // => 1
// Call the bound function as a constructor
new one(); // => Object
```
只有new one()改变了绑定函数的上下文，其他类型的调用总是等于1。

### 箭头函数
Arrow函数旨在以较短的形式声明函数，并在词汇上（[lexically](https://en.wikipedia.org/wiki/Scope_(computer_science)#Lexical_scoping)）绑定上下文。

它可以使用以下方式：

```javascript
var hello = (name) => {
  return 'Hello ' + name;
};
hello('World'); // => 'Hello World'
// Keep only even numbers
[1, 2, 5, 6].filter(item => item % 2 === 0); // => [2, 6]
```
除了详细的关键字功能之外，箭头函数会带来更轻的语法。当函数只有1条语句时，您甚至可以省略返回。

箭头功能是匿名（[anonymous](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/name)）的，这意味着name属性是一个空字符串''。这样它就没有词法函数名称（这对于递归，分离事件处理程序很有用）。

它也不提供[arguments](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/arguments)对象，这与常规功能相反。但是，这被使用ES2015 [rest parameters](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/rest_parameters)修复：
```javascript
var sumArguments = (...args) => {
   console.log(typeof arguments); // => 'undefined'
   return args.reduce((result, item) => result + item);
};
sumArguments.name      // => ''
sumArguments(5, 5, 6); // => 16
```

#### 7.1 箭头函数中的this
> this 是定义箭头函数的封闭上下文

箭头函数不会创建它自己的执行上下文，而是从定义它的外部函数中获取它。
![](https://raw.githubusercontent.com/leegaox/notes/javascript/pics/7_1.png)

以下示例显示了上下文透明度属性：
```javascript
class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }
  log() {
    console.log(this === myPoint); // => true
    setTimeout(()=> {
      console.log(this === myPoint);      // => true
      console.log(this.x + ':' + this.y); // => '95:165'
    }, 1000);
  }
}
var myPoint = new Point(95, 165);
myPoint.log();
```
setTimeout使用与log()方法相同的上下文（myPoint对象）调用箭头函数。如所看到的，箭头函数“从”定义它的函数“继承”上下文。

如果在本例中尝试使用常规函数，它会创建自己的上下文（window或在严格模式下的undefined）。因此，为了使相同的代码正确地使用函数表达式，需要手动绑定上下文：setTimeout(function() {...}.bind(this)).这是冗长的，使用箭头功能是一个更清洁和更短的解决方案。
如果箭头函数被定义在最上面的范围（在任何函数之外），上下文始终是全局对象（浏览器中的窗口window）：
```javascript
var getContext = () => {
   console.log(this === window); // => true
   return this;
};
console.log(getContext() === window); // => true
```

一个箭头函数永远与词法（lexical）上下文绑定。即使使用上下文修改方法，也不能修改：

```javascript
var numbers = [1, 2];
(function() {
  var get = () => {
    console.log(this === numbers); // => true
    return this;
  };
  console.log(this === numbers); // => true
  get(); // => [1, 2]
  // Use arrow function with .apply() and .call()
  get.call([0]);  // => [1, 2]
  get.apply([0]); // => [1, 2]
  // Bind
  get.bind([0])(); // => [1, 2]
}).call(numbers);
```
函数表达式使用.call（numbers）间接调用，这使得这个调用成为numbers。get 箭头函数的this也是numbers，因为它从词汇的角度出发。

无论get怎样调用，箭头函数总是保持初始上下文numbers。使用其他上下文的间接调用get.call([0])或者 . get.apply([0])
，重新绑定get.bind([0])() 都没有作用。

箭头函数不能用作构造函数。如果将其作为构造函数new get()调用，JavaScript会抛出一个错误：TypeError: get is not a constructor。

### 7.2 陷阱：用箭头函数定义方法

**注意：**您可能想使用箭头函数来声明对象的方法.足够公平：与函数表达式相比，它们的声明非常短：（param）=> {...}而不是函数（param）{..}。

本示例使用箭头函数定义类Period上的方法format()：

```javascript
function Period (hours, minutes) {
  this.hours = hours;
  this.minutes = minutes;
}
Period.prototype.format = () => {
  console.log(this === window); // => true
  return this.hours + ' hours and ' + this.minutes + ' minutes';
};
var walkPeriod = new Period(2, 30);
walkPeriod.format(); // => 'undefined hours and undefined minutes'
```
由于format是一个箭头函数，并且在全局上下文（最上面的范围）中定义，所以它将this作为window对象。
即使format作为对象walkPeriod.format()的方法执行，window仍保留为调用的上下文。发生这种情况是因为箭头函数的静态上下文在不同的调用类型中不会改变。
this 是 window，因此this.hours 和 this.minutes 是undefined。该方法返回字符串：'undefined hours and undefined minutes'，这不是预期的结果。

函数表达式解决了这个问题，因为常规函数会根据调用改变其上下文：
```javascript
function Period (hours, minutes) {
  this.hours = hours;
  this.minutes = minutes;
}
Period.prototype.format = function() {
  console.log(this === walkPeriod); // => true
  return this.hours + ' hours and ' + this.minutes + ' minutes';
};
var walkPeriod = new Period(2, 30);
walkPeriod.format(); // => '2 hours and 30 minutes'
```
walkPeriod.format() 是一个对象上的方法调用（见3.1.），，其上下文walkPeriod对象。this.hours为2，this.minutes为30，所以该方法返回正确的结果：“2小时30分钟”。

### 8. 结论
因为函数调用对this有最大的影响，所以从现在开始不要问自己：
> this 从哪里来？

但是请问自己：
> 函数是如何被调用的？

对于箭头功能问问自己：
> this 在箭头函数被定义的地方代表什么？

在处理这个问题时，这种思维方式是正确的，并且可以帮助您避免头痛。

传播关于JavaScript的知识并分享这篇文章，你的同事们会感激。

Don't lose your context ;)
