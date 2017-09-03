日常开发中，我们经常用到this。例如用Jquery绑定事件时，this指向触发事件的DOM元素；编写Vue、React组件时，this指向组件本身。对于新手来说，常会用一种意会的感觉去判断this的指向。以至于当遇到复杂的函数调用时，就分不清this的真正指向。

本文将通过两道题去慢慢分析this的指向问题，并涉及到函数作用域与对象相关的点。最终给大家带来真正的理论分析，而不是简简单单的一句话概括。

相信若是对this稍有研究的人，都会搜到这句话：**this总是指向调用该函数的对象**。

然而箭头函数并不是如此，于是大家就会遇到如下各式说法：

1. 箭头函数的this指向外层函数作用域中的this。
2. 箭头函数的this是定义函数时所在上下文中的this。
3. 箭头函数体内的this对象，就是定义时所在的对象，而不是使用时所在的对象。

各式各样的说法都有，乍看下感觉说的差不多。废话不多说，凭着你之前的理解，来先做一套题吧（非严格模式下）。

```javascript
/**
 * Question 1
 */

var name = 'window'

var person1 = {
  name: 'person1',
  show1: function () {
    console.log(this.name)
  },
  show2: () => console.log(this.name),
  show3: function () {
    return function () {
      console.log(this.name)
    }
  },
  show4: function () {
    return () => console.log(this.name)
  }
}
var person2 = { name: 'person2' }

person1.show1()
person1.show1.call(person2)

person1.show2()
person1.show2.call(person2)

person1.show3()()
person1.show3().call(person2)
person1.show3.call(person2)()

person1.show4()()
person1.show4().call(person2)
person1.show4.call(person2)()
```
大致意思就是，有两个对象`person1`，`person2`，然后花式调用person1中的四个show方法，预测真正的输出。

你可以先把自己预测的答案按顺序记在本子上，然后再往下拉看正确答案。


***
***
正确答案选下：
```javascript
person1.show1() // person1
person1.show1.call(person2) // person2

person1.show2() // window
person1.show2.call(person2) // window

person1.show3()() // window
person1.show3().call(person2) // person2
person1.show3.call(person2)() // window

person1.show4()() // person1
person1.show4().call(person2) // person1
person1.show4.call(person2)() // person2
```

对比下你刚刚记下的答案，是否有不一样呢？让我们尝试来最开始那些理论来分析下。

`person1.show1()`与`person1.show1.call(person2)`好理解，验证了**谁调用此方法，this就是指向谁**。

`person1.show2()`与`person1.show2.call(person2)`的结果用上面的定义解释，就开始让人不理解了。

它的执行结果说明this指向的是window。那就不是所谓的定义时所在的对象。

如果说是外层函数作用域中的this，实际上并没有外层函数了，外层就是全局环境了，这个说法也不严谨。

只有**定义函数时所在上下文中的this**这句话算能描述现在这个情况。

`person1.show3`是一个高阶函数，它返回了一个函数，分步走的话，应该是这样：
```javascript
var func = person3.show()

func()
```
从而导致最终调用函数的执行环境是window，但并不是window对象调用了它。所以说，**this总是指向调用该函数的对象**，这句话还得补充一句：**在全局函数中，this等于window**。

`person1.show3().call(person2)` 与 `person1.show3.call(person2)()` 也好理解了。前者是通过person2调用了最终的打印方法。后者是先通过person2调用了person1的高阶函数，然后再在全局环境中执行了该打印方法。

`person1.show4()()`，`person1.show4().call(person2)`都是打印person1。这好像又印证了那句：**箭头函数体内的this对象，就是定义时所在的对象，而不是使用时所在的对象**。因为即使我用过person2去调用这个箭头函数，它指向的还是person1。

然而`person1.show4.call(person2)()`的结果又是person2。this值又发生改变，看来上述那句描述又走不通了。一步步来分析，先通过person2执行了show4方法，此时show4第一层函数的this指向的是person2。所以箭头函数输出了person2的name。也就是说，箭头函数的this指向的是**谁调用箭头函数的外层function，箭头函数的this就是指向该对象，如果箭头函数没有外层函数，则指向window**。这样去理解show2方法，也解释的通。

这句话就对了么？在我们学习的过程中，我们总是想以总结规律的方法去总结结论，并且希望结论越简单越容易描述就越好。实际上可能会错失真理。

下面我们再做另外一个相似的题目，通过构造函数来创建一个对象，并执行相同的4个show方法。
```javascript
/**
 * Question 2
 */
var name = 'window'

function Person (name) {
  this.name = name;
  this.show1 = function () {
    console.log(this.name)
  }
  this.show2 = () => console.log(this.name)
  this.show3 = function () {
    return function () {
      console.log(this.name)
    }
  }
  this.show4 = function () {
    return () => console.log(this.name)
  }
}

var personA = new Person('personA')
var personB = new Person('personB')

personA.show1()
personA.show1.call(personB)

personA.show2()
personA.show2.call(personB)

personA.show3()()
personA.show3().call(personB)
personA.show3.call(personB)()

personA.show4()()
personA.show4().call(personB)
personA.show4.call(personB)()
```
同样的，按照之前的理解，再次预计打印结果，把答案记下来，再往下拉看正确答案。

***
***
正确答案选下：
```javascript
personA.show1() // personA
personA.show1.call(personB) // personB

personA.show2() // personA
personA.show2.call(personB) // personA

personA.show3()() // window
personA.show3().call(personB) // personB
personA.show3.call(personB)() // window

personA.show4()() // personA
personA.show4().call(personB) // personA
personA.show4.call(personB)() // personB
```

我们发现与之前字面量声明的相比，show2方法的输出产生了不一样的结果。为什么呢？虽然说构造方法Person是有自己的函数作用域。但是对于person1来说，它只是一个对象，在直观感受上，它跟第一道题中的person1应该是一模一样的。 `JSON.stringify(new Person('person1')) === JSON.stringify(person1)`也证明了这一点。

说明构造函数创建对象与直接用字面量的形式去创建对象，它是不同的，构造函数创建对象，具体做了什么事呢？我引用红宝书中的一段话。

> 使用 new 操作符调用构造函数，实际上会经历一下4个步骤：
> 1. 创建一个新对象；
> 2. 将构造函数的作用域赋给新对象（因此this就指向了这个新对象）；
> 3. 执行构造函数中的代码（为这个新对象添加属性）；
> 4. 返回新对象。

所以与字面量创建对象相比，很大一个区别是它多了构造函数的作用域。我们用chrome查看这两者的作用域链就能清晰的知道:

![](https://user-gold-cdn.xitu.io/2017/9/2/4fe92965af4e9fd72fa2c65e031d934c)

![](https://user-gold-cdn.xitu.io/2017/9/2/b1a2a29391fe5324dd7876ab1b521b51)

personA的函数的作用域链从构造函数产生的闭包开始，而person1的函数作用域仅是global，于是导致this指向的不同。我们发现，要想真正理解this，先得知道到底什么是作用域，什么是闭包。

有简单的说法称闭包就是能够读取其他函数内部变量的函数。然而这是一种闭包现象的描述，而不是它的本质与形成的原因。

我再次引用红宝书的文字（便于理解，文字顺序稍微调整），来描述这几个点：

>...每个函数都有自己的执行环境（execution context，也叫执行上下文），每个执行环境都有一个与之关联的变量对象，环境中定义的所有变量和函数都保存在这个对象中。

>...当执行流进入一个函数时，函数的环境就会被推入一个环境栈中。当代码在环境中执行时，会创建一个作用域链，来保证对执行环境中的所有变量和函数的有序访问。函数执行之后，栈将环境弹出。

>...函数内部定义的函数会将包含函数的活动对象添加到它的作用域链中。

具体来说，当我们 `var func = personA.show3()` 时，`personA`的`show3`函数的活动对象，会一直保存在`func`的作用域链中。只要不销毁`func`，那么`show3`函数的活动对象就会一直保存在内存中。（chrome的v8引擎对闭包的开销会有优化）

而构造函数同样也是闭包的机制，`personA`的`show1`方法，是构造函数的内部函数，因此执行了 `this.show3 = function () { console.log(this.name) }`时，已经把构造函数的活动对象推到了show3函数的作用域链中。

我们再回到this的指向问题。我们发现，单单是总结规律，或者用一句话概括，已经难以正确解释它到底指向谁了，我们得追本溯源。

红宝书中说道：
>...this引用的是函数执行的环境对象（便于理解，贴上英文原版：It is a reference to the context object that the function is operating on）。
>...每个函数被调用时都会自动获取两个特殊变量：this和arguments。内部在搜索这个两个变量时，只会搜索到其活动对象为止，永远不可能直接访问外部函数中的这两个变量。


我们看下MDN中箭头函数的概念：
>一个箭头函数表达式的语法比一个函数表达式更短，并且不绑定自己的 `this`，`arguments`，`super`或 `new.target`。...箭头函数会捕获其所在上下文的 `this` 值，作为自己的 `this` 值。

也就是说，普通情况下，this指向调用函数时的对象。在全局执行时，则是全局对象。

箭头函数的this，因为没有自身的this，所以this只能根据作用域链往上层查找，直到找到一个绑定了this的函数作用域（即最靠近箭头函数的普通函数作用域，或者全局环境），并指向调用该普通函数的对象。

或者从现象来描述的话，即**箭头函数的this指向声明函数时，最靠近箭头函数的普通函数的this。但这个this也会因为调用该普通函数时环境的不同而发生变化。导致这个现象的原因是这个普通函数会产生一个闭包，将它的变量对象保存在箭头函数的作用域中**。

故而`personA`的`show2`方法因为构造函数闭包的关系，指向了构造函数作用域内的this。而

```javascript
var func = personA.show4.call(personB)

func() // print personB
```
因为personB调用了personA的show4，使得返回函数func的作用域的this绑定为personB，进而调用func时，箭头函数通过作用域找到的第一个明确的this为personB。进而输出personB。

讲了这么多，可能还是有点绕。总之，想充分理解this的前提，必须得先明白js的执行环境、闭包、作用域、构造函数等基础知识。然后才能得出清晰的结论。

**我们平常在学习过程中，难免会更倾向于根据经验去推导结论，或者直接去找一些通俗易懂的描述性语句。然而实际上可能并不是最正确的结果。如果想真正掌握它，我们就应该追本溯源的去研究它的内部机制。**

我上述所说也是我自己推导出的结果，即使它不一定正确，但这个推断思路跟学习过程，我觉得可以跟大家分享分享。



--[阅读原文](https://github.com/wuomzfx/blog/blob/master/this.md) @[相学长](https://www.zhihu.com/people/xiang-xue-zhang)

--转载请先经过本人授权。
