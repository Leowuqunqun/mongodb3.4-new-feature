# mongoDb 3.4 New Features

<!-- toc -->
- [官方视频教程](#%E5%AE%98%E6%96%B9%E8%A7%86%E9%A2%91%E6%95%99%E7%A8%8B)
  * [官方视频教程](#%E5%AE%98%E6%96%B9%E8%A7%86%E9%A2%91%E6%95%99%E7%A8%8B-1)
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
      - [总结](#%E6%80%BB%E7%BB%93)
    + [Example 2 查询项目同时查询项目对应任务列表](#example-2-%E6%9F%A5%E8%AF%A2%E9%A1%B9%E7%9B%AE%E5%90%8C%E6%97%B6%E6%9F%A5%E8%AF%A2%E9%A1%B9%E7%9B%AE%E5%AF%B9%E5%BA%94%E4%BB%BB%E5%8A%A1%E5%88%97%E8%A1%A8)
      - [总结](#%E6%80%BB%E7%BB%93-1)
- [Decimal Support](#decimal-support)
- [Robust Initial Sync](#robust-initial-sync)
- [Collations](#collations)
- [Ops Manager](#ops-manager)
- [Zone Sharding](#zone-sharding)
- [Compass](#compass)
- [New Aggregation Operators](#new-aggregation-operators)
- [Facets](#facets)
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

```bash
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

##### 总结

* 性能差,不适合树形查询

#### Example 2 查询项目同时查询项目对应任务列表

```bash
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
    ])
```

##### 总结

* 性能下降, 一对多简单关联的时候不适合使用




## Decimal Support

## Robust Initial Sync

## Collations

## Ops Manager

## Zone Sharding

## Compass

## New Aggregation Operators

## Facets

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


























