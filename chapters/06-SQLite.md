# 八卦 (六）瑞士军刀： 随APP携带的SQLite

**八卦为主**


数据库打工仔在擦拭枪支时，喃喃自语的八卦历史。

**为导弹巡洋舰设计的数据库**

## 1. 产品设计
作为产品设计/产品管理的从业者，日常工作的一个核心就是明确产品的优势和定位。还有一个更加重要，又常常被忽略的维度： 产品**边界**

姚明是中锋，并不是说他不能像后卫一样投三分，他明确的知道自己的长处和定位，很能力边界。


前文讨论过postgreSQL，代表经典的数据库设计。作为稳定成熟的关系数据库产品，PG既具有较强的事务性能力，也有丰富的SQL功能带来的分析性能力，同时也提供了时空(Geometric Types, PostGiS)，文本（JSON/JSONB)，还有时序（Timescale, 基于PG的时序分析数据库）。PG是是全能全才吗？它的特点是清楚的模块结构（数据库教科书水平），可扩展性（FDW）和友好开源协议，是可以生长分枝的大树


我们今天就讨论一个数据库专注细分领域的，甚至可以说是小众的，但是又最流行的，产品**边界清晰**的数据库 **SQLite**



| ![SQLite Logo](../images/06-SQLite370.png?) |
|:--:|
| 图片来源 - SQLite.org|



## 2. SQLite - 兆级数据库 

从数据库实例维度，全世界最流行的数据库，不是那些数据库元老（O,DB2,SQL Server), 不是云上新贵（Aurora, Snowflake, Azure)，不是开源MySQL/PG, 不是大数据HBase/Spark。他们加在一起，也比不上SQLite的零头。

现今世界上正在运行的SQLite，有超过万亿（1000B/1e12)的实例l。每一位现代人的日常是被SQLite[^1]围绕的 : 

* 所有的Android手机/手环（华为，小米，三星...)
* 所有的苹果iOS设备手机/Pad/手表
* 所有的苹果电脑 Mac
* 所有的微软Windows10电脑
* 通用浏览器（Firefox, Chrome, and Safari）
* 智能电脑/机顶盒
* 车载多媒体系统
* .... 


SQLite不需要传统意义的安装，部署，调试，是纯粹意义上的[Zero-Configuration](https://www.sqlite.org/zeroconf.html)。不需要DBA, It just work for a developer，甚至许多i/OS, andriod手机APP的开发者，根本不知道他们在使用SQLite。

[^1]: https://www.sqlite.org/mostdeployed.html


## 3. SQLite 产品设计
产品设计，开发实现与应用实践，三者之间不是流水线或waterfall的单行道，而是循环往复，螺旋上升的。

SQLite由Dwayne Richard Hipp个人开发开源的。Hipp也算是计算机领中N多个从学校肄业的大牛之一，区别在于Hipp是在Duke读博士的时候才悟出来的，比Gates(大三） 和Jobs(大一）逊色不少。当年2000年甲方爸爸，世界五大国防工业提供商（俗称军火制造商）之一，列入Fortune 100的 通用动力公司（General Dynamics）需要为美国海军的提供软件系统，Hipp是其中一个临时工。

如果对GD不熟悉，那可以补课旗舰产品[F-16](https://en.wikipedia.org/wiki/General_Dynamics_F-16_Fighting_Falcon)

Hipp作为**政府的合同工**，恰巧因为联邦政府关门（对的，就是那种没有钱导致的政府关门），暂时失业了。待业在家（笔者刚刚短暂的享受了几天），Hipp搞出的事情就是写个数据库，处理个战舰损管控制系统。

### 2.1 需求

| ![驱逐舰奥斯卡奥斯丁\label{06驱逐舰奥斯卡奥斯丁}](../images/06_SQLite-USS_Oscar_Austin.jpeg?) |
|:--:|
| <b>驱逐舰 奥斯卡.奥斯丁 </b>|
| 舰名来自荣誉勋章获得者[奥斯丁下士](https://en.wikipedia.org/wiki/Oscar_P._Austin) |
| 图片来源 - wiki百科|

做数据库的产品经理可能很难遇到如此具体的需求，驱逐舰奥斯卡.奥斯丁上的某个任务对话框弹出DBA梦幻信息[^6]：
>  Can’t connect to database server

而这个任务是战斗损管系统，也就是在弹雨枪林下，水深火热中，需要报警表示那些管道/设施需要应急维修。真枪实弹的秒级交互场景中，如果出现**数据库无法联接**，操作员心中怎能不万马奔腾？

这个联系不上的数据库系统，是当年赫赫有名的Informix。虽然不如Oracle, Db2(那是还没有SQL Server的时代），Informix也是top5 的数据库，二十年前被IBM用10亿美刀收购，也算价值体系了。可是Informix不是为这个业务场景设计的，事实上当年没有任何一款数据库符合此场景。

[^6]: https://thenewstack.io/the-origin-story-of-sqlite-the-worlds-most-widely-used-database-software/

### 2.2 边界：**不是什么**

一方面nformix和已有数据库不适合上述场景，另一方面**新**产品也不用背负老系统的功能集和业务职责。

**NOT another RDBMS**：

* 不是替代当年成熟的数据库(Db2, Oracle)
* 不是(2000年代）新宠的数据仓库（Netezza, greenplum)
* 不是复杂的服务器Server-base系统
* 速度不是重要指标
* 存储量也不是重要指标
* 跨平台不是设计指标（对比2000 java的崛起）
* 功能齐全不是设计目的

### 2.3 定位：为单一APP服务的数据库 

SQLite是自己自足**Self contained**的关系数据库管理系统，直接服务某应用某块。此概念与现在流行的microservices有类似之处。

SQLite面临的不是提高Informix链接数，复杂的链接池算法，或者断点续连的问题。而是面向的是每一个控制模块需要恶劣环境下，甚至出现物理切割的情况下，单模块系统依然可以对立运行，完成大部分设计功能。特别明确一下，这里的模块指代某个硬软件一体的工业模块，不是数据库内部的纯软件模块。

让我们做一个业务对比： 现在中国的银行系统是采用中心体系的，每次一个支行或柜员机的业务操作是与总行数据中心连接完成的。好处是用户可以跨区域跨分行操作，异地存储转账，不足是如果与总行的线路出现问题，分行是瘫痪的。而SQLite的定位是，在正常情况下与总枢纽指挥中心联络，通讯通道异常时，比如前炮台，后轮机，均需要独立完成**90%以上**的职责。



### 2.4 特点（优点）：**NOT Faster, Better, Cheaper**

咬文嚼字的说，**优点**是销售词汇，产品设计角度应该强调的是**特点**。特点在适合的场景中才是优势。SQLite，不是[最快最大](06-SQLite.md#42-描述产品的片儿汤话）的数据库，恰恰相反它是最小的标准数据库RDBMS。

SQLite的特点是如此明显的，所以宣传它不用那些看不出产品特点的片儿汤话[^8]。事实上SQLite的早期成功同商务宣传没有半毛钱关系。它为一个特殊的细分市场提供了具有基础数据库能力的嵌入式系统，这个细分市场在短短几年跳跃进庞大的智能手机时代，而SQLite的生命力强大到轻轻松松的站在时代的浪尖。


[^8]: https://www.baike.com/wiki/%E7%89%87%E5%84%BF%E6%B1%A4%E8%AF%9D


#### 2.4.1 小：*small and simple*

通过C编写，在正式发布5年之后，向Google/Andriod的推广时，SQLite的binary也不过250KB。出世20多年过程中，添加了全文检索，CTE，JSON等高级功能后，SQLite的发布版也只有小小的700KB。

今日头条（ andriod版）的安装后大小161MB，1:200的关系。对了，今日头条APP中很可能也用了SQLite（请字节同学确认，同样问题抛给阿里淘宝，腾讯微信）

SQLite其实很"大"，3.33.0(2020GA)可以最大支持281TB的数据。其系统测试分四大类，几百万测试case。即使最小的，为开发程序员日常check in的把关的“very quick”的Tcl 测试，也有30万个。它对产品质量的重视，是否可以让许多大厂的号称企业级的系统测试汗颜了[^9]。

[^9]: How SQLite Is Tested https://sqlite.org/testing.html

#### 2.4.2 标准数据库

	1. 支持数据库最重要的事务ACID。
	2. 兼容标准的SQL syntax，SQLite 1.0（2000年） 使用PostgreSQL 6.5(1999年) 语法。现在支持大部分SQL-92标准 
	3. C/C++ interface作为原始编程语言接口，为后期衍生开发提供可扩展的链接

#### 2.4.3 嵌入式

大家熟悉数据库系统，Oracle, DB2, MySQL等，是存在于应用程序之外的独立系统，一个Oracle为多个应用服务。还以银行系统为例，应用(application)包括存取（强事务write)，流水单（单储户Read Only），储户留存分析（月底报表, BI Report)，都可以在同一个Oracle集群上操作。

SQLite是嵌入式的数据库，作为应用（APP）的一个部件，同时安装，同应用和用户常常都是一对一关系。SQLlite的**小**赋予了它可以被嵌入的能力。

#### 2.4.4 利用文件系统

定位决定了特殊性：并发少，权限管理简单，性能要求不高。

SQLite不用像系统级数据库，深度管理定制的存储管理系统，比如MySQL 开发自己的innoDB, TiDB 采用TiKV和RocksDB。SQLite依赖操作系统自带的文件系统，读写自己DB file，并且继承文件系统的权限管理。

此设计理念简化系统复杂度，但也并不是没有缺陷。并发读写就是SQLite的明显短板之一。因为整个数据库是一个大文件，依靠文件锁来控制读写冲突。只有在后期（2010）实现[WAL](https://sqlite.org/wal.html)后，才提供了并发功能，同时也不是没有代价的。

#### 2.4.5 Serverless

大家常常混淆了“Serverless”这个技术，与云Serverless Computing 这个技术+业务手段，比如[Serverless Database] (https://en.wikipedia.org/wiki/Serverless_computing#Serverless_databases)，其实还是client/server的服务架构，准确的说是有服务器(Server)的。

Serverless作为一个技术，其经典定义（现在比较小众了）就是纸面意思：没有服务器/no server.。SQLite是Serverless，因为它与应用程序的同一个进程内运行，公用共享同一块内存空间，相互之间直接读写，而不通过消息协​​议(比如RPC call）和网络交互。



#### 2.4.6 "官方"认证

美国国会图书馆，又称美国国家图书馆（Library of Congress) 推荐的独立于平台的开放格式的四种数据存储格式之一 [^2]

此官方认证云国内的政府认证，语音类似，差距很大。

[^2]: https://www.loc.gov/preservation/resources/rfs/data.html


## 3. 马后炮，评英雄

《成功学》最重要的优势就在于：面对一个已经成功的产品，项目，人或者团队，总结（堆砌）其英明决策，其实历史偶然性多于必然性，尤其是具体的人和情。

回头看SQLite, 也是可以总结一些契机和事后方知的因素，促成了SQLite的今天。

### 3.1 项目 vs. 产品

SQlite肯定不是某领导英明决策的结果。本来是个项目（解决Informix掉链问题），Hipp闲的无聊把它作成了产品。

反观许多产品团队（国内较多）打着产品的名号做项目。大部分是短期业务压力下，或者是因为大厂内部竞争，或者因为初创企业生存压力。团队早期做有生命力有世界水平数据库的梦想，很快向现实环境低头了。被甲方爸爸或领导指挥左右，对自己的产品拔苗助长。项目也许拿下了的代价是产品做残了。
<!--
这里不做具体分析了，简单的说国内较权威的摩天轮上列述了[250多国产数据库](https://www.modb.pro/db/24797)，
-->
### 3.2 开源


SQLite从第一天就是开源的，特别要明确一下，不是开源项目。

Hipp一个人开发，开源的。他没有想到SQLite二十年后支持了现代人的分分秒秒的日常生活。甚至在头几年，Hipp都不知道谁使用，搞笑的是他之所以了解到通用电气和日立青睐SQLite，是因为出口管制必要法律流程触发的。

很幸运的是，Hipp这个工程师得到一个有法务背景的IT商人， Mitchell Baker， CEO of Mozilla Foundation，指导和帮助。SQLite才真正成为一个项目[SQLite联盟](https://www.sqlite.org/consortium.htm) ，有了稳定的资金，同时又保留了开发者主导决策产品的发展的权利。


### 3.3 Google和Andriod

SQLite唯一获得的奖项是2005 Google O’Reilly Open Source Award。

当iphone/智能手机被业界认为是后PC革命的时候，smartphone已经开始使用SQLite, Symbian（Nokia）属于最早的之一。

Google/Andriod在iphone获得早期风光之后，也走进了掌上的舞台。他们选中了SQLite。于是所有的Andriod APP, 都用SQLite作为默认的数据库管理。实际情况是APP开发者并不了解数据库，也没有动力去选型。APP使用SQLite因为Google/Andriod是选择（平台作用），也是所有APP教程的选择（生态作用）。

### 3.4 测试

SQLite突然直面了百万的用户，各种bug如雨后春笋一般暴露出来。Hipp花了整整一年时间写测试用例。大多数产品在GA，商业化之后，很难有时间和机会专门提供测试覆盖和产品质量。Hipp是幸运的，在关键时刻等到认可和资助；SQLite是幸运的，它是**开发者的孩子**而不是简单的赚钱的工具。SQLite像是成长中的少年，等到了机会，吸收了营养，长大成人。



<!--
## 其他同行
Realm

Firebase
-->

## 4. 一点感触

### 4.1 业务引导技术方向 - 国内和国际不同的实践

技术服务业务，是行内比较普遍的认知，合乎逻辑。但当我们看到细节的时候，会发现国际和国内的明显不同。

中国和欧美IT技术开发既有相似处，比如美国为联邦政府提供IT软件服务也有类似中国的认证流程，本着对纳税人负责的态度，严谨但也死板，同时政府也常常是重要的金主甲方。不同点是，中国政府可以集中力量办大事，远的高铁，近期的芯片和新能源汽车，政府的方向性是明确清晰的。国际上，技术革新的大方向常常是技术公司和商业引导的，比如说Telsa/SpaceX的跨时代的突破，基本上没有政府的引导，更准确的说是Telsa引导（游说）政府给以免税政策。

具体到SQLite这个国家图书馆的“官方”认证，说起来很有力，其实对于SQLite的成败影响力就非常有限了。

如果读者的产品计划进军国际市场，有点个人建议： 
* 业务销售人员要积极关注政府合规认证；
* 技术开发人员专注设计前沿和产品实现，切记迎合政府。

### 4.2 描述产品的片儿汤话
<!--
产品介绍的时候，常常听到**差异化优势**这个词语，潜意识中，我们甚至把差异和优势等同起来。
-->

常见的某个产品的商业口号是非常类似的，基本上是*快好省*的衍生词汇。 

比如："与 MySQL 和 PostgreSQL 兼容的关系数据库，专为云而打造。性能和可用性与商用数据库相当，成本只有其 1/10" , 2022年九月十三日摘抄[AWS Aurora 官网](https://aws.amazon.com/cn/rds/aurora/)。如果是某个领域的先行者，比如Aurora，倒也罢了，因为是采用了某个突破性新技术，可以担当*快好省*的评价。之后的追随者，也如是定位宣传，就有些东施效颦的偷懒和尴尬了。

PPT/胶片是技术产品人员常常使用的工具，评价一个材料的好坏，用心与否，可以把PPT中的产品名字盖住，用模版的把颜色和字符统一一下，是否还能看出是哪一个大厂的哪一款产品？粗略估计，80%的产品描述过不了此关。

<!--
甚至从理论角度*多快好省*是思想初衷是好的，违反客观条件和自然规律，且没有科学依据的的宣传词汇[^7]

[^7]: https://www.baike.com/wiki/%E5%A4%9A%E5%BF%AB%E5%A5%BD%E7%9C%81
-->


## 5. 八卦篇

提到小而精的经典程序，必须跪拜一下阿波罗登月计划中的**AGC系统**[^3][^4]，提供登月过程中航天器的制导、导航和控制。全部系统安装在72KB的只读ROM里(头条APP的2千分之一），运行空间是4KB(byte) RAM（约为本篇Markdown文本的四分之一）。

友情奉送[Github](https://github.com/chrislgarry/Apollo-11/blob/master/README.zh_cn.md)打卡地址，膜拜一下阿波罗 11 号导航软件AGC中指令模块（Comanche055）和登月模块（Luminary099）原码。

[^3]: https://www.discovermagazine.com/the-sciences/apollo-11s-1202-alarm-explained

[^4]: https://en.wikipedia.org/wiki/Apollo_Guidance_Computer


[^1]: https://www.sqlite.org/mostdeployed.html
[^2]: https://www.loc.gov/preservation/resources/rfs/data.html
[^3]: https://www.discovermagazine.com/the-sciences/apollo-11s-1202-alarm-explained
[^4]: https://en.wikipedia.org/wiki/Apollo_Guidance_Computer
[^5]: https://github.com/chrislgarry/Apollo-11/
[^6]: https://thenewstack.io/the-origin-story-of-sqlite-the-worlds-most-widely-used-database-software/

[^9]: https://sqlite.org/testing.html, How SQLite Is Tested 

[^7]: https://www.baike.com/wiki/%E5%A4%9A%E5%BF%AB%E5%A5%BD%E7%9C%81

[^8]: https://www.baike.com/wiki/%E7%89%87%E5%84%BF%E6%B1%A4%E8%AF%9D

