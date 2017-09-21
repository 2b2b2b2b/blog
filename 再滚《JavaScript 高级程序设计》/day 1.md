## 具名参数与arguments
函数中arguments对象中的值会自动u反应到具名参数中。不过并不是说两者公用相同的内存空间，只是值会同步。

```javascript
    function testArgument(a){
    	a = 20
    	console.log(arguments[0])
    	arguments[0] = 10
    	console.log(a)
    }
    testArgument(2)
    // 20
    // 10
```

## 延长作用域链
- try-catch中的catch块
- with语句

```javascript
//with 语句用法，把width中的参数放到，作用域链上
var a, x, y;
var r = 10;

with (Math) {
  a = PI * r * r;
  x = r * cos(PI);
  y = r * sin(PI / 2);
}
//PI、cos、sin分別指向Math对象的PI 、cos和sin函数，不用在前面添加命名空间（Math.PI）
```

## 垃圾回收机制（GC）
函数中的变量都有自己的生命周期，局部变量只有在函数执行中存在的必要，函数执行完便可去释放局部变量；垃圾回收机制需要跟踪变量是否有用，并且打上标记，具体到实现，大致有两个常用策略：
- 标记清除：进入环境时给所有变量加上标记，去除环境中变量以及被引用的变量的标记，然后对标记的变量进行内存清除
- 引用计数：计算一个值被引用的次数，如果减少了引用就减1，如果到0说明变量无法访问，定时回收计数为0的变量；由于相互引用，以及老版本ie中dom为引用计数法，js对象为标记法，循环引用容易造成内存泄漏，因此引用计数法还是有一定的弊端
- 一旦数据不再使用，应设置为null，进行解除引用，提升性能

## Array字面量表达式bug

```javascript
console.log([,,])
// [undefinded,undefinded] 只创建两项，在IE9+、Firefox、Opera、Safari和Chrome中

// [undefinded,undefinded，undefinded] 只创建三项，在IE8以及更早版本
//强烈不推荐这种写法
```

## Array.length
Array.length不只是可读,可以通过给该属性赋值，修改该数组
```javascript
var array = [1,2,3]
array.length = 4 //array , [1,2,3,undefinded]
array.length = 3 //array , [1,2,3]
array[99] = 100 //array,[1,2,3,.....undefinded*96,100]
```

## 函数内部属性
- 函数内部自带arguments和this这两个属性
- arguments 的主要目的是保存函数参数，但是还有个callee属性，该属性指向拥有arguments对象的函数

```javascript
//阶乘函数普通写法
function factorial(num){
    if(num <= 1){
        return 1
    }
    else{
        num*factorial(num-1)
    }
}
console.log(factorial(5)) // 120，结果正常

var newFactorial = factorial;
factorial = function(){
    return 0
}
console.log(newFactorial(5)) //0;由于函数的执行与函数名紧紧耦合，出现了没有重新定义函数功能的情况

//消除解耦的写法
function factorial(num){
    if(num <= 1){
        return 1
    }
    else{
        num*arguments.callee(num-1)
    }
}
console.log(factorial(5)) //120
var newFactorial = factorial;
factorial = function(){
    return 0
}
console.log(newFactorial(5)) // 120,消除函数名与函数之间的解耦
```
