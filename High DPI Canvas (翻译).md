
> 翻译于https://www.html5rocks.com/en/tutorials/canvas/hidpi/
>
> 转载请声明出处

# 介绍

高分屏大法好，高分屏使得所有东西显示起来都更清晰更加平滑。但是对于苦逼开发者来说却也相应的带来了一系列的新问题。在这篇文章下我们要研究一下：在高分屏下的canvas画图片所带来的问题。

# 设备像素比属性

在很早以前还没高分屏的时候，开发写的的1像素也就是实际的1像素（如果不考虑缩放的情况），你根本不需要做什么特殊的处理。如果css中设置100px那他就是100px。后来出现了高分屏的手机，并且在window对象下面出现了devicePixelRatio 这个神秘的属性，并且还可以用devicePixelRatio在媒体查询中进行判断。这个属性的意思就是：渲染时，css中的像素（逻辑像素）候和实际像素（物理像素）的比值。比如说：iPhone 4S它的devicePixelRatio 属性的值是2，那就是100px逻辑像素等于200px的设备实际像素。

这对我们开发人员意味着什么呢？也就是我们的图像在高分屏上被放大了。在高分屏上，如果我们插入图片时的宽度是根据逻辑像素来设置，渲染的时候图片会根据devicePixelRatio 放大了数倍，当然图片也会变得很模糊。

解决这额个问题的方法就是，创建的图片的时候根据devicePixelRatio 放大数倍（比原照片更大的新的一张图片）然后再用css再把它缩小到原来的样子。因此缩小后的图片不会超过自己原来的尺寸并且不会再模糊。这样一来问题就解决了（当然这又带来了一个新的问题，你要根据不同的DPI设备获取不同的图片）

# 介绍backing store

什么是canvas？这是我们这篇文章主要讨论的问题，接下来我们要做的实验主要体现Chrome和Safari 6之间的差异。有一个新前缀特性在不同的浏览器canvas上下文中：
webkitBackingStorePixelRatio，目前 Mozilla、Opera 、 Microsoft没有这个属性，以后可能会有。

这个属性告诉我们目前浏览器的backing store 和canvas元素之间的关系。你肯定已经想问了backing store是神马？在你用canvas画任何图像的时候，实质上浏览器都会将这些图形存储在canvas的底层存储层，这个我们就称之为backing store 。当浏览器开始用canvas画图的时候都是从backing store 中拿数据再进行绘制。这就意味着有了这个属性我们可以知道backing store的尺寸和canvas尺寸的关系。你肯定想问为什么去用devicePixelRatio的值去确定backing store的尺寸，答案是浏览器不能保证devicePixelRatio 的值出来是对的。即使在Chrome 和 safari 6中两者的devicePixelRatio都是一样的，但是两者对于canvas的绘制方法不同(也就是 webkitBackingStorePixelRatio不同)。最终结论就是我们不能去用devicePixelRatio计算浏览器会放大图片多少倍。

好！现在我们知道了webkitBackingStorePixelRatio是个啥，但是我们还要知道怎么去用。为了简单起见，比方说我们有个canvas他的宽度是200px并且它的webkitBackingStorePixelRatio的值是2.因此底层的backing store尺寸就是400px。要注意的是不同的浏览器在不同的设备上webkitBackingStorePixelRatio的值都不同，不一定是2。
当浏览器开始渲染canvas，它先被比例缩小根据他的逻辑像素200px（之前的假设）。然后再根据devicePixelRatio再放大，假设devicePixelRatio 也是2和webkitBackingStoreRatio相同，那就是400px。（devicePixelRatio 也不一定是2，三星Nexus 7就是1.325），可能你还会有疑惑这个过程，看一下下图：

现在我们知道devicePixelRatio 和 webkitBackingStoreRatio的作用过程，但我们还要进一步看看他们实施的差异。

# 实施差异
比如说在Macbook Pro Rentina 高分屏上，Safari 6的devicePixelRatio 和 webkitBackingStoreRatio的值分别是2和2。但是chrome是2和1。这意味着如果你在canvas上画图Safari将会自动的把你的图片尺寸变成两倍存储在backing store里，
然后又缩小到逻辑像素然后通过devicePixelRatio又放大和backing store里的尺寸一样。但在chrome中图像直接根据逻辑像素尺寸存到backing store中，但在之后devicePixelRatio作用后图片将会被放大因此变模糊了。一图胜前言，下面的图显示不同浏览器里的结果：

在默认的chrome中图像渲染会变得模糊相较于Safari 6，因为我们的图片被不同的尺寸写进backing store ，然后通过devicePixelRatio进行了放大。

这就带来两个问题：
* chrome为什么没和Safari一样，在存到 backing store 中自动放大尺寸。
* 怎么使得开发者能够确保图片能够和设想的一样进行渲染

让我们先来解决第一个问题，之前我们提过webkitBackingStoreRatio的值不一定是2，可能是任何数字，可能超过2。但是如果你认为backing store的比例是2，这就代表着图像存到 backing store中宽高都放大了2倍，并且用了4倍的内存来存储这个canvas元素。如果你的页面有大量的canvas，或者你的内存大小有限比如手机上更可能内存吃紧。是否要自动的去放大存储canvas到backing store这完全取决于你自己。

还有个问题留给我们的是我们怎么能够手动的去放大你的canvas。这个问题的答案其实很简单：通过devicePixelRatio和 webkitBackingStorePixelRatio的比值去放大canvas的height和width然后再用css把他们缩小到原来的逻辑像素尺寸。举个例子：在Chrome浏览器上devicePixelRatio和 webkitBackingStorePixelRatio的值分别是2和1，我们把canvas元素的width和height属性放大两倍，然后用css去缩小到一半。

还有最后个想要说明的是，由于我们手动放大完画布（并用css缩小后），我们必须确保按我们的图片也按比例去放大高度、宽度、和位置。

-----
示例代码：
```
function drawImage(opts) {

    if(!opts.canvas) {
        throw("A canvas is required");
    }
    if(!opts.image) {
        throw("Image is required");
    }

    // get the canvas and context
    var canvas = opts.canvas,
        context = canvas.getContext('2d'),
        image = opts.image,

    // now default all the dimension info
        srcx = opts.srcx || 0,
        srcy = opts.srcy || 0,
        srcw = opts.srcw || image.naturalWidth,
        srch = opts.srch || image.naturalHeight,
        desx = opts.desx || srcx,
        desy = opts.desy || srcy,
        desw = opts.desw || srcw,
        desh = opts.desh || srch,
        auto = opts.auto,

    // finally query the various pixel ratios
        devicePixelRatio = window.devicePixelRatio || 1,
        backingStoreRatio = context.webkitBackingStorePixelRatio ||
                            context.mozBackingStorePixelRatio ||
                            context.msBackingStorePixelRatio ||
                            context.oBackingStorePixelRatio ||
                            context.backingStorePixelRatio || 1,

        ratio = devicePixelRatio / backingStoreRatio;

    // ensure we have a value set for auto.
    // If auto is set to false then we
    // will simply not upscale the canvas
    // and the default behaviour will be maintained
    if (typeof auto === 'undefined') {
        auto = true;
    }

    // upscale the canvas if the two ratios don't match
    if (auto && devicePixelRatio !== backingStoreRatio) {

        var oldWidth = canvas.width;
        var oldHeight = canvas.height;

        canvas.width = oldWidth * ratio;
        canvas.height = oldHeight * ratio;

        canvas.style.width = oldWidth + 'px';
        canvas.style.height = oldHeight + 'px';

        // now scale the context to counter
        // the fact that we've manually scaled
        // our canvas element
        context.scale(ratio, ratio);

    }

    context.drawImage(pic, srcx, srcy, srcw, srch, desx, desy, desw, desh);
}
```

总结：目前市场上的设备越来越多，他们的设备屏幕比例也各不相同。了解浏览器如何通过backing store的像素比例和设备像素比（dpr）来控制你的图片和canvas是保证图片质量和清晰的保证。
