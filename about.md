---
layout: page
title: 简历
permalink: /about/
---

# 联系方式

- 手机： 13922304745 
- Email：contou.y@gmail.com
- QQ：   747613151
- 微信号： 13922304745

---

# 个人信息

 - 杨志冲/男/1992
  - 本科/南昌理工学院工商管理专业
   - 工作年限：3年
     - 技术博客：http://rickrickrickyyy.github.io
     - GitHub：http://github.com/rickrickrickyyy
     - 职位：Android高级程序员，应用架构师
     - 城市：广州
     - 我的优势：5年工作经验,熟悉java,scala,kotlin等语言,熟悉并在实际中应用过knn,kmeans,深度学习,opencv等技术.

         ---

# 工作经历

## 广州奥玄信息科技有限公司 (2017年9月 ~ 至今)
1.负责安卓，ios以及后端三个端口的软件开发。
2.使用c++语言开发安卓跟ios客户端使用摄像头测量脚数据算法的主要逻辑。
3.使用scala语言开发后端服务器Restful CRUD的api。
4.使用kotlin,java开发安卓端客户端。
5.使用swift开发ios客户端。
6.使用各个平台中不同的并发编程工具对各个平台中的关键代码进行性能优化。


## 在家学习spark (2017年3月 ~ 2017-9月)

### 1975年2015年天气变化地图
1.使用spark加载天气数据和地点数据,算出每个地点每年的平均气温。使用spatial interpolation方法生成每年对应的气温图。其中为了提高运算速度使用平均气温数据构建了quad tree,从而可以使用nO(logn)的运行效率来找到对应像素点上面最近的几个有气温数据的点。
2.使用1975-1989年的平均气温数据算出这段时间的平均气温数据,使用1990年到2015年每年的年平均气温数据减去1975-1989年的总平均气温数据,算出1990年到2015年每年相对于1975到1989年总平均气温的变化度，然后使用bilinear interpolation方法来生成1990年到2015年每年的气温变化图.
[项目展示](https://rickrickrickyyy.github.io/observatory/)

### StackOverflow问题分类
1.读取csv问题答案文件
2.将Rdd[String]转换为Rdd[问题]
3.根据问题id将数据分组
4.算出每个问题答案组的最高得分
5.使用kmeans 算法进行分类,其中在算新的中心点的时候根据spark的特性为了提高运行效率使用了reduceByKey函数，而不是使用groupByKey(这个方法导致数据shuffle,会在集群的机器中产生网络层的数据交换因此效率变低)(kmeans算法逻辑:1算新中心点 2算新中心点的得分converged,返回结果，否则回到1)


 
## 昭阳医生 （2016年3月 ~ 2017年2月）

### 昭阳医生app安卓端
全权负责安卓端app的开发,使用了databinding,recyclerview,realm,retrofit等技术,然后封装了TabActivity,BaseActivity,ListFragment,BaseListAdapter这些基础类使开发效率大大提高,代码量大大减少,维护成本大大降低。实施了之后同事跟领导觉得我的开发效率非常高,之后就一直改需求。这个项目中比较困难的地方是IM聊天列表界面和问卷编辑保存界面,因为这个界面里面有大量不同类型的ViewType而且item之间还有可能产生交互(例如多选题的选项),使用了mvvm的设计模式，最终简化了这两个界面的实现方法，并提高了代码的可读性可扩展性。

## 广州市红甘果科技有限公司 （ 2015年1月 ~ 2016年3月 ）

### 一加社区app安卓端
我在此项目负责了首页列表展示,幸运抽奖转盘。首页用了DrawerLayout来展示侧边栏,当时公司内常用的一个框架是SlidingMenu,使用了DrawerLayout之后使代码简洁了不少。这个项目中比较难的地方是抽奖转盘,因为要转几圈之后停到指定的地方去,在网上找了一些例子然后通过自己的理解,最终解决了这个问题。这个项目中最自豪的技术细节是用了官方自带的ripple effect    



### 新机宝
我在此项目负责了实用service来下载文件,还有其他一些列表类型的界面。使用了sqlite数据库来保存需要下载的任务的信息,使用了content provider来query数据,使用了一个Thread来拉取下载任务,然用用了ThreadPoolExecutor来执行下载任务,并限制下载任务的数量不能超过3个,在执行任务的同时还要将progress更新到sqlite数据库。



---

# 技能清单

以下均为我熟练使用的技能

- 数据库相关：MySQL/MongoDb/Cassandra
- 版本管理、文档和自动化部署工具：Git/Git Flow/Jenkins
- 单元测试：Junit
- 云和开放平台：/友盟/Jpush/腾讯开放平台/网易云信/微博开放平台/微信应用开发
- 后端框架：play/akka/spring/scala/iptables/linux

## 参考技能关键字

- android
- java
- ui
- http
- socket
- framework
- linux

---

# 致谢
感谢您花时间阅读我的简历，期待能有机会和您共事。
