# 【性能优化】Daraz Seller Center 首页实践

## 背景

&emsp;&emsp;首页作为用户登录进系统后看到的第一个页面，包含了整个店铺的数据大盘、平台活动、系统通知等态势信息，其门户的地位直接影响着用户对网站的整体预期
&emsp;&emsp;长久以来，因为历史技术债和人手问题，首页一直处于用户体验极差的泥潭，有业务领导曾直言我们的卖家平台现在还处于 win98 年代
&emsp;&emsp;随着 Daraz 技术力量的茁壮发展，卖家平台走过了野蛮生长的阶段，今年开始精细化运作，首页作为全站性能体验的第一站，改变由此开始

&emsp;&emsp;话不多说，先上优化前后效果对比：
&emsp;&emsp;旧版：
![旧版](https://intranetproxy.alipay.com/skylark/lark/0/2022/gif/347225/1672474934283-3544bc6c-8027-4280-b1fc-d1d05e4e3696.gif)
&emsp;&emsp;新版：
![新版](https://intranetproxy.alipay.com/skylark/lark/0/2022/gif/347225/1672474930637-8c8c54a2-a38d-4bde-90bb-9381e6c17d86.gif)

&emsp;&emsp;GIF 对比图为巴基斯坦同学在本地线上环境录制，因为优化前最早的页面已经无法打开，旧版 GIF 使用的是经过第一波优化后的 4200ms 版本，而非最早的 6500ms 版本

## 性能的定义

&emsp;&emsp;经过上半年在 ARMS、AEM、Itrace 等多个监控平台中进行对比，最后选中了 AEM 作为卖家的基准数据监控平台
&emsp;&emsp;AEM 中对于性能的监控有 onload、ondomready、首次渲染、首次内容渲染、最大内容渲染这 5 个指标，此次性能优化的目的是提升用户的感知加载速度，从而提升用户体验，因此选用了最大内容渲染（下文将采用缩写 LCP 来指代）作为性能提升的关键参考指标
&emsp;&emsp;由于南亚国家的网络环境并不是很好，网速常常出现大的波动，平均值会被各种极端条件拉高，因此选用中位数作为参考标准
&emsp;&emsp;Daraz 经营范围内的五个国家中，属巴基斯坦（PK）的客户数最多，加载速度最慢，因此使用巴基斯坦站点的数据作为主要参考

### 【第一波】减少文件体积与提升执行速度

**优化前数据：**

![image](https://intranetproxy.alipay.com/skylark/lark/0/2022/png/347225/1671958044271-ef74d091-b670-4719-9c56-8b7815078e95.png?x-oss-process=image%2Fformat%2Cwebp)

&emsp;&emsp;LCP：6500ms
&emsp;&emsp;（刚开始时还没有确定 LCP 作为关键参考指标，并且 AEM 平台当时没有上线中位数功能，因此指标数据为 onload 时间的平均值，根据优化到最后所发现的一系列问题，推测当时的 LCP 中位数约为 6500ms）

**问题分析：**

![image](https://intranetproxy.alipay.com/skylark/lark/0/2022/png/347225/1671957795415-fa9d4929-06d9-4b56-91a8-5b6400f71e1a.png?x-oss-process=image%2Fformat%2Cwebp%2Fresize%2Cw_1408%2Climit_0)

&emsp;&emsp;经过对页面加载链路和代码的分析，得出 3 点结论：
&emsp;&emsp;&emsp;&emsp;1、MTOP 请求嵌套过多，到达最终的页面渲染，需要至多 3 层接口的数据返回
&emsp;&emsp;&emsp;&emsp;2、代码项目采用 React15 和 lsc 脚手架，打包出的 JS 和 CSS 产物体积大，执行速度慢
&emsp;&emsp;&emsp;&emsp;3、代码逻辑繁杂，业务的实现有大量包套包、多层嵌套执行的现象

**解决方案：**

&emsp;&emsp;如此根深蒂固的问题，改良已不可能解决，直接祭出程序员大招：重写
&emsp;&emsp;此次优化是借力业务重构顺带着的项目，刚好首页在业务上也无法满足新的需求，需要重新设计，因此重写计划得以顺利进行
&emsp;&emsp;既然要重写了，并且从今年开始卖家平台进入精细化运作阶段，老旧的技术底座承载不了 boss 们对卖家平台的期望，因此把前端项目结构也一并重写掉了
&emsp;&emsp;最终采用 React18 和 Webpack 进行项目开发和构建，并联合后端采用主接口的 MTOP 请求方式来减少请求嵌套

**优化结果：**

![image](https://intranetproxy.alipay.com/skylark/lark/0/2022/png/347225/1671960154411-57f208ca-60f0-465d-9322-710dcc93e962.png?x-oss-process=image%2Fformat%2Cwebp%2Fresize%2Cw_1021%2Climit_0)
![image](https://intranetproxy.alipay.com/skylark/lark/0/2022/png/347225/1671960218886-37b13da0-7e42-4f10-ad23-83b98f516038.png?x-oss-process=image%2Fformat%2Cwebp)

&emsp;&emsp;LCP：4200ms
&emsp;&emsp;（2022.10.1 日的数据，此时 AEM 已经上线了 LCP 中位数）

### 【第二波】精简执行顺序

**优化前数据：**

![image](https://intranetproxy.alipay.com/skylark/lark/0/2022/png/347225/1671960991444-c9eb8f0a-806f-4557-ab3b-0acd7476f16e.png?x-oss-process=image%2Fformat%2Cwebp)

&emsp;&emsp;LCP：4200ms

**问题分析：**

![image](https://intranetproxy.alipay.com/skylark/lark/0/2022/png/347225/1671964818490-aee3daba-10d9-435d-b5e9-79d7a7c21249.png?x-oss-process=image%2Fformat%2Cwebp%2Fresize%2Cw_1363%2Climit_0)

&emsp;&emsp;有时候机会是在无意间产生的，还记得在第一波优化时我们顺带实现了一整套新的前端开发方案吗？就在实现这套方案时，为了接入 dada 平台，并且最小化的精简项目体积，我跑去询问 dada 项目组的同学有些 dada 平台自带的文件是否可以删除。他们告诉我可以删除的文件的同时，还吐槽了下老 dada 维护困难，他们最近开发的新 dada 平台就没有这些冗余文件的问题。我问了下新 dada 如何实现，和老 dada 有何差别，他们派了位同学热情的给我介绍了半个小时，并展示了一个 demo
&emsp;&emsp;我此刻才意思到了老代码问题的严重性绝不只是代码本身，连它依赖的平台都有着深深的技术债
&emsp;&emsp;我急忙去分析了请求的文件和顺序，在密密麻麻的文件请求之中，发现 HTML 到业务 JS、CSS 文件之间居然有 2 层逻辑：
&emsp;&emsp;第 1 层是 dada 平台的逻辑，他们会先去请求到 dada 的配置文件，然后再去执行业务的逻辑
&emsp;&emsp;第 2 层属于历史问题，有一个 Daraz 和 Lazada 共用的 layout 文件，这个 layout 文件会先判断访问的是 Daraz 还是 Lazada，然后决定页面布局的逻辑和请求站点的全局数据，当全局数据请求完毕后才会加载真正的业务文件

**解决方案：**

&emsp;&emsp;针对上述问题，做出了 2 点改动：
&emsp;&emsp;&emsp;&emsp;1、迁移旧 dada 平台到 dada 团队开发的新 dada 平台
&emsp;&emsp;&emsp;&emsp;2、重写 layout 文件
&emsp;&emsp;这样做的原因是，对于迁移到新 dada，同团队迁移成本比换一个平台或者自己维护后端模版 ROI 更高，且作为内测用户，可以享受到 dada 团队更多技术上的关怀
&emsp;&emsp;新 dada 采用自由配置 html 模版的办法，直接省去了过去先走 dada 逻辑的步骤，并且由于模版可以自由配置，解锁了未来更多的可能性
&emsp;&emsp;新 layout 只存在 daraz 的逻辑，并且和业务文件并行加载，缓存站点全局数据
![image](https://intranetproxy.alipay.com/skylark/lark/0/2022/png/347225/1671968020001-a6c04367-920b-4f85-aae4-ef892686b8da.png?x-oss-process=image%2Fformat%2Cwebp%2Fresize%2Cw_1127%2Climit_0)
&emsp;&emsp;改动之后，从之前的 7 段请求缩短到 3 段请求即可开始渲染页面

**优化结果：**
&emsp;&emsp;LCP：4200ms
&emsp;&emsp;这里 LCP 数据没变，并且没放图，是因为有一个小插曲
&emsp;&emsp;由于切换新 dada 平台和使用重写的 layout 是一个非常大的改动，且这次没有业务项目可以借力，为了稳妥起见，发了一个流量每天只有二十多人的 verifyBrandInformation 页面到线上去试试水
&emsp;&emsp;然后就踩坑了，集团的 mtop 文件，内部有些特殊逻辑，新 layout 请求左侧菜单的 MTOP 接口偏偏触发了这个逻辑，任何请求都莫名的在请求左侧菜单这个 MTOP 请求发出后才可以发出，强制排队，即使我发现了这个问题，手动异步或者延迟发送，后续请求仍然会紧跟着请求菜单的请求结束后才发出
&emsp;&emsp;这个问题直接导致了迁移至新 dada 页面的 LCP 时间和之前差不多，甚至还多了 100ms
&emsp;&emsp;在我急忙修复了这个 bug 后，跟着又出现了第二个坑
&emsp;&emsp;这个坑倒不是程序问题，而是统计问题，由于 verifyBrandInformation 页面流量很小，受网络波动的影响大，攒了几天的数据加起来不到 100 个，根据这仅有的数据我发现不仔细看几乎看不出优化的效果，尽管在国内的网络环境下，刷新时左侧的菜单栏已经不动了，就像单页面应用一样，但是当时 pk 的本地数据告诉我不能冒这个险把首页发到线上，因此首页在第二波优化后就一直躺在预发环境，没有真正发上线，直到第三波优化时才一起发上去的
&emsp;&emsp;随着时间的推移，样本的积累，现在再去看 verifyBrandInformation 页面才能观测到是有 500ms 的优化成果的，甚至还把秒开率从不到 1%提升到了 12%
![image](https://intranetproxy.alipay.com/skylark/lark/0/2022/png/347225/1671967216902-d2618161-28e6-4738-8c34-b54ebde1756c.png?x-oss-process=image%2Fformat%2Cwebp%2Fresize%2Cw_1500%2Climit_0)
![image](https://intranetproxy.alipay.com/skylark/lark/0/2022/png/347225/1671967231194-ef9f8932-6b74-4ebe-bfd0-9cd9e2f8478c.png?x-oss-process=image%2Fformat%2Cwebp%2Fresize%2Cw_1500%2Climit_0)

### 【第三波】缓存 HTML

**优化前数据：**

![image](https://intranetproxy.alipay.com/skylark/lark/0/2022/png/347225/1671960991444-c9eb8f0a-806f-4557-ab3b-0acd7476f16e.png?x-oss-process=image%2Fformat%2Cwebp)

&emsp;&emsp;LCP：4200ms

**问题分析：**

![image](https://intranetproxy.alipay.com/skylark/lark/0/2022/png/347225/1671977623941-d4b6754a-ea5b-4bfe-9aa6-103dc6b91dc0.png?x-oss-process=image%2Fformat%2Cwebp%2Fresize%2Cw_1021%2Climit_0)

&emsp;&emsp;经过第二波优化后，layout 文件和业务文件已经是并行加载了，因此可以抽象成一个文件来简化分析，加载结构又回到了最初的样子
![image](https://intranetproxy.alipay.com/skylark/lark/0/2022/png/347225/1671977910688-4191c18f-4c0e-4422-9643-facc70e54edb.png?x-oss-process=image%2Fformat%2Cwebp)
&emsp;&emsp;但是显然不对，同样的页面国内预发环境 LCP 已经到了 1200ms，pk 却仍然高达 4200ms
&emsp;&emsp;看来问题光从自己的电脑上已经无法看出来了，必须到当地的环境去看，虽然无法直接在真实用户的电脑上调试分析，但是我们找到了身处当地的产品同学，获得许可后，通过远程控制他的电脑在浏览器上进行调试
![image](https://intranetproxy.alipay.com/skylark/lark/0/2022/png/347225/1671978300545-cf0e10f3-a292-4051-8b9b-f418e0985132.png?x-oss-process=image%2Fformat%2Cwebp%2Fresize%2Cw_1500%2Climit_0)
&emsp;&emsp;通过对每个文件的请求进行观测，发现 pk 当地的任何 html、js、css、mtop 请求都有一个 500ms 左右的 Waiting 时间，这个时间代表的是从请求发出，到收到响应第一个字节所花费的时间，这个时间一般受到服务器距离、线路等问题的影响
&emsp;&emsp;一开始以为是新 dada 平台和中台网关没调试好，走的线路有问题，但是找了相关同学经过几轮调查后发现他们那里并没有问题
&emsp;&emsp;顿时脑海中产生了一个可怕的想法：它原来就是这样的
&emsp;&emsp;和当地同学再次远程连线，查看了几个旧 dada 平台的页面，坐实了这个可怕的想法，但我仍然不敢相信，Daraz 拥有上百万的用户，居然没有在本地部署 CDN 服务器，每个请求都顶着这么高的延迟跑在这个应用上
&emsp;&emsp;我又去看了买家端最为重要的 PDP 页面，发现静态资源 CDN 的域名和卖家是一样的，也都是 g.alicdn.com，我又换了几个 lzd、aeis、assets、aeu 等开头的 cdn 地址，发现一个比一个慢，g.alicdn.com 居然还是中间最快的
&emsp;&emsp;我顿时心生恻隐，广大的 Daraz 用户居然就在这种水深火热的情况下坚持使用我们的应用
&emsp;&emsp;要知道虽然一个请求天生自带 500ms 的延迟看起来不多，但是这个延迟时间是可以累加的，一个最精简的经典顺序，HTML -> JS -> MTOP 请求，一套下来不算下载时间、执行时间、渲染时间，光请求延迟就 1500ms，更别说我们的网站还大量充斥着第一波第二波优化中解决的请求套请求，文件套文件的逻辑了，每套一层就多 500ms，难怪一开始 LCP 能高达 6500ms 这么离谱了
&emsp;&emsp;虽说这把高端局，但我感觉已经发现了破局的关键所在，就在于解决这巨高的请求延迟

**解决方案：**

![image](https://intranetproxy.alipay.com/skylark/lark/0/2022/png/347225/1671981103309-8a14b9fa-c261-43e8-ba9a-69f6691aab9f.png?x-oss-process=image%2Fformat%2Cwebp)

&emsp;&emsp;总所周知，解决请求慢的最佳方法就是不请求，我不请求就不存在请求慢的问题
&emsp;&emsp;JS、CSS 浏览器自带缓存，MTOP 请求不能缓存，那么就得在 HTML 上做文章了，在分析了 HTML 没有任何动态的数据后，决定使用 service worker 对 HTML 文件进行缓存

**优化结果：**

![image](https://intranetproxy.alipay.com/skylark/lark/0/2022/png/347225/1671981591642-71d0bea7-e1ce-414f-aff0-f20b9beebaf8.png?x-oss-process=image%2Fformat%2Cwebp)

&emsp;&emsp;LCP：2200ms

&emsp;&emsp;第三波带着第二波的优化一起发布的，这次的效果极为明显，由于用了新的 url 地址，检测起来很方便，仅仅半个小时就出了结果，LCP 时间降到了 2200ms，降幅高达 52%，秒开率从不到 1%提升到了 14%
&emsp;&emsp;事后来看，如果排除第二波的优化效果，第三波的优化效果约在 1500ms，远超了预期。这点现在还没想明白，理论上即使 HTML 是经过后端逻辑的，请求速度比纯静态资源要慢，最多也就降低 1000ms，不会到 1500ms 这么多，根据当前数据只能猜测是不是第二波的优化被低估了，其实不止 500ms，毕竟他节省了 2 个请求。又或者有不少数量用户的网络情况要比我们那位本地同事的网络更差，所以他们其实 HTML 请求延迟更高，对于第三波优化的收益也更大

### 【第四波】请求前置

**优化前数据：**

![image](https://intranetproxy.alipay.com/skylark/lark/0/2022/png/347225/1671981591642-71d0bea7-e1ce-414f-aff0-f20b9beebaf8.png?x-oss-process=image%2Fformat%2Cwebp)

&emsp;&emsp;LCP：2200ms

**问题分析：**

![image](https://intranetproxy.alipay.com/skylark/lark/0/2022/png/347225/1671983399236-af1e27b1-49ed-4b0f-95ba-6f6a6584ac75.png?x-oss-process=image%2Fformat%2Cwebp)

&emsp;&emsp;通过反复观看之前给当地同学调试的录像，发现在 HTML -> JS -> MTOP 之间有一段空隙，这个空隙是文件的执行的时间，比方说 HTML 文件执行到加载 JS 这一行代码时，才会发出请求去获取 JS 文件，JS 文件执行到发出 MTOP 这一行代码时，才会发出请求获取 MTOP 信息
&emsp;&emsp;MTOP 虽说一般是 JS 文件发出的，但并不是非得 JS 文件才能发出，倘若使用 HTML 将 MTOP 早一步发出，页面的渲染内容也就可以早一步获取了

**解决方案：**

![image](https://intranetproxy.alipay.com/skylark/lark/0/2022/png/347225/1671987953805-7496b7b7-265f-42d3-a098-5a5b33baca31.png?x-oss-process=image%2Fformat%2Cwebp)

&emsp;&emsp;我将 MTOP 前置到 HTML 模版内进行请求，和 JS、CSS 等文件并行请求，这样就可以提前 JS 文件一个身位，节省出 JS 文件读取缓存及执行的时间，总行程从之前的 3 段请求变为 2 段请求

**优化结果：**

![image](https://intranetproxy.alipay.com/skylark/lark/0/2022/png/347225/1671983991621-bfb6dc4a-3be3-4da2-b4d7-de6c3900c91b.png?x-oss-process=image%2Fformat%2Cwebp)

&emsp;&emsp;LCP：1900ms
&emsp;&emsp;由于首页的流量比较大，所以即使优化幅度只有 300ms 也能够看出明显效果，秒开率也从 14%提升到了 21%

### 【第五波】常驻数据缓存

**优化前数据：**

![image](https://intranetproxy.alipay.com/skylark/lark/0/2022/png/347225/1671983991621-bfb6dc4a-3be3-4da2-b4d7-de6c3900c91b.png?x-oss-process=image%2Fformat%2Cwebp)

&emsp;&emsp;LCP：1900ms

**问题分析：**

![image](https://intranetproxy.alipay.com/skylark/lark/0/2022/png/347225/1671984224573-7d9e18df-2bd0-42a7-947f-d77f1525aded.png?x-oss-process=image%2Fformat%2Cwebp%2Fresize%2Cw_1500%2Climit_0)

&emsp;&emsp;根据谷歌制定的 LCP 性能标准，低于 2.5s 的就算是 GOOD，经过第三波优化，首页就已经来到了 GOOD 的段位了，在国内预发环境最佳数据甚至已经下潜到了 0.3s，但还有机会
![image](https://intranetproxy.alipay.com/skylark/lark/0/2022/png/347225/1671987855977-b4d87fc5-2e50-4c13-b555-274d8efad537.png?x-oss-process=image%2Fformat%2Cwebp)
&emsp;&emsp;第四波优化提到当前的瓶颈在于 MTOP 请求，由于你不可能把 MTOP 请求也缓存掉，这样用户压根请求不到真实数据。首页作为店铺数据大盘之所在，需要时刻有最新的数据展示，也不能采用先展示缓存数据再请求真实数据的 T+1 方案，因为一旦网络波动，正确数据姗姗来迟，用户将会在正确数据到来前产生疑惑，降低用户体验
&emsp;&emsp;重新审视提升 LCP 的初衷，我们的目的是想缩短用户对于网站已经加载完毕的感知时间，用户知道他的网络会有波动，他接受这个问题，但同时他也想知道到底是谁出了问题，是网站本身慢，还是网站发出的请求慢，虽然在技术人员看来这是一回事，但是用户会认为这是两回事
![image](https://intranetproxy.alipay.com/skylark/lark/0/2022/png/347225/1671987079144-114f018e-1762-4be3-a4c8-ed132f62551e.png?x-oss-process=image%2Fformat%2Cwebp%2Fresize%2Cw_1500%2Climit_0)
&emsp;&emsp;举个例子，这是 AEM 查询数据时的画面，页面很早就加载好了，但是加载具体数据的时候要等上一段时间，试问一下，你心里除了怪 AEM 慢，会不会同时还有我要查询的数据种类很多，我的网站数据量很大，所以相比页面加载，拉取数据的时间会长一些的想法
&emsp;&emsp;用户很可能也是这么想的，首页的结构很早就加载出来了，但是首页所需要的数据种类很多，要拉取很多的大盘数据，相比页面加载确实会花费更多的时间
&emsp;&emsp;一旦问题从首页本身渲染的慢转变为拉取数据慢，涉及到用户自身个体差异，并且用户开始理解慢的原因时，用户就会变得宽容起来
&emsp;&emsp;因此我们只要将网站加载和请求数据在用户心中分离开，并且减少网站加载部分的时间，就可以让用户不损失用户体验（感到疑惑）的同时提升用户体验（LCP 时间缩短）

**解决方案：**

![image](https://intranetproxy.alipay.com/skylark/lark/0/2022/png/347225/1672028525925-4af8034e-70c2-489a-97ee-005ab8a77080.png?x-oss-process=image%2Fformat%2Cwebp%2Fresize%2Cw_1500%2Climit_0)

&emsp;&emsp;首页的页面是由多个模块组合而成，虽说不同状态、权限下的用户看到的首页是不一样的，但是具体到某个用户身上，他的状态、权限切换速度并没有大盘数据那么快，很长时间才会度过一个阶段，因此我们可以将 MTOP 请求中关乎页面结构和样式、不常变动的数据（例如 banner、daraz 大学课程等）进行缓存，常常变动的业务数据使用 '-' 或者 空 来占位
&emsp;&emsp;如此便可以将用户心中网站的加载和数据的请求分离开来，对于网站的加载部分来说，HTML、JS、MTOP 这三块都可以放心使用缓存

**优化结果：**

![image](https://intranetproxy.alipay.com/skylark/lark/0/2022/png/347225/1672476731806-aceb140d-0036-4694-a7ea-dac01152e1d5.png?x-oss-process=image%2Fformat%2Cwebp)

&emsp;&emsp;LCP：1600ms
&emsp;&emsp;优化了大约 300ms，秒开率从 21%提升到了 25%

### 【最终优化结果】

LCP：6500ms -> 1600ms 降幅为 406%
秒开率：接近 0% -> 25% 增幅为 25%

## 总结

**要尽可能看到真实用户的表现**
![image](https://intranetproxy.alipay.com/skylark/lark/0/2022/png/347225/1672476486690-5323a78a-9f6d-4a9d-ac0a-7b03eb075f6c.png?x-oss-process=image%2Fformat%2Cwebp)
![image](https://intranetproxy.alipay.com/skylark/lark/0/2022/png/347225/1672476329815-972461ae-9612-455b-95fe-520f69269fc8.png?x-oss-process=image%2Fformat%2Cwebp)
![image](https://intranetproxy.alipay.com/skylark/lark/0/2023/png/347225/1672544957603-8fe0393c-9587-4be4-999f-8c8364015277.png?x-oss-process=image%2Fformat%2Cwebp)

&emsp;&emsp;前两波优化都是在自己和周围同学的电脑上试的，我们本地的效果相当好，但实际到了线上却成效不大
&emsp;&emsp;直到第三波优化，使用了本地同学的电脑调试后，才发现了问题的关键，第三波优化的成效也是最明显的
&emsp;&emsp;这里由于没办法直接接触到真实用户的电脑，本地同学的网络环境在当地也算是高级写字楼，和真实的用户环境仍然有很大的差别，所以第三波优化取得超出预期的效果，却仍然不知道具体原因，十分遗憾

**海外环境与国内环境的差别**

&emsp;&emsp;中国网民规模为 10.51 亿，互联网普及率达 74.4%
&emsp;&emsp;美国网民规模为 3.11 亿，互联网普及率达 93.9%
&emsp;&emsp;巴基斯坦网民规模为 0.82 亿，互联网普及率达 36.5%

&emsp;&emsp;大部分同学从学习到工作基本上都是待在国内，服务国内的用户。淘宝网马上都创办 20 年了，国内无论是网络基建还是上网习惯都已接近发达国家水平，但是对于南亚五国来说，他们的冲浪之路才开始不久
&emsp;&emsp;因此我们用国内的方法论直接套在国外并不合适，国内一提到优化大家就想到减小体积、减少请求、提高运行效率啥的，这些确实是优化的重点，但如果不对症下药事无巨细去做这些 ROI 是很低的
![image](https://intranetproxy.alipay.com/skylark/lark/0/2022/png/347225/1672475929036-1a3982bb-4a6c-49a7-8221-285c525fa4c6.png?x-oss-process=image%2Fformat%2Cwebp%2Fresize%2Cw_1500%2Climit_0)
![image](https://intranetproxy.alipay.com/skylark/lark/0/2022/png/347225/1672476112151-7356ca09-0bee-47b1-aa81-874682d64857.png?x-oss-process=image%2Fformat%2Cwebp%2Fresize%2Cw_1500%2Climit_0)
&emsp;&emsp;同一套优化方案在不同国家或者同一国家不同时段都有着明显的分布规律，弄清楚其中的原因才能够因地制宜，最大效率的指导优化方案的设计

**进一步细化有助于问题的解决**

&emsp;&emsp;第五波优化中，为了解决 MTOP 的瓶颈问题，打算缓存 MTOP 接口，但是缓存了 MTOP 接口就会造成数据更新不及时的问题，不缓存 MTOP 接口又无法解决瓶颈。当时是回想了初衷，坚定了信念之后才决定坐下来进一步思考，既要提升性能带来的用户体验又不能损失别处的用户体验，然后才发现了可以缓存一部分 MTOP 接口的做法
&emsp;&emsp;这种细化问题，把非黑即白的问题进一步拆分，从而打开思路的方法我觉得可以作为通用的解决问题的方法（之前饭后和同事讨论第三次世界大战全球战力划分问题时，我说中俄要面临欧盟的压力，没法全力对付美国，同事说欧盟也不是铁板一块，每个成员国有自己的利益诉求，可以加以利用。现在看来也有异曲同工之妙）
