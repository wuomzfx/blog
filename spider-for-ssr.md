前端发展到现在，SPA应该已经被应用的非常广了。可惜的是，我们前进的是快，而人家搜索引擎爬虫跟用户的浏览器设备还跟不上脚步。辛辛苦苦写好的单页应用，结果到了SEO跟浏览器兼容这一步懵逼了。

很多同学肯定都想过服务端渲染的问题。然而一看vue、react关于服务端渲染的文档，可能就被唬住了。之前写好的并不能无缝迁移。而且，每当有个项目，就需要去run一套node服务。当然，架构能力好些的朋友，可以做好集中化管理。

所以，当我想在项目中，采用vue或者react的时候，就遇到这些非常大的阻力。正当我头疼脑热的时候呢，我发现了一条新途径。

在前不久呢，同事在群里分享了[puppeteer](https://github.com/GoogleChrome/puppeteer)，它GitHub的介绍如下：

> Puppeteer is a Node library which provides a high-level API to control headless Chrome over the DevTools Protocol. It can also be configured to use full (non-headless) Chrome.

大意就是说，一个提供操作Headless Chrome的API的node库。

再具体的说，就是能在node环境中，通过一些API，来“模拟”真实chrome访问页面，并对其进行模拟用户操作、获取DOM等。

那既然它能够像真实Chrome那样去访问页面并且输出渲染后的html，我为什么不能通过它来给我们做服务端渲染呢？

设想一下，我们有这样一个服务A，它能够像chrome一样访问指定页面，并把最终页面上的dom返回给你。

而你原本的业务服务器B，只需要判断是爬虫，或者低版本IE来访问时，调取该服务，得到html，将html返回给用户，这就实现了服务端渲染。大致流程图如下：

![](https://user-gold-cdn.xitu.io/2017/9/30/241c30c3fb75ba37c6178cd447de2ce3)

有这样一个思路后，我们就想办法来实践它。实践的过程，就是解决问题的过程。仔细想想，我们会遇到如下几个问题：

> Q1: 即使是模拟Chrome去请求页面，很多时候视图也是异步渲染的。比如先请求列表接口，得到数据再渲染出列表DOM。这个时间，我们并没有办法把控。那这个服务，到底时候才应该把加载完成的HTML返回呢？

遇到问题时，首先可以看看人家的文档 [Puppeteer API](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md)。欣喜的是，我们找到了如下几个方法：
```javascript
page.waitFor(selectorOrFunctionOrTimeout[, options[, ...args]])
page.waitForFunction(pageFunction[, options[, ...args]])
page.waitForNavigation(options)
page.waitForSelector(selector[, options])
```
我们可以通过一些设定，让页面在某种情况下才返回。比如我们通过设定 `page.waitForSelector('#app')`, 让页面出现 `id="app"` 的元素时，才把html内容返回。

或者通过设定 `page.waitForFunction('window.innerWidth < 100')`,当页面宽度小于100px时，才将此时的html内容返回。

通过这些方法，我们就能有办法控制，想要输给爬虫的，是什么时候、什么样的页面。

> Q2: 如果IE用户访问量比较大怎么办。我们虽然通过这样的系统，让本渲染不出页面的部分浏览器（IE9以下）能够渲染出页面了。但这样的请求过程相对而言会更耗时，这不是很合理。

那我们只要做一个缓存系统便好。每次请求，都会去判断此请求是否存在未过期的缓存HTML，如果存在，则直接返回缓存HTML，否则再去请求页面，保存缓存。

> Q3: 虽然页面是出来了，IE用户还是没办法做一些JS的交互。

这个我们没办法在服务层上去解决了，但我们可以在前端上做更友好的交互提示。如果判断用户是低版本IE，则出现一个小Tip，提示用户下载更好的浏览器，获取更好的体验。

> Q4: 单页应用的路由多是用锚点（哈希模式）来做的，而哈希参数，服务端无法获取，那就没办法请求正确的页面了。

这个有办法解决，可以采用HTML History模式的路由，如[vue-router](https://router.vuejs.org/zh-cn/essentials/history-mode.html)，然后路由链接最好以生成a标签+href的模式写在页面中，而不是`onclick`后js跳转，这样爬虫能最好的爬取整站页面。

当问题都想到办法解决后，我们就能开始真正coding了。

啪啪啪，啪啪啪 => [SSR-SERVICE](https://github.com/DXY-F2E/ssr-service)

好，然后就好了，不到200行的代码，我们就实现了一个 通用化的、服务化的、单页应用服务端渲染解决方案。


--[阅读原文](https://github.com/wuomzfx/blog/blob/master/spider-for-ssr.md) @[相学长](https://www.zhihu.com/people/xiang-xue-zhang)

--转载请先经过本人授权。
