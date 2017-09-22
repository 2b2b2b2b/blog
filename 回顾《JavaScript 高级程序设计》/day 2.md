# ECMA对象中的数据属性和访问器属性
- 数据属性
    - [[Configurable]]，能否通过delete删除属性从而重新定义，或者能否把属性修改为访问器属性，默认值true
    - [[Enumerabel]]，能否通过for-in遍历访问，默认true
    - [[Writable]],定义能否修改属性，默认true
    - [[Value]],包含这个属性的数据的位置，默认为Undefinded
    - 修改属性默认属性必须使用Object.defineProperty()方法
- 访问器属性
    - [[Configurable]]，能否通过delete删除属性从而重新定义，或者能否把属性修改为访问器属性，默认值true
    - [[Enumerabel]]，能否通过for-in遍历访问，默认true
    - [[Get]]，读取属性时候调用的函数,默认undefinded
    — [[Set]],写入属性时候调用的函数

- 构造函数的操作符new做了什么
    1. 创建新的临时对象
    2. 将构造函数作用域赋给新对象 （把构造函数原型给新对象）
    3. 执行构造函数中的代码
    4. return出临时对象

## 判断属性是实例私有还是原型
- hasOwnProperty
- 若有属性无论在私有还是原型上 in 操作符 皆为true

## 继承
- 组合继承，组合原型链和借用构造函数继承
```javascript
function SuperType(name){
    this.name= name
    this.colors = ['red','blue', 'yellow']
}
SuperType.prototype.sayName = function(){
    console.log(this.name)
}
function SubType(name,age){
    SuperType.call(this,name)
    this.age = age
}
SubType.prototype = new SuperType()
```

- 原型式继承,在不是很麻烦的情况下，不需要创建构造函数，即可用原型式继承,
但是所有引用类型都会共享相同值,Object.create()ie9+支持
```javascript
var person = {
    name:'hello',
    friends:['a','b','c']
}
var anotherPerson = Object.create(person)
anotherPerson.name='haha'
anotherPerson.friends.push('d')

var yetAnotherPerson = Object.create(person)
yetAnotherPerson.name='hehe'
yetAnotherPerson.friends.push('e')
console.log(anotherPerson.name)//'haha'
console.log(yetAnotherPerson.name)//'hehe'
console.log(person.friend)//'['a','b','c','d','e']'

```

- 寄生组合式继承，组合集成商JS中最常用的继承，但是它会调用两次超类构造函数。寄生组合式继承只需要调用一次超累，并且避免创建不必要属性。同时还能保持原型链。寄生组合式继承是引用类型最理想的继承范式
```javascript
function inheritPrototype(subType,superType){
    var prototype = Object(superType.prototype) //创建对象
    prototype.constructor = subType//增强对象
    subType.prototype = prototype//指定对象
}

function SuperType = (name){
    this.name = name ;
    this.colors = ['red','blue']
}
SuperType.prototype.sayName = function(){
    console.log(name)
}
function Subtype(name,age) {
    SuperType.call(this,name)
    this.age = age
}
inheritPrototype(Subtype,SuperType)
Subtype.prototype.sayAge = function{
    console.log(this.age)
}
```
