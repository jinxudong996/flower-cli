## JavaScript对象属性及创建

> 对象就是一组属性的无序集合

###### 属性

> 属性分为数据属性和访问器属性

数据属性主要有四个特性

- Configurable    表示属性是否可以通过delete删除并重新定义，是否可以修改它的特性，以及是否可以把它改为访问器属性。默认为true
- Enumerable  表示属性是否可以通过for-in循环返回。默认为true
- Writable  表示属性的值是否可以被修改。默认为true
- Value  包含属性实际的值。默认为undefined

修改属性的默认特性，得使用Object.defineProperty()，该方法接收三个参数：要给其添加属性的对象、属性的名称和描述符对象。

```JavaScript
let person = {}
Object.defineProperty(person, 'name', {
    writable: false,
    value: 'nick'
})
console.log(person.name)  // nick
person.name = 'tom'
console.log(person.name)  // nick
```

访问器属性也有四个特性描述他们的行为：

- configured  表示属性是否可以通过delete删除并重新定义，是否可以修改它的特性，以及是否可以把它改为访问器属性。默认为true
- Enumerable  表示属性是否可以通过for-in循环返回。默认为true
- get  获取函数，在读取属性值时调用，默认为undefined
- set   设置函数，在写入属性时调用，默认为undefined

访问器属性也要通过Object.defineProperty()函数来修改。

```JavaScript
let book = {
    year_: 2021,
    edition: 1
}
Object.defineProperty(book, 'year', {
    get(val) {
        console.log('getting', val)
        return this.year_
    },
    set(val) {
        console.log('setting---',val)
        //this.year_ = val
        this.edition += 1
    }
})
book.year = 2050   // setting--- 20250
console.log(book.year_)  //2021
console.log(book.year)  // getting undefined  2021 
console.log(book.edition  2
```

加入需要在同一个对象上定义多个属性，则需要使用Object.defineProperties()方法，该方法接收两个参数，要修改的对象和描述符对象，用法同Object.defineProperty()一致。vue的v-model双向绑定就是利用Object.defineProperty()方法的get与set方法来实现的。



Object.assign()，该方法接收一个目标对象和一个或多个源对象作为参数，将每个源对象中可枚举属性（）和自有属性复制到目标对象上。该方法的原理就是，对每个符合条件的属性，调用源对象上的get方法取得属性值，使用目标对象的set方法设置属性值。该方法只是浅复制。



> 对象解构：使用对象匹配的结构来完成复制

```javascript
let person = {
    name: 'nick',
    age: 18
}
let { name, age} = person  //解构赋值
let {name, job} = person //如果没匹配到，就是undefined
let {name, job='singer'} = person  //设置默认值
```

###### 创建对象

- 工厂模式

  ```JavaScript
  function createPerson(name, age) {
      let o = new Object()
      o.name = name
      o.age = age
      o.sayHi = function() {
          console.log('hi')
      }
      return o
  }
  let person = createPerson('nick', 18)
  ```

  通过创建一个object实例，并为新实例添加属性和方法，这种工厂模式虽然可以解决创建多个类似对象的问题，但没有解决对象标识问题，即新创建的对象是什么类型。

- 构造函数模式

  ```JavaScript
  function CreatePerson(name, age) {
      this.name = name
      this.age = age
      this.sayHi = function() {
          console.log('hi')
      }
  }
  let person = new createPerson('nick', 18)
  ```

  使用new操作符，调用CreatePerson构造函数来创建实例，需要执行如下五个步骤：

  1. 在内存里创建一个新对象
  2. 这个新对象内部的Prototype特性被赋值为构造函数的prototype属性
  3. 构造函数内部的this被赋值为新的实例
  4. 执行构造函数内部的代码，为新实例添加属性和方法
  5. 如果构造函数返回非空对象，则返回此对象；否则就返回新实例

  新实例person有一个constructor属性指向构造函数CreatePerson，也可以借此来标识对象，通过instanceof操作符来确定对象类型。

  构造函数模型很好的解决了对象标识问题，但在构造函数内部定义的方法，每次创建新实例都会重新创建一遍。

  ```javascript
  //CreatePerson里定义的方法
  this.sayHi = function() {console.log('hi')}
  this.sayHi = new Function(console.log('hi'))
  person1.sayHi === person2.sayHi  //false
  ```

  两种方式在逻辑上是等价的，也就是说在不同的实例上，创建的方法虽然同名却不相等。做着同样的事情，就没有必要定义两个不同的Function实例，这个问题可以通过原型模式来解决。

- 原型模式

  每个函数都会创建一个prototype属性，这个属性称为原型对象，包含着特定引用类型的实例共享的属性和方法。

  ```javascript
  function CreatePerson(name, age) {
      this.name = name
      this.age = age
  }
  CreatePerson.prototype.sayHi = function() {
      console.log('hi')
  }
  let p1 = new CreatePerson('nick', 18)
  let p2 = new CreatePerson('tom', 19)
  p1.sayHi === p2.sayHi //true
  CreatePerson.prototype.constructor === CreatePerson //true
  ```

  自定义构造函数时，原型对象默认只会获得constructor属性，其他所有方法都继承自Object。每次调用构造函数创建一个新实例，该实例内部prototype就会指向构造函数的原型对象。在Firefox、Safari和Chrome浏览器里会为每个对象暴露_proto_属性（JavaScript高级程序设计里这样说的，然而在nodeJs里，实例也有该属性），该属性就指向对象的原型。在其他实现中，对象的原型就被完全隐藏了。

  这里就涉及到了三个对象，实例、构造函数和原型对象。实例与构造函数之间没有直接的联系，实例与原型对象之间通过_proto__ _属性有着直接的联系。

  ```javascript
  CreatePerson.prototype.isPrototypeOf(p1)  //true
  Object.getPrototypeOf(p1) == CreatePerson.prototype  //true
  ```

  原型对象上有个内置的isPrototypeOf方法，可以检查是否是由构造函数创建的实例，和instanceof作用一样。object上的内置的getPrototypeOf方法，可以返回实例的原型对象。

  ###### 对象迭代

  ES7新增了两个静态方法Object.values()和Object.entries()。Object.values()方法用于返回对象值的数组，Object.entries()返回键值对的数组。

  

```JavaScript
const o = {
    name:'nick',
    age: 15
}
console.log(Object.values(o)) // ["nick", 15]
console.log(Object.entries(o))  //["name", "nick"] ["age", 15]
```

###### Object()方法汇总 ---MDN

- Object.prototype.constructor     返回Object构造函数的引用

- Object.prototype.__proto__   已从web标准中删除，但很多浏览器支持。访问器属性，指向对象的原型

- Object.prototype.__define__Getter() 已从web标准中删除，但很多浏览器依然支持。将一个函数绑定在当前对象的指定属性上，当那个属性的值被读取时，你所绑定的函数就会被调用。

- Object.prototype.__define__Setter() 已从web标准中删除，但很多浏览器依然支持。`将一个函数绑定在当前对象的指定属性`上，当那个属性被赋值时，你所绑定的函数就会被调用。

- Object.prototype.__lookupGetter__() 已从web标准中删除，但很多浏览器支持。返回当前对象上指定属性的属性读取访问器函数

- Object.prototype.__lookupSetter__() 已从web标准中删除，但很多浏览器支持。返回的 setter 钩子的函数名

- Object.asign() 用于将所有可枚举属性的值从一个或多个源对象分配到目标对象。它将返回目标对象。是对象浅拷贝的一种方法。

- Object.create()创建一个新对象，使用现有的对象来提供新创建的对象的__proto__ 。可以代替Object.prototype.__proto__的使用

  ```javascript
  const person = {
      isHuman: false,
      printIntroduction: function() {
          console.log(`My name is ${this.name}. Am I human? ${this.isHuman}`);
      }
  };
  const me = Object.create(person);
  console.log(me.isHuman) //false
  console.log(me.__proto__ === person)  //true
  
  ```

- Object.defineProperties()方法直接在一个对象上定义新的属性或修改现有属性，并返回该对象。

- Object.defineProperty()方法会直接在一个对象上定义一个新属性，或者修改一个对象的现有属性，并返回此对象。

- Object.entries()方法返回一个给定对象自身可枚举属性的键值对数组，其排列与使用for of循环遍历该对象时返回的顺序一致（区别在于 for-in 循环还会枚举原型链中的属性）

-  Object.freeze()方法可以**冻结**一个对象。一个被冻结的对象再也不能被修改；冻结了一个对象则不能向这个对象添加新的属性，不能删除已有属性，不能修改该对象已有属性的可枚举性、可配置性、可写性，以及不能修改已有属性的值。此外，冻结一个对象后该对象的原型也不能被修改。freeze() 返回和传入的参数相同的对象。

- Object.fromEntries()方法把键值对列表转换为一个对象

- Object.getOwnPropertyDescriptor()方法返回指定对象上一个自有属性对应的属性描述符。（自有属性指的是直接赋予该对象的属性，不需要从原型链上进行查找的属性），同Object.prototype.__lookupGetter__()用法一样。

- Object.getOwnPropertyDescriptors()方法用来获取一个对象的所有自身属性的描述符。同Object.prototype.__lookupSetter__()用法一致。

- Object.getOwnPropertyNames()方法返回一个由指定对象的所有自身属性的属性名（包括不可枚举属性但不包括Symbol值作为名称的属性）组成的数组。

- Object.getOwnPropertySymbols()方法返回一个给定对象自身的所有 Symbol 属性的数组

- Object.getPrototypeOf()方法返回指定对象的原型（内部[[Prototype]]属性的值）

- Object.prototype.hasOwnProperty()方法会返回一个布尔值，指示对象自身属性中是否具有指定的属性（也就是，是否有指定的键）。

- Object.is() 方法判断两个值是否为同一个值

- Object.isExtensible()方法判断一个对象是否是可扩展的（是否可以在它上面添加新的属性）

- Object.isFrozen()方法判断一个对象是否被冻结

- Object.prototypeObj.isPrototypeOf()方法用于测试一个对象是否存在于另一个对象的原型链上

- Object.isSealed()方法判断一个对象是否被密封。

- Object.keys()方法会返回一个由一个给定对象的自身可枚举属性组成的数组，数组中属性名的排列顺序和正常循环遍历该对象时返回的顺序一致 

- Object.preventExtensions()方法让一个对象变的不可扩展，也就是永远不能再添加新的属性。

- Object.propertyIsEnumerable()方法返回一个布尔值，表示指定的属性是否可枚举。

- Object.seal()方法封闭一个对象，阻止添加新属性并将所有现有属性标记为不可配置。当前属性的值只要原来是可写的就可以改变。

  > 冻结是对象所有的属性只读的，不可更改，封闭只是冻结新属性，已有的属性还是可以更改的。

- Object.setPrototypeOf()方法设置一个指定的对象的原型 ( 即, 内部[[Prototype]]属性）到另一个对象或null

- Object.prototype.toLocaleString()方法返回一个该对象的字符串表示。此方法被用于派生对象为了特定语言环境的目的（locale-specific purposes）而重载使用

- Object.prototype.toString()方法返回一个表示该对象的字符串。

- Object.prototype.valueOf()方法返回指定对象的原始值。

- Object.values()方法返回一个给定对象自身的所有可枚举属性值的数组，值的顺序与使用for...in循环的顺序相同 ( 区别在于 for-in 循环枚举原型链中的属性 )。

  

###### 对象遍历

1. for...in

   > for...in语句以任意顺序遍历一个对象出Symbol以外的可枚举属性
   >
   > for...of语句在可迭代对象，诸如Array，Map，Set，String，TypeArray，arguments等，在对象上创建一个迭代循环，调用自定义迭代狗子，并为每个不同属性的值执行语句。

   ```JavaScript
   function color() {
       this.name = 'nick'
       this.age = 18
   }
   color.prototype.getJob = function() {
       console.log('getting job')
   }
   
   let p1 = new color()
   
   for(let prop in p1){
       console.log('key: ' + prop + ', ' + 'value:' + p1[prop])
   }
   //打印结果
   key: name, value:nick
   key: age, value:18
   key: getJob, value:function() {
       console.log('getting job')
   }
   ```

   for...in可以遍历对象所有的属性和方法，包括原型上的方法。

2. Object.keys()

   ```JavaScript
   function color() {
       this.name = 'nick'
       this.age = 18
   }
   color.prototype.getJob = function() {
       console.log('getting job')
   }
   
   let p1 = new color()
   console.log(Object.keys(p1)) //["name", "age"]
   console.log(Object.getOwnPropertyNames(p1))//["name", "age"]
   ```

   数组中属性名的排列顺序和正常循环遍历该对象时返回的顺序一致。

3. Object.entries()和Object.getOwnPropertyNames()

   ```JavaScript
   function color() {
       this.name = 'nick'
       this.age = 18
   }
   color.prototype.getJob = function() {
       console.log('getting job')
   }
   
   let p1 = new color()
   
   console.log(Object.entries(p1))
   //["name", "nick"]
   //["age", 18]
   ```

   遍历该对象时返回的顺序一致（区别在于 for-in 循环还会枚举原型链中的属性）

4. Object.values()

   ```JavaScript
   function color() {
       this.name = 'nick'
       this.age = 18
   }
   color.prototype.getJob = function() {
       console.log('getting job')
   }
   
   let p1 = new color()
   
   console.log(Object.values(p1)) //nick 18
   ```

   

