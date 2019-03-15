# 数据库概论

>暂时不整理数据库的内容了，[cyc大佬](https://cyc2018.github.io/CS-Notes/#/notes/%E6%95%B0%E6%8D%AE%E5%BA%93%E7%B3%BB%E7%BB%9F%E5%8E%9F%E7%90%86?id=%E4%BA%94%E3%80%81%E5%A4%9A%E7%89%88%E6%9C%AC%E5%B9%B6%E5%8F%91%E6%8E%A7%E5%88%B6)整理的太棒了！

## 0 目录

## 1 概述

### 1.1 数据库的基本概念

* 数据：描述事物的符号记录，是数据库中存储的基本对象
* 数据库：长期存储在计算机内、有组织、可共享的大量数据的集合
* 数据库管理系统：用户和操作系统之间的一层数据管理软件
* 数据库系统：引入数据库后的计算机系统

### 1.2 关系模型的基本概念

    数据模型三要素：数据结构、数据操作、完整性约束条件

* 关系：一个关系通常对应为一张表
* 元组：表中的一行为一个元组
* 属性：表中的一列即为一个属性，给每一个属性起一个名称即为属性名
* 码：表中的某个属性组，他可以唯一确定一个元组
* 域：属性的取值范围
* 分量：元组中的一个属性值
* 关系模式：对关系的描述

### 1.3 数据库系统三级模式结构

1. 模式（逻辑模式）：数据库中全体数据的逻辑结构和特征的描述，一个数据库只有一个模式
2. 外模式（用户模式）：数据库用户使用的局部数据的逻辑结构和特征的描述，每个用户只能看见和访问所对应的外模式中的数据
3. 内模式（存储模式）：数据物理结构和存储方式的描述，一个数据库只有一个内模式