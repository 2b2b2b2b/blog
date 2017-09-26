## Yields Process
浏览器在操作系统中只分配了一定量的资源，所以在执行长时间的语句或循环遍历时要特别注意。因此我们可以利用‘分块处理的思想去解决’。利用setTimeout把数组分成模块进入待办事宜，从而避免阻塞
```javascript
function chunk(array , process , context){
    setTimeout(()=>{
        var item = array.shift()
        process.call(context,item)
        if(array.length > 0){
            setTimeout(argument.callee,100)
        }
    },100)
}
```

## 函数节流
在浏览器中某些计算处理的陈本相当昂贵，例如resize操作，函数节流的思想就是某些操作不可以没有间断的反复执行，利用定时器在一定时间内执行，第二次调用清楚之前定时器，设置另外个，以此反复执行
```javascript
function throttle(method , context){
    clearTimeout(method.tid)
    method.tid = setTimeout(()=>{
        method.call(context)
    })
}
```
