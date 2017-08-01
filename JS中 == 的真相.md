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
在Javascript中所有的条件语句和操作符遵循了一致的强制转换的规则。我们就用`if`来举例子
