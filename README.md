# liquidbase-demo



## 内容大纲

1. 为什么要使用liquibase
2. liquibase一些基本概念介绍
3. 实践和使用liquibase
4. liquibase来实现回滚



比如大家都喜欢玩某个版本的游戏，结果技术告诉你两个消息，一个好消息，一个坏消息
好消息是：代码和模型都还在；
坏消息是：数据库结构无法恢复到那个版本；

现在项目中是否会遇到这样的问题

- 在DEV环境测试了一波代码没有问题，然后转化到TEST环境，这个时候代码无缝迁移过去了，但是数据库中的表结构，或者一些字典值总是忘记添加，然后你又的取找对应的sql。而且由于是多个人同时开发的，总会难免遗漏一些sql。然后你又发到RELASE环境，同样的问题又发生了。
- 代码可以通过打TAG来快速的切换部署到任何一个节点，但是如果数据库不能做到，结果都是一样的：程序无法正常运行。

## Liquibase的一些特点

和手工相比


|        手工部署         |    Liquibase部署     |
| :---------------------: | :------------------: |
|    容易出错和有延迟     |  减少错误、实时部署  |
|     对回滚无能为力      |  可以方便的实现回滚  |
| 需要在部署之前先运行sql |  可以和应用同步发布  |
|     事务性难以保证      |      简单且容易      |
|  对迁移数据库无能为力   | 无需任何改动即可实现 |

使用liquibase和以前手动编写sql逻辑完全变了。以前自己写sql来生成表、添加数据等等。现在改成，统一来编写liquibase来编写SQL，然后让liquibase来生成数据库结构和内容；

### 轻松的和CI/CD进行整合

1.将变更集文件推送到要素存储库
 2.针对Dev分支创建拉取请求
 3.在同行评审和批准之后，将功能分支与开发分支合并
 4.在Dev服务器上配置的CI / CD实现触发Liquibase进行数据库更新
 5.Liquibase会自动执行任何新的变更日志文件（而且足以记住已经运行了哪些脚本）



### 一些基本的概念

- databaseChangeLog
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

**MD5SUM是根据id+author+filename中的SQL语句形成的一个MD5值。如果已经运行过的sql，你修改了sql内容，这个时候就会报错**。

此时只是以运行sql的内容为准，多加空格之类的并不能影响MD5SUM的值。

一个changeset里面的sql可以放多个，这样就可以运行了。

<sq></sql>标签和使用xml来定义sql语句相比，使用<sql>标签更加的通俗易懂，但是xml更加的结构化，并且能够实现复杂的功能，比如可以当满足某个条件的时候执行一个sql。

举几个例子来说：

> 1. 可以在java代码中实现一个类来做数据迁移，而sql可能不能满足，而xml可以满足。
> 2. 如果是复杂的sql的话，可以使用<sqlFile>来引用sql，使sql和xml做分离。

### DATABASECHANGELOGLOCK

顾名思义这个表是运行Liquibase的一个lock表，如果LOCKED=1,则说明有人在执行，当多个人同时执行的时候会报错的。

当LOCKED这个字段的值设置为1的时候，那么别的应用就会等待。直到其释放锁。

## 命令进行回滚

回滚到具体某个tag

```
liquibase rollback --tag=v1.0
```

回滚一次

```
liquibase rollbackCount 1
```

回滚到某个日期

```
liquibase rollback-to-date --date=2020-05-07
```

重新设置数据库，让数据到某个状态

### update-to-tage

> 这个命令可以把数据恢复到某个tag的时候。
>
> 需要注意的是，以下这个命令在win10的idea中执行是没有效果的，会报错:
>
> > liquibase : 无法将“liquibase”项识别为 cmdlet、函数、脚本文件或可运行程序的名称。请检查名称的拼写，如果包括路径，请确保路径正确.
>
> 但是打开cmd命令执行是没有问题的。
>
> 执行的时候需要你配置文件liquibase.properties的位置执行

```
liquibase update-to-tag --tag=version_test_rollback  --changelog-file=changelog-master.xml
```

### Mvn中使用命令

像这类的高级操作，推荐使用命令行来进行操作。命令行的操作比maven插件的更丰富，功能更强大（而且有些是付费才有的）。

**注意事项，这里必须使用cmd的命令来进行，如果使用idea自带的terminal是运行不了的**

```
mvn liquibase:rollback -Dliquibase.rollbackTag=v1.0
```

```
mvn liquibase:rollback -Dliquibase.rollbackCount=1
```

帮助命令

```linux
mvn liquibase:help

mvn liquibase mvn liquibase:changelogSyncToTag --help
```

## 使用liquibase的一些注意事项

1. SQL迭代的基础上禁止在原有的changeSet上进行修改sql。因为liquibase是以id+author+filename作为唯一值的。
2. 生产环境建议使用命令行来执行liquibase的语句。因为语句更灵活，而且命令更方便。
3. 虽然<sql>标签使用很方面，但是要杜绝这种情况出现，因为这导致数据库没有迁移能力减弱。
4. 从理论上来讲，生产环境是只允许DBA来进行操作liquibase的。

### 参考文章

- https://www.pkslow.com/archives/liquibase
- https://juejin.cn/post/7081583157371207716
- https://www.cnblogs.com/gao241/archive/2013/03/20/2971526.html
- https://springboot.io/t/topic/2952  回滚
- https://docs.liquibase.com/change-types/create-table.html 创建表的一些信息







```

```

