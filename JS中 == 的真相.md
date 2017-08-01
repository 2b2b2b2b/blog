> 翻译于https://javascriptweblog.wordpress.com/2011/02/07/truth-equality-and-javascript/

> 转载声明出处


你是否也和新手一样，对于这样的问题很困惑：
```javascript

if ([0]) {
    console.log([0] == true); //false
    console.log(!![0]); //true
}

```

或者说这种：

```javascript

if ("potato") {
    console.log("potato" == false); //false
    console.log("potato" == true); //false
}

```

值得庆幸的是在浏览器上还有标准去遵循。有些作者告诉我们要尽量避免这种强制转换和不去这样写。
我希望说服你的是这种强制转换的特征是有利有弊的（至少应该去理解），而不是一味的去避免...

X的值是true吗 ? x等不等于y ? 关于真值和相等判断的问题在JavaScript中主要是三个核心的概念：条件表达式和操作符（if ,
&&, || ...）；近似等于（==）；严格等于（===）。然我们逐一去分析每个问题


# 条件表达式
在Javascript中所有的条件语句和操作符遵循了一致的强制转换的规则。我们就用`if`来举例子。
if语句中，表达式讲强制使用toBoolean将表达式转化成布尔值，在es5语法规范中定义了以下算法：

| 参数类型        | 结果           |
| ------------- |:-------------:|
| Undefined     | false |
| Null     | false     |
| Bollean |  原值（不转换）   |
| Number |  如果参数是+0、-0或者NaN则结果是false，其余结果都是true     |
| String | 如果字符串长度是0（空字符串）结果是false，不然则是true      |
| Object | true      |

在JS中使用这个公式来区分值是true（true,"abc",36,[1,2,3],{a:16}）,或者是false(false,0,"",null,undefined)

这下我们就知道了刚开始介绍的例子中`if([0])`进入了true的模块语句，因为数组是个Object，Object被强制转化成了true
这里有几个更多的例子，他们的结果可能比较令人惊奇，但是还是尊崇了以上所说的规则：
```javascript
var trutheyTester = function(expr){
    return expr ? "true":"false"
}
trutheyTester({})//"true",对象一直都是true
trutheyTester(false)//"false"
trutheyTester(new Boolean(false))//"true",这是个对象
trutheyTester("") //"false"
trutheyTester(new String(""))//"true",一个对象
trutheyTester(NaN)//"false"
trutheyTester(new Number(NaN))//"true"
```
# 等于操作符(==)
==这个操作符的判断相当的宽松。两边的值可能被认为相等即使它们的类型不一样。因此在执行比较值钱，==操作符会把一个或者两个值给转化为单一的类型进行比较（通常为数字）。
因此至少几个知名的JS大牛都推荐尽量去避免==操作符。
但是这种避免的策略让人感到很困扰，因为恐惧和逃避是知识的敌人，要做到对一个语言的正真掌握你必须去直面。另外避免==并不能让你逃离它的陷阱，因为在JS中强制转换是无处不在的，就像刚才我们所看到的例子一样，数组索引，以及各种连接等等。另外合理的使用==可以使代码更加简洁优雅，可读性更高。
无论如何，让我们看看ECMA如何定义 == 的工作方式。其实真的没有那么难以理解，只要记住undefined和null互相相等 然后大多数类型会被强制转换成数字，以方便比较：
| 参数X类型| 参数Y类型  |结果           |
| ------------- |:-------------:|:-------------:|
