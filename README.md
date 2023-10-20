# liquidbase-demo

参考文章

- https://www.pkslow.com/archives/liquibase
- https://juejin.cn/post/7081583157371207716

比如大家都喜欢忘某个版本的游戏，结果技术告诉你两个消息，一个好消息，一个坏消息
好消息是：代码和模型都还在；
坏消息是：当时的那一版数据库已经找不到或者恢复不了了



## Liquibase的一些特点

### 轻松的和CI/CD进行整合

1.将变更集文件推送到要素存储库
 2.针对Dev分支创建拉取请求
 3.在同行评审和批准之后，将功能分支与开发分支合并
 4.在Dev服务器上配置的CI / CD实现触发Liquibase进行数据库更新
 5.Liquibase会自动执行任何新的变更日志文件（而且足以记住已经运行了哪些脚本）





### 一些基本的概念

- changeset



## 关于一些配置文件

liquibase是有一个主文件的，这个主文件内可以执行你想要的sql指令。

如果是全部执行的则可以使用如下的配置

```xml
<includeAll path="change" relativeToChangelogFile="true"  />
```

如果你是想执行特定的一些文件可以使用以下语句

```sql
<include file="change/v1.20220702.xml" relativeToChangelogFile="true"/>
```

### DATABASECHANGELOG表的解释

这个表是一个元数据表，表中保存了一些liquibase的运行记录。其中

```
id+author+fileanme形成来作为唯一标识。
```

MD5SUM是根据id+author+filename中的语句形成的一个MD5值。如果已经运行过的sql，你修改了sql内容，这个时候就会报错。

一个changeset里面的sql可以放多个，这样就可以运行了。

<sq></sql>标签和使用xml来定义sql语句相比，使用<sql>标签更加的通俗易懂，但是xml更加的结构化，并且能够实现复杂的功能，比如可以当满足某个条件的时候执行一个sql。





## 疑问

1. liquibase的执行原理是什么？ID能否重复。同样的配置，进行多次执行，会对数据库有影响吗（这个需要使用sql语句来验证）？ 如果改掉id或者作者的任意一个呢？