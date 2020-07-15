# 个人项目
# 主题：基于MySQL全文索引构建简单的中文"搜索引擎"
# 要求：
  * 个人独立完成，写一篇实践小论文，并用编写实验过程的实验代理
  + 需要完成可运行的实验代码
 + 需要有实验结果截图
 + 需要分享心得
  
---
## 前言
#### 这个主题的难点其实是在“中文”两字上，mysql原本的全文搜索就支持英文搜索，但由于中文不是自动分词的语言，所以原本的mysql并不支持中文全文检索，但是mysql5.7 以上，就多了插件ngram，使得我们可以进行中文检索。
#### 所以，我在发现原本的mysql不能用ngram对中文分词后，先将mysql升级到了5.7版本
---
## 1 建表book并导入测试数据
```
-- phpMyAdmin SQL Dump
-- version 4.0.10.11
-- http://www.phpmyadmin.net
--
-- 主机: 127.0.0.1
-- 生成日期: 2020-07-15 23:35:08
-- 服务器版本: 5.7.30-log
-- PHP 版本: 5.3.29

SET SQL_MODE = "NO_AUTO_VALUE_ON_ZERO";
SET time_zone = "+00:00";


/*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
/*!40101 SET @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS */;
/*!40101 SET @OLD_COLLATION_CONNECTION=@@COLLATION_CONNECTION */;
/*!40101 SET NAMES utf8 */;

--
-- 数据库: `searchengine`
--

-- --------------------------------------------------------

--
-- 表的结构 `book`
--

CREATE TABLE IF NOT EXISTS `book` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `title` varchar(255) DEFAULT NULL,
  `content` text,
  PRIMARY KEY (`id`) USING BTREE,
  KEY `title` (`title`),
  FULLTEXT KEY `fulltext_title_content` (`title`,`content`) /*!50100 WITH PARSER `ngram` */ 
) ENGINE=InnoDB  DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC AUTO_INCREMENT=15 ;

--
-- 转存表中的数据 `book`
--

INSERT INTO `book` (`id`, `title`, `content`) VALUES
(1, '稻草人手记', '麦田已经快收割完了，农夫的孩子拉着稻草人的衣袖，说“来，我带你回家去休息吧!”稻草人望了望那一小片还在田里的麦子，不放心的说“再守几天吧，说不定鸟儿们还会来偷食呢!”孩子回去了，稻草人孤孤单单的守着麦田。'),
(2, '平凡的世界', '作品高度浓缩了中国西北农村的历史变迁过程，作品达到了思想性与艺术性的高度统一，特别是主人公面对困境艰苦奋斗的精神，对今天的大学生朋友仍有启迪。这是一部全景式地表现中国当代城乡社会生活的长篇小说。全书共三部。'),
(3, '四世同堂', '小说在卢沟桥事变爆发、北平沦陷的时代背景下，以祁家四世同堂的生活为主线，形象、真切地描绘了以小羊圈胡同住户为代表的各个阶层、各色人等的荣辱浮沉、生死存亡。'),
(4, '花田半亩', '一个只有20岁的女孩面对死亡时的沉静，她对病痛的体会，她对生命的洞察，像一个漩涡，把读者也同时卷入。让人们看见那些疼痛蔓延的夜晚，一个孩子，用她年轻的光芒将整个生命照亮。'),
(5, '围城', '小说没有明确的故事线索，只是一些由作者琐碎的见识和经历”拼凑”成的琐碎的情节。就一般而言，情节琐碎的书必然要有绝佳的言语表达才能成为一本成功的作品。'),
(6, '送你一颗子弹', '这本书里记录的是作者2005—2009年左右(尤其是2006—2007年)生活里的点点滴滴。在这本书里，被“审视”的东西杂七杂八，有街上的疯老头，有同宿舍的室友，有爱情、电影和书，大到制度，小到老鼠。'),
(7, '活着', '地主少爷富贵嗜赌成性，终于赌光了家业一贫如洗，穷困之中的富贵因为母亲生病前去求医，没想到半路上被国民党部队抓了壮丁，后被解放军所俘虏，回到家乡他才知道母亲已经去世。'),
(8, '呐喊', '作品真实地描绘了从辛亥革命到五四运动时期的社会生活，从革命民主主义出发，抱着启蒙主义目的和人道主义精神，揭示了种种深层次的社会矛盾。\r\n\r\n'),
(9, '家', '小说描写了高家四代人的生活，并将他们设置为新旧两大阵营。以高老太爷、冯乐山、高克明、周伯涛以及高克安、高克定为代表的老一辈统治者，他们专横颟顸，虚伪顽固。'),
(10, '人生', '天闷热得像一口大蒸笼，黑沉沉的乌云正从西边的老牛山那边铺过来。地平线上，已经有一些零碎而短促的闪电。但还没有打雷。只听见那低沉的、连续不断的嗡嗡声从远方的天空传来。'),
(11, '无限恐怖', '除了放在现在依然独特别致的创意之外，还有着许多优点，精彩的剧情，优秀的人物塑造等。如果你还没看过就一定不要错过。'),
(12, '长生界', '这部小说虚构了一个超脱与人世间之外的长生界，主角萧晨被动追寻上古遗秘的踪迹，引起了无数光怪陆离的故事。');

/*!40101 SET CHARACTER_SET_CLIENT=@OLD_CHARACTER_SET_CLIENT */;
/*!40101 SET CHARACTER_SET_RESULTS=@OLD_CHARACTER_SET_RESULTS */;
/*!40101 SET COLLATION_CONNECTION=@OLD_COLLATION_CONNECTION */;

```

### 建表结果
### 这是一个很简单的表，我在title和content字段建立了全文索引。
![表结构.jpg](https://github.com/Lily-161/-/blob/master/个人项目/image%20file/表结构.jpg)
## 2 搜索
### 2.1试试以“小说”为关键词进行搜索，搜索代码：
```
select *, MATCH (title, content) AGAINST ('小说') as score
from book where MATCH (title, content) AGAINST ('小说' IN NATURAL LANGUAGE MODE)
```
### 搜索结果
![Image text](https://github.com/Lily-161/-/blob/master/个人项目/image%20file/结果1.jpg)
![Image text]()
---
### 2.2 试试以“四世同堂”为关键词进行搜索，搜索代码：
```
select *, MATCH (title, content) AGAINST ('四世同堂') as score
from book where MATCH (title, content) AGAINST ('四世同堂' IN NATURAL LANGUAGE MODE)
```
### 搜索结果 
![Image text](https://github.com/Lily-161/-/blob/master/个人项目/image%20file/结果2.jpg)
---
### 2.3 试试以“四世同”为关键词进行搜索，搜索代码：
```
select *, MATCH (title, content) AGAINST ('四世同') as score
from book where MATCH (title, content) AGAINST ('四世同' IN NATURAL LANGUAGE MODE)
```
### 搜索结果
![Image text](https://github.com/Lily-161/-/blob/master/个人项目/image%20file/结果3.jpg)
---

### 2.3 试试以“四”为关键词进行搜索，搜索代码：
```
select *, MATCH (title, content) AGAINST ('四') as score
from book where MATCH (title, content) AGAINST ('四' IN NATURAL LANGUAGE MODE)
```
### 搜索结果
![Image text](https://github.com/Lily-161/-/blob/master/个人项目/image%20file/结果4.jpg)

#### 可以发现，出错了，竟然没有搜索结果？

---
#### 当我用一个关键字搜索时，搜索出错，其原因在于我的分词机制，我的分词是默认以2个字分词的，即四世同堂会分为“四世”和“同堂”，而“四”不在我的分词范围内，所以解决办法很简单，把分词长度设为1就好，方法如下：
#### 在配置文件 my.ini 中修改 ngram_token_size = 1 ，并重启 mysqld 服务。
---
#### 现在再来测试，结果成功了：
![Image text](https://github.com/Lily-161/-/blob/master/个人项目/image%20file/结果5.jpg)
---
## 心得
### 这个搜索引擎还是非常基础，他不能一次性把不同分词长度的关键词检索出来，但是我觉得可以用php网页稍微完善一下，做一个前端，比如做几个网页：“单字搜索”页面，“标题搜索页面”、“内容搜索页面”，及“全文检索页面”，每个页面用相应地SQL语句就行了。



