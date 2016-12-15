# mongoDb 3.4 New Features

<!-- toc -->

- [教程](#%E6%95%99%E7%A8%8B)
  * [官方视频教程](#%E5%AE%98%E6%96%B9%E8%A7%86%E9%A2%91%E6%95%99%E7%A8%8B)
- [Read-only Views](#read-only-views)
  * [Commands](#commands)
  * [Example](#example)
  * [Note](#note)
- [Improved BI Connector](#improved-bi-connector)
- [LDAP Authorization](#ldap-authorization)
- [Log Redaction企业版功能](#log-redaction%E4%BC%81%E4%B8%9A%E7%89%88%E5%8A%9F%E8%83%BD)
- [Faster Balancing](#faster-balancing)
- [graphLookup](#graphlookup)
  * [Basic Commands](#basic-commands)
  * [Examples](#examples)
    + [Example 1 查询所有母任务](#example-1-%E6%9F%A5%E8%AF%A2%E6%89%80%E6%9C%89%E6%AF%8D%E4%BB%BB%E5%8A%A1)
    + [Example 2 查询所有子任务](#example-2-%E6%9F%A5%E8%AF%A2%E6%89%80%E6%9C%89%E5%AD%90%E4%BB%BB%E5%8A%A1)
    + [Example 3 查询项目同时查询项目对应任务列表](#example-3-%E6%9F%A5%E8%AF%A2%E9%A1%B9%E7%9B%AE%E5%90%8C%E6%97%B6%E6%9F%A5%E8%AF%A2%E9%A1%B9%E7%9B%AE%E5%AF%B9%E5%BA%94%E4%BB%BB%E5%8A%A1%E5%88%97%E8%A1%A8)
    + [Example 4 官方Demo](#example-4-%E5%AE%98%E6%96%B9demo)
    + [graphLookUp 注意点](#graphlookup-%E6%B3%A8%E6%84%8F%E7%82%B9)
- [Decimal Support](#decimal-support)
- [Robust Initial Sync](#robust-initial-sync)
- [Collations](#collations)
- [Ops Manager](#ops-manager)
  * [官方文档](#%E5%AE%98%E6%96%B9%E6%96%87%E6%A1%A3)
- [Zone Sharding](#zone-sharding)
- [Compass](#compass)
  * [下载链接](#%E4%B8%8B%E8%BD%BD%E9%93%BE%E6%8E%A5)
- [New Aggregation Operators](#new-aggregation-operators)
- [Facets](#facets)
  * [基本概念](#%E5%9F%BA%E6%9C%AC%E6%A6%82%E5%BF%B5)
  * [Example](#example-1)
- [Linearizable Read Concern](#linearizable-read-concern)
  * [Linearizable](#linearizable)
- [Spark Connector V2](#spark-connector-v2)
- [Upgrade and Downgrade Process](#upgrade-and-downgrade-process)
- [Installation-windows](#installation-windows)


<!-- tocstop -->

## 教程

### [官方视频教程](https://university.mongodb.com/courses/MongoDB/M034/2016_November/syllabus)

## Read-only Views

### Commands

```bash
    db.runCommand( { create: <view>, viewOn: <source>, pipeline: <pipeline> } )
   
    db.createView(<view>, <source>, <pipeline>, <collation> )
```

### Example

```javascript
    use raven

    db.runCommand({
        create: 'top_folder_pid',
        viewOn: 'folder',
        pipeline: [
            {
                $group: {
                    _id: "$ProjectID",
                    folderCount: {$sum: 1}
                }
            },
            {
                $sort: {
                    folderCount: -1
                }
            }
        ]
    })

    //find
    db.top_folder_pid.find()
    //update will throw exception
    db.top_folder_pid.update({_id: ""}, {$set: {folderCount: 1}})
    //Aggregation
    db.top_folder_pid.aggregate([
        {
            $match: { _id: "fe288386-3d26-4eab-b5d2-51eeab82a7f9" }
        }
    ])

    //drop a view
    db.top_folder_pid.drop();
```

### Note

1. 视图为只读, 在视图上写操作会出错

1. 视图使用底层集合的索引, 如基于集合folder创建的视图可以用到folder上的索引

1. 视图在执行时,会参照底层集合是否为分片,即: 若底层集合为分片, 则创建的视图也并不可以使用[$lookup](https://docs.mongodb.com/manual/reference/operator/aggregation/lookup/#pipe._S_lookup)或者 [$graphLookup](https://docs.mongodb.com/manual/reference/operator/aggregation/graphLookup/#pipe._S_graphLookup)

1. 视图在执行时才会计算, 类似sql 的视图, 所以不能执行以下命令
    * [db.collection.mapReduce()](https://docs.mongodb.com/manual/reference/method/db.collection.mapReduce/#db.collection.mapReduce)
    * [$text](https://docs.mongodb.com/manual/reference/operator/query/text/#op._S_text),因为$text 只能在aggregate第一个阶段执行
    * [geoNear](https://docs.mongodb.com/manual/reference/command/geoNear/#dbcmd.geoNear),因为$geoNear 只能在aggregate第一个阶段执行

1. 视图的[find()](https://docs.mongodb.com/manual/reference/method/db.collection.find/#db.collection.find)操作不支持以下[projection](https://docs.mongodb.com/manual/reference/operator/projection/)
    * [$](https://docs.mongodb.com/manual/reference/operator/projection/positional/#proj._S_)
    * [$elemMatch](https://docs.mongodb.com/manual/reference/operator/projection/elemMatch/#proj._S_elemMatch)
    * [$slice](https://docs.mongodb.com/manual/reference/operator/projection/slice/#proj._S_slice)
    * [$meta](https://docs.mongodb.com/manual/reference/operator/projection/meta/#proj._S_meta)

1. 视图可以重命名

1. 视图的字符串比较只能用默认的排序,如: 如果视图创建时指定的是英文,则不能按其他语言做字符串比较

1. If the aggregation pipeline used to create the view suppresses the _id field, documents in the view do not have the _id field.


## Improved BI Connector
> [视频教程](https://university.mongodb.com/courses/MongoDB/M034/2016_November/courseware/Chapter_2_Improved_BI_Connector/57ec1d6816f46429ae8ef37d)

## LDAP Authorization
> 企业版功能 [LDAP Authorization](https://docs.mongodb.com/manual/release-notes/3.4/#ldap-enhancements)

## Log Redaction企业版功能
> [Log Redaction企业版功能](https://docs.mongodb.com/manual/release-notes/3.4/#log-redaction)

## Faster Balancing
> [Faster Balancing](https://docs.mongodb.com/manual/release-notes/3.4/#faster-balancing)

## graphLookup

### Basic Commands

```javascript
    {
        $graphLookup: {
            from: <collection>,
            startWith: <expression>,
            connectFromField: <string>,
            connectToField: <string>,
            as: <string>,
            maxDepth: <number>,
            depthField: <string>,
            restrictSearchWithMatch: <document>
        }
    }
```

### Examples

#### Example 1 查询所有母任务

```javascript
    db.task.aggregate([
        {
            $match: {
                TaskID: "881e79ae-edbe-4131-8642-8d688709f163"
            }
        },
        {
            $graphLookup: {
                from: "task",
                startWith: "$ParentID",
                connectFromField: "ParentID",
                connectToField: "TaskID",
                as: "Parents"
            }
        }
    ]);
```

#### Example 2 查询所有子任务

```javascript
    db.task.aggregate([
        {
            $match: {
                TaskID: "31ea3d22-7353-4ac0-9520-47779627d631"
            }
        },
        {
            $graphLookup: {
                from: "task",
                startWith: "$TaskID",
                connectFromField: "TaskID",
                connectToField: "ParentID",
                as: "SubTasks"
            }
        }
    ]);

```


#### Example 3 查询项目同时查询项目对应任务列表

```javascript
    //graphLookup
    db.folder.aggregate([
        {
            $match: {
                FolderID: "5c027e7d-e98a-487b-814c-2cf633f4d223"
            }
        },
        {
            $graphLookup: {
                from: "task",
                startWith: "$FolderID",
                connectFromField: "FolderID",
                connectToField: "FolderID",
                maxDepth: 2,
                depthField: "depth",
                as: "Tasks"
            }
        }
    ]);

    //lookup
    db.folder.aggregate([
        {
            $match: {
                FolderID: "96367dcc-6c41-4eb3-b391-905c907e9840"
            }
        },
        {
            $lookup: {
                from:"task",
                localField:"FolderID",
                foreignField:"FolderID",
                as:"Tasks"
            }
        }
    ]);
```

#### Example 4 官方Demo

1. [Examples](https://docs.mongodb.com/manual/reference/operator/aggregation/graphLookup/#examples)

#### graphLookUp 注意点

1. 在*from*里指定的collection不可以是sharded;

1. 设置maxDepth 参数为0, 等效于 [$lookup](https://docs.mongodb.com/manual/reference/operator/aggregation/lookup/#pipe._S_lookup) 查询(不递归)

1. 在使用 $graphlookup 时需要注意到 [aggregration pipeline的限制](https://docs.mongodb.com/manual/core/aggregation-pipeline-limits/) , 包括 [内存限制](https://docs.mongodb.com/manual/core/aggregation-pipeline-limits/#agg-memory-restrictions)

1. $graphlookup 不能像其他聚合操作一样把硬盘空间当做内存来使用, 所以你必须接受100M 内存限制

## Decimal Support

```javascript
    //shell Example
    db.inventory.insert( {_id: 1, item: "The Scream", price: NumberDecimal("9.99"), quantity: 4 } )
```

> Unlike the double data type, which only stores an approximation of the decimal values, the decimal data type stores the exact value. For example, a decimal NumberDecimal("9.99") has a precise value of 9.99 where as a double 9.99 would have an approximate value of 9.9900000000000002131628....

> 翻译: 数值:0.99  double类型真实存储的值近似9.9900000000000002131628...., Decimal类型存储0.99


## Robust Initial Sync

* 优化了副本集初始同步的性能

* 初始化同步重试逻辑更佳可靠

* 可以用 *rs.status()* 查看初始同步状态和进度

## Collations



## Ops Manager

### [官方文档](https://docs.opsmanager.mongodb.com/current/application/#mms-overview)

## Zone Sharding

## Compass

### [下载链接](https://www.mongodb.com/downloads?jmp=docs&_ga=1.111399943.2031717362.1470108291#compass)

## New Aggregation Operators

## Facets

### 基本概念

> 在一个阶段中以相同的输入并行执行多个聚合操作
> 在$facet操作中不能使用以下操作符: *$facet*,*$out*,*$geoNear*,*$indexStats*,*$collStats*

### Example

```javascript
    use taskcenter
    db.task.aggregate([
        {
            $match: { FolderID: "68878ce2-b5a7-4989-8db1-6f1e9025e37a" }
        },
        {
            $facet: {
                "MembersJoinTaskCount": [
                    { $unwind: "$Members" },
                    { $match:{"Members.Type":0}},
                    { $sortByCount: "$Members.AccountID" }
                ],
                "ChargeUserTaskCount":[
                    {$sortByCount:"$ChargeAccountID"}
                ],
                "Deadlines":[
                    {$group:{_id:"$Deadline",count:{$sum:1}}}
                ],
                "DeadlineTasksBucketByDate":[
                    {$bucket:{
                        groupBy:"$Deadline",
                        boundaries:[new Date('2015-1-1'),new Date('2015-6-1'),new Date('2016-6-1')],
                        default:"Other",
                        output:{
                            "count":{$sum:1},
                            "charges":{$push:"$ChargeAccountID"}
                        }
                    }}
                ],
                "DeadlineTasksBucketByDate(Auto)":[
                    {
                        $bucketAuto:{
                            groupBy:"$Deadline",
                            buckets:10
                        }
                    }
                ]
            }
        }
    ]);
```


## Linearizable Read Concern

### [Linearizable](https://docs.mongodb.com/manual/reference/read-concern/#readconcern.)


## Spark Connector V2

## Upgrade and Downgrade Process

## Installation-windows

```bash
    mongod
        --dbpath=D:\mongodb3.4
        --logpath=D:\mongodb3.4\log.txt
        --port 27018
        --install
        --serviceName MongoDB3.4
        --serviceDisplayName mongoDB3.4

```


























