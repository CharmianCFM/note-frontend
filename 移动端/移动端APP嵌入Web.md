### 1 webview
[**Webview **](https://juejin.cn/post/6950890297450561550#heading-0)**是一个基于webkit引擎，可以解析DOM 元素、展示html页面的控件，它和浏览器展示页面的原理是相同的，所以可以把它当做浏览器看待。**
> （chrome浏览器也是基于webkit引擎开发的，Mozilla浏览器是基于Gecko引擎开发的）
> Android的Webview在低版本和高版本采用了不同的webkit版本内核，4.4后直接使用了Chrome。
> 个人理解，电脑上展示html页面，通过浏览器打开页面即可浏览，而手机系统层面，如果没有webview支持，是无法展示html页面，所以webview的作用即用于手机系统来展示html界面的，所以它主要在手机系统上加载html文件时被需要。

webview是原生系统用于移动端 APP 嵌入(Embed) Web 技术，方式是内置了一款高性能webkit内核浏览器。
> 一般会在SDK中封装为一个叫做WebView组件。分类：
> 安卓（Android）：SDK 中有WebView控件；
> 苹果（IOS，MacOS）：WebView/UIWebView/WKWebView(UIView/NSView)；。

WebView控件功能强大，除了具有一般View的属性和设置外，还可以对url请求、页面加载、渲染、页面交互进行强大的处理。
**如何与App native的交互？**
1.JSBridge

- 在IOS中，主要使用WebViewJavascriptBridge来注册
- 在Android中，需要通过addJavascriptInterface来注册

2.schema
**微信小程序中的webview**
小程序的主要开发语言是 JavaScript ，小程序中，逻辑层和渲染层是分开的，分别运行在不同的线程中。 具体的运行环境如下：

| 运行环境 | 逻辑层 | 渲染层 |
| --- | --- | --- |
| iOS | JavaScriptCore | WKWebView |
| 安卓 | V8 | chromium定制内核 |
| 小程序开发者工具 | NWJS | Chrome WebView |

可以看出，小程序的渲染层也是运行在webview上的；
**为什么webview会很慢？**
普通用户访问webview经历过程如下：

1. 交互无反馈
2. 到达新的页面，页面白屏
3. 页面基本框架出现，但是没有数据；页面处于loading状态
4. 出现所需的数据

![image.png](https://cdn.nlark.com/yuque/0/2024/png/40468162/1710760230957-553dfff8-3905-4940-9044-1a2115757252.png#averageHue=%23fbfbfb&clientId=u16d82795-cff8-4&from=paste&height=183&id=ufba6529e&originHeight=497&originWidth=1512&originalType=binary&ratio=2&rotation=0&showTitle=false&size=109159&status=done&style=none&taskId=udce5b295-8b2e-4f9e-b493-42572e4f055&title=&width=557)
在浏览器中，我们输入地址时（甚至在之前），浏览器就可以开始加载页面。 而在客户端中，客户端需要先花费时间初始化WebView完成后，才开始加载。
**webview的性能优化**

- WebView初始化慢，可以在初始化同时先请求数据，让后端和网络不要闲着。
- 后端处理慢，可以让服务器分trunk输出，在后端计算的同时前端也加载网络静态资源。
- 脚本执行慢，就让脚本在最后运行，不阻塞页面解析。
- 同时，合理的预加载、预缓存可以让加载速度的瓶颈更小。
- WebView初始化慢，就随时初始化好一个WebView待用。
- DNS和链接慢，想办法复用客户端使用的域名和链接。
- 脚本执行慢，可以把框架代码拆分出来，在请求页面之前就执行好。

**webview 和原生Native如何选择？**
> 在内嵌的WebView中应该限制允许打开的WebView的域名，并设置运行访问的白名单。或者当用户打开外部链接前给用户强烈而明显的提示。

在一个客户端内，native目前主要功能是提供高效而基础的功能；内部的WebView则添加一些性能体验要求不高但动态化要求高的能力。
提高客户端的动态能力，或者提高WebView的性能，都是提升App功能覆盖的方式。
而目前的各种框架，ReactNative、Week包括微信小程序，都是这个趋势的尝试。
![image.png](https://cdn.nlark.com/yuque/0/2024/png/40468162/1710760817591-8cd8a1c4-8cb2-46fd-97f2-44bc7147cae8.png#averageHue=%23f3eee1&clientId=u16d82795-cff8-4&from=paste&height=322&id=u3e2d98a0&originHeight=1153&originWidth=1512&originalType=binary&ratio=2&rotation=0&showTitle=false&size=333160&status=done&style=none&taskId=u03f0cca1-a88d-46bc-8176-cd8448fc3a0&title=&width=422)

### 2 离线包
统的 H5 技术容易受到网络环境影响，因而降低 H5 页面的性能。通过使用离线包，可以解决该问题，同时保留 H5 的优点。
**离线包** 是将包括 HTML、JavaScript、CSS 等页面内静态资源打包到一个压缩包内。预先下载该离线包到本地，然后通过客户端打开，直接从本地加载离线包，从而最大程度地摆脱网络环境对 H5 页面的影响。
使用 H5 离线包可以给您带来以下优势：

- **提升用户体验**：通过离线包的方式把页面内静态资源嵌入到应用中并发布，当用户第一次开启应用的时候，就无需依赖网络环境下载该资源，而是马上开始使用该应用。
- **实现动态更新**：在推出新版本或是紧急发布的时候，您可以把修改的资源放入离线包，通过更新配置让应用自动下载更新。因此，您无需通过应用商店审核，就能让用户及早接收更新。

**离线包结构**
离线包是一个 .amr 格式的压缩文件，将后缀 amr 改成 zip 解压缩后，可以看到其中包含了 HTML 资源和 JavaScript 代码等。待 H5 容器加载后，这些资源和代码能在 WebView 内渲染。
以 iOS 系统为例，下图显示了一般资源包的目录结构：

- 一级目录：一般资源包的 ID，如 20150901。
- 二级目录及子目录即为业务自定义的资源文件。建议所有的前端文件最好保存在一个统一的目录下，目录名可自定义，如 /www，并设定当前离线包默认打开的主入口文件，如 /www/index.html。

![image.png](https://cdn.nlark.com/yuque/0/2024/png/40468162/1710762240226-6d0baf26-3240-440b-8901-ea308265c4b4.png#averageHue=%23fcfcfb&clientId=u16d82795-cff8-4&from=paste&height=237&id=ud74f017d&originHeight=474&originWidth=1236&originalType=binary&ratio=2&rotation=0&showTitle=false&size=44715&status=done&style=none&taskId=u4390adb4-3379-4232-83c9-019c2361d1d&title=&width=618)
**离线包类型**
通常，在 H5 的开发过程中，会存在使用一些基础通用库的情况，比如 zepto，fastclick 等。在 App 中的 WebView，有时候缓存不可靠，曾经发现有机型在退出后，缓存自动失效。为了进一步提升 H5 页面性能，使用全局离线包，将一系列的通用资源打成一个特殊的 App 包，下发到客户端。
离线包可以分为以下类型：

- **全局离线包**：包含公共的资源，可供多个应用共同使用。
- **私有离线包**：只可以被某个应用单独使用。

使用全局离线包后，在访问 H5 的时候，都会尝试在这个包尝试读取。如果该离线包里有对应资源的时候，直接从该离线包里取，而不通过网络。因此，全局离线包的机制主要是为了解决对于通用库的使用。
由于要保证离线包的客户端覆盖率以及足够的通用性，此包一般的更新周期至少为 1 个月，并且严格控制离线包的大小。
**渲染过程**
当 H5 容器发出资源请求时，其访问本地资源或线上资源所使用的 URL 是一致的。
H5 容器会先截获该请求，截获请求后，发生如下情况：

- 如果本地有资源可以满足该请求的话，H5 容器会使用本地资源。
- 如果没有可以满足请求的本地资源，H5 容器会使用线上资源。

因此，无论资源是在本地或者是线上，WebView 都是无感知的。
离线包的下载取决于创建离线包时的配置：

- 如果 **下载时机** 配置为 **仅 WiFi**，则只有在 WiFi 网络时会在后台自动下载离线包。
- 如果 **下载时机** 配置为 **所有网络都下载**，则在非 WiFi 网络时会消耗用户流量自动下载，慎用。
- 如果当前用户点击 APP 时，离线包尚未下载完毕，则会跳转至 fallback 地址，显示在线页面。**fallback** 技术用于应对离线包未下载完毕的场景。每个离线包发布时，都会同步在 CDN 发布一个对应的线上版本，目录结构和离线包结构一致。fallback 地址会随离线包信息下发到本地。在离线包未下载完毕的场景下，客户端会拦截页面请求，转向对应的 CDN 地址，实现在线页面和离线页面随时切换。

![image.png](https://cdn.nlark.com/yuque/0/2024/png/40468162/1710762385889-bafd6f8f-9602-4f5c-a1d6-6e12e7ce2943.png#averageHue=%23f9efd3&clientId=u16d82795-cff8-4&from=paste&height=352&id=u53606360&originHeight=2168&originWidth=2025&originalType=binary&ratio=2&rotation=0&showTitle=false&size=472827&status=done&style=none&taskId=u1cca33af-580c-4854-827f-eeff140d1db&title=&width=329)
**离线包运行模式**
要打开离线包，需要完成以下步骤：

1. 请求包信息：从服务端请求离线包信息存储到本地数据库的过程。离线包信息包括离线包的下载地址、离线包版本号等。
2. 下载离线包：把离线包从服务端下载到手机。
3. 安装离线包：下载目录，拷贝到手机安装目录。
### 3 Hybrid App
[https://juejin.cn/post/7062967241268019214](https://juejin.cn/post/7062967241268019214)
**APP的种类**
App目前主要分为三类，分为Web App、Hybrid App、 Native App。
**Web App**
Web App即移动端的网站，将页面部署在服务器上，然后用户使用各大浏览器访问，不是独立APP，无法安装和发布。[手机淘宝](https://link.juejin.cn?target=https%3A%2F%2Fmain.m.taobao.com%2F)就是一个最常见的Web App。
优点：

1. 开发成本低，可以跨平台，调试方便。
2. 维护成本低 更新无需通知用户，不需要手动升级 无需安装App，不会占用手机内存。

缺点：

1. 无法获取系统级别的通知，提醒，动效等等。
2. 用户留存率低 设计受限制诸多 体验较差。

**Native App**
Native App就是我们常说的原生App啦，分为Android开发和IOS开发。Android基于Java语言，底层调用Goolge提供的API，IOS基于Objective c或Swift，底层调用Apple官方提供的Api。
优点：

1. 直接依托于操作系统,交互性最强,性能最好。
2. 功能最为强大,特别是在与系统交互中,几乎所有功能都能实现。

缺点

1. 开发成本高，无法跨平台，不同平台Android和iOS上都要各自独立开发。
2. 门槛较高，原生人员有一定的入门门槛，相比广大的前端人员而言较少。更新缓慢，特别是发布应用商店后，需要等到审核周期。维护成本高。

**Hybrid App**
接下来就是今天的主角啦，Hybrid App(混合应用程序)，主要原理就是将 APP 的一部分需要动态变动的内容通过 H5 来实现，通过原生的网页加载控件 WebView (Android)或 WKWebView（iOS）来加载H5页面（以后若无特殊说明，我们用 WebView 来统一指代 android 和 iOS 中的网页加载控件）。这样以来，H5 部分是可以随时改变而不用发版，动态化需求能满足；同时，由于 H5 代码只需要一次开发，就能同时在 Android 和 iOS 两个平台运行，这也可以减小开发成本，也就是说，H5 部分功能越多，开发成本就越小。我们称这种  **h5+原生 的开发模式为混合开发**，采用混合模式开发的 APP 我们称之为混合应用或 Hybrid APP。
你可以简单的理解为是Web App和Native App的杂合体。
优点：

1. 开发成本较低，可以跨平台，调试方便 维护成本低，功能可复用。
2. 功能更加完善，性能和体验要比起web app好太多，更新较为自由。

缺点：

1. 相比原生，性能仍然有较大损耗，不适用于交互性较强的app。

**总结**
![](https://cdn.nlark.com/yuque/0/2024/webp/40468162/1710767684212-39cb0d1f-e0cd-4660-a085-3981745ff453.webp#averageHue=%23f0f3ef&clientId=u16d82795-cff8-4&from=paste&id=u5fb87370&originHeight=344&originWidth=539&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=uc2ab78b3-4b67-4c41-b47c-832f6b7b772&title=)
说到这小伙伴们对App种类以及Hybrid App是不是有一定的了解了呢。好奇的宝宝又要问了，难道我们嵌入到App里面的H5页面不需要动态通信的吗？对，是需要的。接下来我们介绍Hybrid App和我们的H5页面的通信方式。
**App和H5通信**
通信分为H5端调用原生App端方法和原生App端调用H5端方法。App又分为IOS和Android两端，他们之间是有区别的，下面我们分开来介绍。
**App端调用H5端方法**
App端调用H5端的方法，调用的必须是绑定到window对象上面的方法。
**H5端调用App端方法**
提供给H5端调用的方法，Android 与 IOS 分别拥有对应的挂载方式。分别对应是:苹果 UIWebview JavaScriptCore 注入（这里不介绍）、安卓 addJavascriptInterface 注入、苹果 WKWebView scriptMessageHandler 注入。

