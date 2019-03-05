# 学习笔记-clutter

## 1. 零碎概念

* 类：构造对象的模板。  
* 对象：实例（数据）。  
* 实例域：对象中的数据（每个特定的类实例，即对象，都有一组特定的实例域值）。  
* 方法：操纵数据的过程。  
* 封装：即数据隐藏，关键在于不能让类中的方法直接访问其他类的实例域。

---

## 2. Java中的修饰符

&emsp;&emsp;在Java学习的过程中经常会遇到各种各样的修饰符，有时候看过一眼就过去了，但是真正说到各种修饰符的区别还是会有点模糊，所以在这里按照类修饰符、成员变量修饰符以及方法修饰符的顺序做一个整理。  

① 类修饰符：

|修饰符|语义|
|---|---
|public|可被任何对象访问。`源文件中只能有一个公有类（即主类），Java源文件的名字与主类一致。`
|abstract|没有实现的方法，需要子类提供方法实现。
|final|不可以被任何类继承，通常是用来完成某种标准功能的类。

* 特别的，当class前面不加任何访问修饰符的时候，为“`默认访问模式`”，该类只能被同一个包中的类访问或引用。  

② 成员变量修饰符：

|修饰符|语义|
|---|---
|public|指定变量为公共变量，可以被任何对象的方法访问。
|private|指定变量不会被外部的其他类（`包括子类`）操作调用。
|protected|指定变量对所有子类可见，但是子类只能在自己的作用范围内访问自己继承的父类变量。
|final|指定变量的值不能更改。
|static|指定变量被所有的对象共享，所有的实例都可以使用该变量。

③ 方法修饰符：

|修饰符|语义|
|---|---
|public|指定方法为公共方法。
|private|指定方法只能被自己的类等方法访问，其他类（`包括子类`）不能访问。
|protected|指定方法可以被所有子类访问。
|final|指定方法不能被重载。
|static|指定方法不需要实例化即可激活，且静态方法无法访问非静态变量。

---

## 3. Mysql jdbc连接问题

### 错误log

>WARN: Establishing SSL connection without server's identity verification is not recommended. According to MySQL 5.5.45+, 5.6.26+ and 5.7.6+ requirements SSL connection must be established by default if explicit option isn't set. For compliance with existing applications not using SSL the verifyServerCertificate property is set to 'false'. You need either to explicitly disable SSL by setting useSSL=false, or set useSSL=true and provide truststore for server certificate verification.  
java.sql.SQLException: The server time zone value '???ú±ê×??±??' is unrecognized or represents more than one time zone. You must configure either the server or JDBC driver (via the serverTimezone configuration property) to use a more specifc time zone value if you want to utilize time zone support.

### 报错原因：

1. 项目中使用的驱动版本与Mysql版本不一致，驱动版本为Connector/J 8，Mysql版本为5.7，Mysql5.5之后的版本对安全性的要求更高，默认采用的是SSL连接，而驱动在连接时并没有相关配置，所以会有警告提示，但并不影响使用。  
2. mysql返回的时间有问题，比实际时间要早8小时。

### 解决方案：

1. 无安全性要求，在连接的URL后面加上useSSL=false显式声明即可.  
2. 在jdbc连接的url后面加上serverTimezone=GMT即可解决问题，如果需要使用gmt+8时区，需要写成GMT%2B8.

``` java
static final String DB_URL="jdbc:mysql://localhost:3306/test?useSSL=false&serverTimezone=GMT";
```

## 3.位运算符

java中的位运算符，可以应用的数据类型有`int`、`long`、`short`、`char`和`byte`等。位运算符作用在所有的位上，位运算的好处是可以快速地处理一些特定的问题。

|操作符|描述|
|---|---|
|&|对应位都是1为1，否则为0|
|\||对应位都是0为0，否则为1|
|^|对应位相同为1，否则为0|
|~|按位取反|
|<<|按位左移|
|>>|按位右移|
|>>>|按位右移补零|

典型的应用：leetcode136

## 4.树的一些概念

### 树结构概述

*根结点*： 同一棵树中除本身之外所有结点的祖先。  
*双亲结点*： 若一个结点含有子结点，则该结点称为其子结点的双亲结点。  
*子结点*： 一个结点含有的子树的根结点。  
*路径*： 从一个结点往下可以达到的孩子或孙子结点之间的通路。  
*结点的度*： 一个结点含有的子树的个数。  
*结点的权*： 给树中结点赋予的有着某种含义的数值。  
*叶子结点*： 度为0的结点  
*层*： 从根开始定义起，根为第1层，根的子结点为第2层，依此类推。  
*树的高度*： 树中结点的最大层次。  
*森林*： m（m>=0）棵互不相交的树的集合。  

#### 二叉树

*二叉树*：任何一个结点的子结点树不超过2。  
*满二叉树*：所有叶子结点都在最后一层，而且结点的总数为2^n-1，n是树的高度。  
*完全二叉树*：所有叶子结点都在最后一层或倒数第二层，且最后一层的叶子结点在左边连续，倒数第二层的叶子结点在右边连续。
