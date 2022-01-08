# 前言 - 老枪的思考

**此文以八卦为主，如果认真，你我就都输了**


数据库打工仔在擦拭枪支时，喃喃自语的八卦历史。

多年前的一个晚上，擦拭着“新”玩具 --
一把1940s年代的[Mosin–Nagant莫辛-納甘步槍](https://zh.wikipedia.org/wiki/%E8%8E%AB%E8%BE%9B-%E7%B4%8D%E7%94%98%E6%AD%A5%E6%A7%8D)，思路还神游在刚才的数据库架构讨论会议中：
反复强调可靠性、通用性、兼容性、产品化；
一次次竞品对标性能，功能，产品成熟性，易用，可维护，低成本.....


## 老枪
在仰望星空在PPT中设计世界第一的数据库产品的时候，
看看手里这把1891年设计，架构跨越三个世纪的老枪，
不就是一个最经典的产品设计吗？

1944年出厂，加上半个世纪前的军剩弹，***可靠性***无出其右

![The old gun\label{01-LeVieueFuisl}](../images/01-LeVieuxFusil.jpg?raw=true)


## 数据库
早在大数据这个名字出现之前，各种各样的数据源已经此起彼伏的挑战数据库管理系统：
关系型，面向对象，XML/JSON，KV，宽表，图，plaintext,
image/video/audio，etc.

当我们浅显的以为这是21世纪数据库的面对的新挑战的时候，
一不小心低头看了看100年前工程师们面对着这些***数据源***：
- 跨平台（手枪 vs 步枪），
- HTAP（全手动，半自动，全自动），
- 多模（shotgun)
前人大神们不只是设计好了，而且已经上线多年了

![Ammo and Data format\label{01-Ammo Data Format}](../images/01-AmmoData.jpg?raw=true)


## 性能是什么？

几乎每隔几个月，都能够看到某某DB发表了性能/performance报告，
一般来说，都是用倍这个单位，只是30%都不好意思出来写软文。

比如某数据库两个月前的[优秀博文：性能提升100倍！](https://bbs.huaweicloud.com/forum/forum.php?mod=viewthread&tid=170723&extra=page%3D1)，赫然在官网上。其实读进去，博文还是有些干货的，属于合格的技术文章，而且某个或某几个query提升10X～100X都是有可能的（请搞优化器的同学们来深究）。可是上面文章的题目怎么都像**传销**或**网络诈骗**：“某某某某... 10倍收益不是梦！！！” 。这里挑战一下做NLP和语言AI分析的同学：你的模型能否把数据库大厂关于性能提高的博文们都撸一遍，简单叠算，看看是不是已经提高过千倍万倍了？

后来内卷还是正规了不少，采用TPC为标准，虽然也有它的偏薄，
至少是同一个平台上有监督的竞争
![TPC Benchmarks Timeline
Overview\label{01-TPC}](../images/01-TPCtimeline.jpeg?raw=true)

对于数据库benchmark真正有兴趣，又喜欢八卦的同学，肯定应该听到过最近的databricks vs snowflake的关于TPC-DS的PK：

* [2021-11-02: Databricks Sets Official Data Warehousing Performance Record](https://databricks.com/blog/2021/11/02/databricks-sets-official-data-warehousing-performance-record.html) by Databricks founder， 声称"... Databricks was 2.7x faster ..."
* [2021-11-12: Industry Benchmarks and Competing with Integrity ](https://www.snowflake.com/blog/industry-benchmarks-and-competing-with-integrity/) by snowflake founder and CEO
* [2021-11-15: Snowflake Claims Similar Price/Performance to Databricks, but Not So Fast!](https://databricks.com/blog/2021/11/15/snowflake-claims-similar-price-performance-to-databricks-but-not-so-fast.html)



论战的是是非对错，暂且放到一边，的的确确的高手过招，成了不少专业人士的谈资的。看热闹的人很多，高手也年末总结发言[Databases in 2021: A Year in Review](https://ottertune.com/blog/2021-databases-retrospective/) ， CMU的Pavlo很阿里味“简单真诚”的评论“only old people care about official TPC numbers.” 只有老头们才关心TPC。

我这里只八卦一个问题：为什么databricks claims 仅仅**2.7X**的性能超越，变被对手如此重视，而某些软文动辄十倍百倍，而无人理睬呢？哦对了，Databricks和snowflake在12月底刚刚发布的Gartner云数据库象限报告中，双双进入领导者象限。

思维总是奇怪跳跃的我，想起《神雕侠侣》最后一回 《华山之巅》，愿意跳跃的同学请自己移步去读头三页。


## 性能不是什么？ 
刚刚入行的时候以为数据库跑的最快就是最好的。
略懂一二后，发现跑得快的定义就很复杂：
- 延时(latency),
- 吞吐量（throughput）
- 数据的时效性（realtime)
etc.

那一把火器的性能又是怎么定义：打的远，打的准，打的快，子弹效率高？

也许数据库的设计可以从过去几百年中最受重视的另一个工程产品中学到点什么？
没有任何一个系统能做到***既要也要***，
数据库做不到[CAP](https://en.wikipedia.org/wiki/CAP_theorem)，
枪也做不到快准远狠兼顾，还能价廉物美的批量生产。

写到这里，思路乱的很，好似被改进提高N多次的系统，不知道该从哪里说起。那就下回分解吧


# 目录
1. 前言 - 老枪的思考
2. 数据库系统可靠性与Mosin Nagant

<!--
3. 模块设计 PostgreSQL vs. AR15
4. 朴实无华 MySQL vs. AK47
5. 有容乃大 MongoDB vs. Shotgun霰弹枪 
6. 兼容并包 Opensource vs. OpenAPI Remington700
7. 瑞士军刀 SQLite vs. 10/22 takedown
8. 扩展性(Scalability): Revolver左轮 vs semi-auto半自动手枪
9. SKS(五六半) 
10. 可用性 （Make it work, Make it faster, Make it cheaper) - 扳机，timezone
...
99. 极致性能 
-->

#### 此文以八卦为主， 如果认真，你我就都输了

![](../images/ywm.gif?raw=true)
