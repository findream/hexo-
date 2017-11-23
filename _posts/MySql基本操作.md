---
title: MySql基本操作
date: 2017-10-14 20:45:11
tags:
---
# 数据库简介：</br>
**&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;数据库(Database)是按照数据结构来组织、存储和管理数据的仓库，它产生于距今六十多年前，随着信息技术和市场的发展，特别是二十世纪九十年代以后，数据管理不再仅仅是存储和管理数据，而转变成用户所需要的各种数据管理的方式。数据库有很多种类型，从最简单的存储有各种数据的表格到能够进行海量数据存储的大型数据库系统都在各个方面得到了广泛的应用......**
<!-- more -->
* **第一部分：创建数据库及其使用**
    * <font color=#DC143C>1)创建数据库：  **Create database [数据库名字];**</font>
    * <font color=#DC143C>2)使用数据库：  **Use [数据库名];**</font>
    * <font color=#DC143C>3)查看数据库：  **Show databases;**</font>
    * <font color=#DC143C>4)查看正在使用的数据库：**select database();**</font></br>
   ![](https://i.imgur.com/aPE9qWN.png)
</br>
---
* **第二部分：数据表的创建和使用**
    * <font color=#DC143C>1)数据表的创建：**create table [表名]（列名 类型名，………）;**</font>
   ![](https://i.imgur.com/u6VhVYW.png)
   ![](https://i.imgur.com/WXrp6DT.png)
   ![](https://i.imgur.com/Z2E3xvd.png)
    * <font color=#DC143C>2)外键约束：**foreign key【（属性）】 references 【字表名】【（属性名）】;**</font>
   ![](http://oxnvtxe03.bkt.clouddn.com/4.png)    
   ![](http://oxnvtxe03.bkt.clouddn.com/5.png)
    * <font color=#DC143C>3)数据表的查看：**show tables；**</font>
    * <font color=#DC143C>4)查看表中每一列的元素：  **show columns from【数据表名】;**</font>
   ![](http://oxnvtxe03.bkt.clouddn.com/6.png)
    * <font color=#DC143C> 5）建立索引: **create unique/cluster index [索引名] on [表名][(列名，asc、desc)；**</font>
   ![](http://oxnvtxe03.bkt.clouddn.com/1.png)
</br>
---
* **第三部分：数据表的修改：**
    * <font color=#DC143C> 1）增加列: **alter tabele [表名] add [属性/列名] 数据格式 【first/after+[列名]】**</font>
   ![](http://oxnvtxe03.bkt.clouddn.com/2.png)
    * <font color=#DC143C> 2）删除列：**alter table [表名] drop
   column [列名];**</font></br>
   ![](https://i.imgur.com/wPEr183.png)
    * <font color=#DC143C> 2) 修改列的属性:   **alter table [表名] alter column [列名][数据类型]**</font>
    * <font color=#DC143C> 给列添加主键约束:   **alter table [表名] add  primary key/ unique key([列名]);**</font>
   ![](http://oxnvtxe03.bkt.clouddn.com/1111.png)
   ![](http://oxnvtxe03.bkt.clouddn.com/1111.png) 
    * <font color=#DC143C> 3)删除默认约束：**alter table [表名] alter [列名] drop default;**</font>
   ![](http://oxnvtxe03.bkt.clouddn.com/7.png)
    * <font color=#DC143C> 4)删除主键约束：**alter table [表名] drop primary key;**</font>
   ![](http://oxnvtxe03.bkt.clouddn.com/8.png)
    * <font color=#DC143C>5)删除唯一约束:**alter table [表名] drop index [列名];**</font></br>
   ![](http://oxnvtxe03.bkt.clouddn.com/10.png)
    * <font color=#DC143C>6)删除外键约束：**alter table [表名] drop foreign key [约束名称，通过show create table [表名查看];**</font>
   ![](http://oxnvtxe03.bkt.clouddn.com/11.png)
   ![](http://oxnvtxe03.bkt.clouddn.com/12.png)
   ![](http://oxnvtxe03.bkt.clouddn.com/13.png)
    * <font color=#DC143C>7)删除表:   **drop table [表名];**</font></br>
   ![](http://oxnvtxe03.bkt.clouddn.com/14.png)
</br>
---
* **第四部分：对数据的修改：**
    * <font color=#DC143C> 1)插入全部数据：**insert 【数据表名】values(数据1，数据2……)；**</font><br>
   ![](http://oxnvtxe03.bkt.clouddn.com/15.png)
    * <font color=#DC143C> 2)插入部分数据：**Insert [数据表名]（属性名1，属性名2……）values(数据1，数据2);**</font>
   ![](http://oxnvtxe03.bkt.clouddn.com/16.png)
    * <font color=#DC143C> 3)修改数据：**Insert [数据表名] set (列名1)=（表达式1）………；**</font>
   ![](http://oxnvtxe03.bkt.clouddn.com/17.png)
    *  <font color=#DC143C> 4)复制表的数据：</font></br>
   ![](http://oxnvtxe03.bkt.clouddn.com/23.png)
   ![](https://i.imgur.com/BMfOmdw.png)
    * <font color=#DC143C> 5)更新数据表单的数据：**update table [数据表名] set [表达式]  where [条件] (条件语句为可选语句)**</font>
   ![](http://oxnvtxe03.bkt.clouddn.com/19.png)  
   ![](http://oxnvtxe03.bkt.clouddn.com/18.png)
    *  <font color=#DC143C> 6)删除元组：**delete from [表名] where[条件]**</font></br>
   ![](http://oxnvtxe03.bkt.clouddn.com/22.png)
</br>
---
* **第五部分：查询语句：**
    * <font color=#DC143C> 1)查询记录： ** select * from 【数据表名】;**</font></br>
   ![](http://oxnvtxe03.bkt.clouddn.com/20.png) 
    * <font color=#DC143C> 2）多表查询： **select [表名].[列名]....from [数据表名]；**</font>
   ![](http://oxnvtxe03.bkt.clouddn.com/24.png)
    * <font color=#DC143C> 3)定义别名：**select 【列名】as [别名] from [表名]；**</front>
   ![](http://oxnvtxe03.bkt.clouddn.com/25..png)
    * <font color=#DC143C> 4) 条件查询： **select 【查询内容】from 【数据表名】where 【条件表达式】;**</font>
   ![](http://oxnvtxe03.bkt.clouddn.com/26.png)
    * <font color=#DC143C> 5） 分组：**select 【查询内容】from 【数据表名】group by [列名表达式]**</front>
   ![](https://i.imgur.com/BVBjrbh.png)
    * <font color=#DC143C> 6）having语句：**select 【查询内容】from 【数据表名】group by [列名表达式] having 【条件】**</front>
   ![](https://i.imgur.com/U3N8EHO.png)
    * <font color=#DC143C> 6）order by语句：**select 【查询内容】from 【数据表名】order by [列名表达式]**</front>
   ![](https://i.imgur.com/e48tUT2.png)
    * <font color=#DC143C> 6）having语句：**select 【查询内容】from 【数据表名】limit n(,m)**</front>
   ![](https://i.imgur.com/UzudAAh.png)
   ![](https://i.imgur.com/mchgJk2.png)
</br>
---
* **第六部分：高级查询语句**
   * <font color=#DC143C>子查询：
       * 不相关子查询：
           * 1.子查询语句不能使用order by。
           * 2.内层查询不依赖外部查询。
           * 3.子查询并不显示结果，只是返回给外部查询
           * 4.返回多个结果可以用any或者all来表示。
           * 5.也可以使用聚合函数（max，min等）。
           * 6.谓词表和谓词和聚集函数及in谓词的等价关系
           ![](https://i.imgur.com/1QiNhwo.png)
           ![](https://i.imgur.com/3afujFC.png)
           * 例子1：（in或者not in）
           ![](https://i.imgur.com/fQQL2nH.jpg)
           * 例子2： （比较运算符）
           ![](https://i.imgur.com/J98u6i6.jpg)
           * 例子3：  (谓词和聚集函数)
           ![](https://i.imgur.com/FdpTiEX.jpg)
       * 相关子查询：
           * 依赖于外层查询
           * 执行时子查询需要不断引用父查询中的列值
       * Exists：
           * 子查询不返回具体数值，只返回逻辑值 
           * select 属性列表达式用*表示，因为exists不具有实际意义
           ![](https://i.imgur.com/80mFy02.jpg)
</br>
---
* **第七部分**
    * 新建视图： **create view [视图名]（属性列）[select 子查询] with check option**
    ![](https://i.imgur.com/7LDi4uv.png)
    ![](https://i.imgur.com/mlqY7Jc.png)
    * 删除视图：**drop view [视图名]**
    ![](https://i.imgur.com/PxuAVjI.png)
    * 查询视图信息： **与表一致**
    ![](https://i.imgur.com/Tk4KWkz.png)
    * 更新数据与表一致。</front>
   
    


    
    


