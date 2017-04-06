# 移动端适配初探

IE时代，每一个前端同学都要费尽心力兼容各代版本IE。如今IE虽渐渐淘汰，但移动互联网崛起，我们又有了新的麻烦。因为各个手机的屏幕不同，为了保证产品视觉呈现与体验更加完美和谐，移动端的适配也显得较为棘手。

目前较为知名的一个方案是手淘的flexible。它具体如何实现移动端的适配，我们后面再阐述，因为在此之前，我们需要先理清一些基础概念。

* **screen.width**
设备屏幕的宽度，以像素计。这个属性只读，不会因为浏览器的拖拉缩放而改变。

* **window.innerWidth**
浏览器窗口的内部宽度。它包含滚动条，并且会随着浏览器拖拉缩放而改变。具体表现为呈现出内容的浏览器窗口的宽度。

* **clientWidth**
元素的可见宽度（不计滚动条）。document.documentElement.clientWidth 即整个html的可见宽度。

* **offetWidth**
元素的实际宽度（包含滚动条）。document.documentElement.offsetWidth即整个html的实际宽度。

然后我们来看一个莫名其妙的事情。我手上是一只三星s7e，号称屏幕分辨率1440*2560。

现在我写了一个页面，body里设置了一个div，设置宽高为300px。并打印了刚阐述的那些宽度尺寸与div的宽度，没看到结果前，乍一想，应该是4个1440和一个300吧。然后屏幕蹦跶出这样的结果。

![](http://mmbiz.qpic.cn/mmbiz_jpg/wP62YcCWCXXQMNDWBuZMFY6UsEzeayALAhEQZCesXpiaquNte82bgcqrwwNGAtDficBMMOIO5JXc7n5lWwegFHkA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)
莫名其妙吧，跟想像中完全不一样啊。屏幕宽度咋成412px了，说好的2k屏呢？还有980px又是哪儿蹦达出来的。为了搞清这当中的奥秘，我们又得来学习一些概念了。

* **设备像素**
顾名思义，就是设备的真实物理像素，指每英寸屏幕所拥有的像素(pixel)数目。

* **设备独立像素**
device independent pixel，独立于设备的用于逻辑上衡量像素的单位。

* **CSS像素**
指的是CSS样式代码中使用的逻辑像素，比如上面那个div，我设置了宽度300px，就是CSS像素，它并不一定等于它真实的物理像素。所以CSS像素即是用在CSS语言中的设备独立像素。

* **设备像素比**
device pixel ratio，即设备像素/设备独立像素，即物理像素/逻辑像素。在网页中就是设备像素与CSS像素的比值。设备像素比能通过window.devicePixelRatio获取。

通过dpr我知道了s7e此比值为3.5。那么screen.width 就是这手机的独立像素的值，412*3.5，确实等于1442。

可是问题又来了，980px又是如何来的呢？为什么告诉我逻辑分辨率是412*731了，怎么html的宽度又是980呢？

## Viewport（视区）
Viewport可以理解成浏览器用来呈现网页的那一部分区域，它是一个移动端的特性，可以通过Meta标签来设定。

### Layout viewport
因为移动设备的逻辑分辨率一般都是比PC小的，为了能让移动设备也能比较好的呈现PC网页内容，各大移动端浏览器就把viewport的默认值设成了980px或者1024px。

也就是说，不管你设备屏幕多大，分辨率多少，浏览器窗口的逻辑宽度，就是设定的值（比如980px），如果一个元素宽度是980px，那就刚好占满屏幕宽度。如果html宽度超出980px，那窗口就出现滚动条。除非隐藏内容，或者缩放屏幕。

这个呈现网页的区域，就叫做layout viewport。于是，我做的页面出现了上述结果。（至于为什么会多了1px成了981，这个不同手机有不同结果，我还未探索到确切原因，猜测是分辨率不同导致处理策略不同特意设的偏差值。）

### Visual viewport
上面陈述到，layout viewport可能是会比浏览器可视区域大的，这个visual viewport指的就是可视的viewport区域。简单的理解，就是屏幕可视区域。可以在visual viewport上拖拉缩放，来查看layout viewport的内容。

### Ideal viewport
layout默认值是980px，可是我们很多页面就是专门为了移动端设计的，我们不希望用户要缩放、横屏滚动才能看到内容，而且内容还很小。最好viewport就是屏幕宽度，这样viewport就是ideal vieaport 。

### Viewport meta
我们希望可以自己设定viewport值，来呈现自己想呈现的内容，那就得通过meta标签来定义viewport属性，具体如下：

Name          | Value                  | Description
--------------|------------------------|----
width         | 正整数或device-width   | 定义视口的宽度，单位为像素
height        | 正整数或device-height  | 定义视口的高度，单位为像素
initial-scale | [0.0-10.0]             | 定义初始缩放值
minimum-scale | [0.0-10.0]             | 定义缩小最小比例，它必须小于或等于maximum-scale设置
maximum-scale | [0.0-10.0]             | 定义放大最大比例，它必须大于或等于minimum-scale设置
user-scalable | yes/no                 | 定义是否允许用户手动缩放页面，默认值yes

最常用的写法就是
```html 
<meta name="viewport" content="width=device‐width,
                               initial‐scale=1,
                               user-scalable=0">
```

意思就是，设置viewport视区大小为设备宽度，即理想视区，默认缩放1.0倍（就是不缩放）,并且禁止用户缩放。

还有一种常用的写法，就是根据自己设备的dpr来还原设备本来宽度，如假设手机devicePixelRatio为2，则设置
```html 
<meta name="viewport" content="initial-scale=0.5,
                               maximum-scale=0.5,
                               minimum-scale=0.5,
                               user-scalable=no">
```

## 开始适配
了解了这些，前端适配工作总算可以开始了。但马上出现了一道大山。我们让所有设备的viewport都等于device-width，带来了一个新的问题。不同设备的device-width并不相同。即使不考虑pad，只考虑各大主流手机，也从320-400多不等。我们希望不同尺寸设备能呈现几乎一致的内容来保证用户体验。

在布局上，我们可以通过诸如flex布局等方法来解决。但是当涉及元素尺寸大小的时候，如果还是用px做单位，那么将会呈现天差地别的差异。那该怎么办呢？

### CSS单位rem
传统页面尺寸我们都用px定义。而rem是相对于根元素<html>的font-size来做计算的。举个例子：根元素的font-size：16px; 那么1rem=16px，0.5rem = 8px。这样一来，我们只要根据不同屏幕尺寸设定不同的font-size。后面只要采用rem做尺寸单位，就能保持视觉一致。那该如何判断不同尺寸屏幕并设置相应font-size呢？

###　媒体查询
媒体查询允许我们来根据设备不同来设定不同的css样式，最常用的就是根据不同屏幕尺寸来设定不同样式。举个栗子：

```css
@media screen and (max-width: 320px) {
    html {
        font-size: 20px;
    }
}
```

意思就是当layout viewport小于320px时，根节点的font-size为20px;

### JS判断
在页面加载时判断当前屏幕尺寸，并根据尺寸设定相应的font-size到根节点。

这样一来，我们做前端适配的思路就清晰了。

## 适配步骤

1. **设置viewport为device-width。**

2. **通过媒体查询、或者js，判断不同屏幕尺寸，设定根节点font-size。**

3. **使用rem来设定元素尺寸单位。**

这样我们再去看手淘的[解决方案](https://github.com/amfe/article/issues/17)[5]，就能大致清晰它的原理了。
但对比上述的几点，可以发现flexible并不是采用viewport设置为device-width，而是采用还原成设备物理分辨率，这样的好处是为了解决高分屏下的1px问题。具体就不展开讨论了。


## 参考
1. [W3CSchool](http://www.w3school.com.cn)(http://www.w3school.com.cn)
2. [A pixel is not a pixel is not a pixel](http://www.quirksmode.org/blog/archives/2010/04/a_pixel_is_not.html)(http://www.quirksmode.org/blog/archives/2010/04/a_pixel_is_not.html)
3. [移动前端开发之viewport的深入理解](http://www.cnblogs.com/2050/p/3877280.html)(http://www.cnblogs.com/2050/p/3877280.html)
4. [移动前端第一弹：viewport详解](http://www.kancloud.cn/jaya1992/fe-notes/86799)(http://www.kancloud.cn/jaya1992/fe-notes/86799)
5. [使用Flexible实现手淘H5页面的终端适配](https://github.com/amfe/article/issues/17)(https://github.com/amfe/article/issues/17)
