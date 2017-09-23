## location
- location既是window的属性又是document的属性
- location.reload()，若有浏览器缓存，从缓存加载重新加载；若加参数true则从服务器加载

## 浏览器检测
- 能力检测：检测浏览器是否包含某种功能
- 怪癖检测：检测浏览器是否包含某种缺陷（bug）
- 用户代理检测：检测navigator.userAgent，优先级低于前两者，因为有浏览器进行电子诈骗（spoofing）仿照其他浏览器.

## 常用跨域手段
- cors：在请求头中加入Origin包含（协议、域名、端口），服务端相应头中加入相应Access-Control-Allow-Origin
- 图像ping：利用`<img>`的src属性，利用onload和onerror事件处理函数，得知响应。缺点：只能发GET请求、无法访问服务器响应文本  
- JSONP:利用`<script>`的src进行请求，再利拿到的json进行回调函数的处理，缺点：无法监听是否失败，只能通过自己增加定时器去判断；由于跨域容易被夹带恶意脚本，难以确保安全


## Comet（服务器推送）

ajax是浏览器主动请求，Comet是服务器主动推送的概念，实现comet 主要有两种：长轮询 、 http流

- 长轮询：是短轮训的翻版，短轮训是浏览器端不停向服务端询问是否更新，长轮询是页面发送一个请求，然后连接一直保持，直到数据接受，再关闭连接，重新发起新请求，周而复始

- http流：整个生命周期只使用一个请求，保持连接周期性发送请求到浏览器

```javascript
    function createStreamingClient(url,progress,finished){
        var xhr = new XMLHttpRequest()
        var received = 0;
        xhr.open('get',url,true)
        xhr.onreadytstatechange = function(){
            var result
            if (xhr.readyState == 3){
                result = xhr.responseText.substring(received)
                received += result.length
                progress(result)
            }
            else if (xhr.readyState == 4) {
                finished(xhr.responseText)
            }
        }
    }

```
