





# 数据预处理

拿到的是Foursquare的数据集，数据的格式原文的readme是这样的

```txt
FourSquare dataset descroptions:

1. catergories.txt is a json file contains the tree information of all the categories in Foursquare.

2. Venues fold contains the Venues visited by the users in LA and NYC with the following fields:
Venue id | venue name| latitude| longitude| address|city|state| checkin #| checked user#| current user#| todo# |category # | category id 0| category id 1| ?
 
3. Users fold contains the user profiles of users who visited LA and NYC from foursquare, with the following fields:
User id| first name| lastname| profile pic| gender| home city|
 
4. Tips fold contains the series of tips from single user, with the following format:
User id| tip| tip|?
For each tip has the following format:
Venue id | 0 | null|  text| created time |todo#| done #| category # | category id 0| category id 1| ?

5. Friendship fold constains the user friendships with the following format:
userid | userid.

Thanks.

```

大致自己做了如下的翻译：

```txt
Foursquare数据集的描述：

1：类别文本文件是一个json格式的文件，包含Foursquare的所有的类别信息，它的数据格式为树形的信息

2：商户文件夹里面存放的是被洛杉矶和纽约的用户访问过的商户，它有如下的字段：
商户id | 商户的名称 | 纬度 | 经度 | 地址 | 城市 | 州 | 签到的总次数 | 签到的人数 | 当前的人数 | 
todo | 类别 | 类别id 0 | 类别id 1 | ?

3:用户文件夹里面存放的是用户的个人的信息，有如下的字段
用户id | 名 | 姓 | 头像 | 性别 | 家庭住址 |

4：小贴士（评论）文件夹包含每个用户一系列的评论数据 ，数据的格式如下所示：
用户id | 评论 | 评论 | ？
每一个评论的数据格式是如下所示的：
地点id | null | 论文的文本数据 | 评论的时间 | todo | done | 类别 | 类别id 0 | 类别id 1

5：用户的好友关系
用户id | 用户id
```

首先需要解释的就是类别文件的树形结构：

![mark](http://www.hhaxmm.cn/blog/20181213/mB2dbPo0LGrA.png)

可以看到总共是有8个大的类别的：这8个大的类别分别是：

Arts & Entertainment（艺术和娱乐） ，College & University（大学），Food（美食），Great Outdoors、Home,、Work、 Other（户外，家，办公场所），Nightlife Spot（夜生活场所），Shop（超市），Travel Spot（旅游景点）

![mark](http://www.hhaxmm.cn/blog/20181213/rzKPQfVye0cI.png)

从数据的格式中可以看出，每一个大的类别，下面有非常多的小的类别，每一个小的的类别是一个具体的类别，比如说Arts & Entertainment（艺术和娱乐）这个大的类别下面就有17个小的类别，小的类别下面可能还有小的类别，所以在商户的数据集里面就会出现 category | category id 0 | category id 1 |? ; 比如说在 NYC-Venues.txt的第一行对应的数据是 ：

```txt
  Venue name          category       category id
GB Productions(电影)       1	    4bf58dd8d48988d137941735
Girasole（意大利餐厅）       1	     4bf58dd8d48988d110941735
Verrazano-Narrows Bridge   2      4bf58dd8d48988d1df941735	4bf58dd8d48988d1f9931735
```

按照上面的推理：

1：对应的因该是College & University (大学类)

![mark](http://www.hhaxmm.cn/blog/20181213/LPVHNMENsgCb.png)

对于第二条数据4bf58dd8d48988d110941735对应的结果是——意大利餐厅

```txt
{"id":"4bf58dd8d48988d110941735","name":"Italian Restaurant","pluralName":"Italian Restaurants",
```

对于第三条数据4bf58dd8d48988d1df941735对应的结果是 第四类（Great Outdoors、Home,、Work、 Other）中的Bridge类别 ，实际的地名就是`韦拉札诺海峡大桥`也就是桥的类别

```txt
{"id":"4bf58dd8d48988d1df941735","name":"Bridge","pluralName":"Bridges","icon":"https://foursquare.com/img/categories/parks_outdoors/default.png","categories":[]}
```
对于第三条数据4bf58dd8d48988d1f9931735同时还有个类别是travel（旅行）中的高速公路，现实的生活中这个韦拉札诺海峡大桥来连接纽约市的史泰登岛与布鲁克林，作为Highway类别也是对的

```txt
{"id":"4bf58dd8d48988d1f9931735","name":"Highway or Road","pluralName":"Highways or Roads","icon":"https://foursquare.com/img/categories/travel/highway.png","categories":[]}
```

















