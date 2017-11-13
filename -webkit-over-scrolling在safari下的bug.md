这周五调试了一个相当隐蔽的bug，大概是`-webkit-overflow-scrolling:touch`在safari下面的bug，也有可能其他浏览器也有，未做考证

bug 描述： 容器在页面初始化的时候未拿到子元素的高度（子元素是通过ajax后添加进去），随后设置`-webkit-overflow-scrolling:touch`页面整体滑动，而不是父容器中的内容上下滑动。

IOS版本：10.1.1

```html
  <!--大致布局-->
  <style >
    .wapper{
      height: 100%;
      position: relative;
    }
    header,footer{
      height: 200px;
      position: absolute;
      left: 0;
      z-index: 999;
    }
    header{
      top: 0;
    }
    footer{
      bottom: 0;
    }
    main{
      height: 100%;
      padding:200px 0 ;
      overflow: auto;
      -webkit-overflow-scrolling:touch;
    }
  </style>
  <div class="wapper">
    <header></header>
    <main>
      <!--中间内容通过请求后面加入-->
    </main>
    <footer></footer>
  </div>
```

解决办法： main中后加载的内容外层包裹一层div加个属性`min-height:101%`,是的页面一开始就知道内部容器溢出需要滚动，而不是滚动全屏。

调试了整整一天，希望记录下有更多的人不要踩到这个坑~
