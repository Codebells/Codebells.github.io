---
title: nebula
date: 2022-10-23 15:11:21
categories: 
  - [database]
  - [graphdb]
tags: [database,graphdb]
category_bar: true
---

# 查询引擎

- conf/：查询引擎配置文件目录
- package/：graph 打包脚本
- resources/：资源文件
- scripts/：启动脚本
- src/：查询引擎源码目录
  - src/context/：查询的上下文信息，包括 AST（抽象语法树），Execution Plan（执行计划），执行结果以及其他计算相关的资源。
  - src/daemons/：查询引擎主进程
  - src/executor/：执行器，各个算子的实现
  - src/optimizer/：RBO（基于规则的优化）实现，以及优化规则
  - src/parser/：词法解析，语法解析，：AST结构定义
  - src/planner/：算子，以及执行计划生成
  - src/scheduler/：执行计划的调度器
  - src/service/：查询引擎服务层，提供鉴权，执行 Query 的接口
  - src/session/：Session 管理
  - src/stats/：执行统计，比如 P99、慢查询统计等
  - src/util/：工具函数
  - src/validator/：语义分析实现，用于检查语义错误，并进行一些简单的改写优化
  - src/visitor/：表达式访问器，用于提取表达式信息，或者优化
- tests/：基于 BDD 的集成测试框架，测试所有 NebulaGraph 提供的功能

# 存储

- conf/：存储引擎配置文件目录
- package/：storage 打包脚本
- scripts/：启动脚本
- src/：存储引擎源码目录
  - src/codec/：序列化反序列化工具
  - src/daemons/：存储引擎和元数据引擎主进程
  - src/kvstore/：基于 raft 的分布式 KV 存储实现
  - src/meta/：基于 KVStore 的元数据管理服务实现，用于管理元数据信息，集群管理，长耗时任务管理等
  - src/storage/：基于 KVStore 的图数据存储引擎实现
  - src/tools/：一些小工具实现
  - src/utils/：代码工具函数

# 内核工具包

- src/common/clients/：meta，storage 客户端的 CPP 实现
- src/common/datatypes/：NebulaGraph 中数据类型及计算的定义，比如 string，int，bool，float，Vertex，Edge 等。
- rc/common/expression/：nGQL 中表达式的定义
- src/common/function/：nGQL 中的函数的定义
- src/common/interface/：graph、meta、storage 服务的接口定义
