# 商品详情页模版编辑器

### 事情的起因

![image](https://img.alicdn.com/imgextra/i4/O1CN012YDCIG1UjmW3LfE5f_!!6000000002554-0-tps-1538-1706.jpg)

&emsp;&emsp;一次在逛 Daraz App 时，连续点开了好几个推荐商品，发现他们都仅有几张主图，没有对商品的详细描述，例如商品的材质，使用了什么新技术，比其他商品优越在哪里等等

&emsp;&emsp;平日里看惯了国内花里胡哨的商品详情，突然就好奇为什么巴基斯坦的卖家会这样卖东西，难道他们的买家光看看主图就会下单？

&emsp;&emsp;询问过业务团队后得知，原来买家们对这种商品也不买账，同类产品，只有主图没有详情介绍的商品，销量都很惨淡

&emsp;&emsp;有一次前端周会，又从 CTO 那里了解到了一些情况，原来 Daraz 大部分的 GMV 都是靠少量的大卖家撑起来的，人数众多的的中小卖家，反而只贡献了小部分的 GMV，除了货源、渠道等硬性原因外，很多中小卖家缺乏商品详情页的装修经验也是造成这种情况的重要原因之一

&emsp;&emsp;因此，如果能够降低中小卖家的装修成本，提升商品详情页的质量，那么商品的销量肯定会有所提升

&emsp;&emsp;思来想去，最后我觉得可以采取模版这一办法，直接将类似产品的商品详情模版放上去，商家将模版里的图片、文本换成自己产品的就可以了，同时模版也可以启发商家，让商家想出比以往更多的介绍方式

### 编辑器实现方案

&emsp;&emsp;横向对比了国内外近十款电商商家工作台的商品详情页编辑器，结合 Daraz 的情况，最终的成品如下：

&emsp;&emsp;在商品编辑页拥有一个所见即所得的详情页展示区域，以及编辑器入口

![image](https://img.alicdn.com/imgextra/i4/O1CN01VyDbSO1LhBjIeNkoA_!!6000000001330-0-tps-3308-2158.jpg)

&emsp;&emsp;编辑器中可以选择各类商品模版

![image](https://img.alicdn.com/imgextra/i2/O1CN01sWijx81mo3IGqByo1_!!6000000005000-0-tps-3308-2158.jpg)

&emsp;&emsp;可以改动模版对应的图片及文本

![image](https://img.alicdn.com/imgextra/i3/O1CN01oK0QpE26Q2qbvyExr_!!6000000007655-0-tps-3308-2158.jpg)

&emsp;&emsp;可以自由添加单个组件，定制个性化详情页

![image](https://img.alicdn.com/imgextra/i2/O1CN01nAQxfD1KKur9xFk5b_!!6000000001146-0-tps-3308-2158.jpg)

&emsp;&emsp;可以自由添加纯文本或者纯图片

![image](https://img.alicdn.com/imgextra/i1/O1CN01oIsFxW27ezNzKu5AL_!!6000000007823-0-tps-3308-2158.jpg)

### 商品详情页图片数量占比 A/B 测试数据

![image](https://img.alicdn.com/imgextra/i4/O1CN01RLVDWf1dCC12Z8tit_!!6000000003699-0-tps-3308-2158.jpg)

&emsp;&emsp;经过线上 1%流量的验证，使用新的模版编辑器的商品详情页，拥有至少 2 张介绍图片的商品比例为 42.53%，高于使用普通编辑器的 17.69%；拥有至少 8 张介绍图片的商品比例为 8.40%，高于使用普通编辑器的 3.84%；

### 成交率 A/B 测试数据

![image](https://img.alicdn.com/imgextra/i4/O1CN01rEABaq1Kl1QnhqhUO_!!6000000001203-0-tps-3274-1788.jpg)

![image](https://img.alicdn.com/imgextra/i4/O1CN01CHBl5F1VWKV1dD64a_!!6000000002660-0-tps-986-1686.jpg)

&emsp;&emsp;经过线上 1%流量的验证，发现 Mobiles & Tablets 和 Sports & Outdoors 类别的成交率增长明显，分别从 1.79%提升至 3.61%，2.19%提升至 4.15%

&emsp;&emsp;同时也发现 Fashion 类的成交率，从 1.85%下降至 1.18%

&emsp;&emsp;Mobiles & Tablets 和 Fashion 类别作为全平台 GMV 贡献占比最高的 2 大类，却有着截然不同的效果，可能和商品的属性有着很大关联（个人猜测：电子类产品，如果没有详细的商品参数介绍、应用场景，就很难评估这个商品是否值得购买；而时尚类的商品主要用途为外观，因此看下主图就能知道外观如何，再看下面料材质什么的，就可以大差不大了）

&emsp;&emsp;因此，通过 A/B 测试的结果，就可以判断，如果用户上传的是电子类或者户外类等有明显成交率提升的类别，就可以推荐用户使用新的模版编辑器，如果是时尚等成交率下降的类别，就推荐默认编辑器
