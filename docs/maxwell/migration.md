## oracle 数据库迁移mysql



### 分析oracle mysql 的不同点

​    <img src="/docs/img/image2022-12-29_10-26-57.png" alt="oracle官方对照表" style="zoom:50%;" />

​            <img src="/docs/img/image2022-12-29_10-27-10.png" alt="oracle官方对照表" style="zoom:67%;" />

<img src="/docs/img/image2022-12-29_10-27-10.png" alt="oracle官方对照表" style="zoom:75%;" />

 主键：

​            oracle 现在项目中使用的是 select SEQ_TEST.NEXTVAL from DUAL 获取主键id

​          mysql 使用 AUTO_INCREMENT 方式创建主键id， 主键id的起始id要大于存量数据的id

改写工具： https://www.sqlines.com/online



### 调研数据库同步工具

 oracle 数据同步的几种方式:

- 基于时间戳定时dump
- oracle 物化视图(materialized view)
- oracle trigger机制，比如DataBus , SymmetricDS
- oracle CDC(Change Data Capture)
- oracle DCN(Data Change Notification) -- 增量数据需要一个rowid字段
- oracle日志文件，比如LogMiner，OGG

同步工具对比

| 工具名   | 作者     | 全量同步 | 增量同步 | 自动创建表结构 | 类型转换 |
| -------- | -------- | -------- | -------- | -------------- | -------- |
| dataX    | alibaba  | √        |          |                |          |
| yugong   | alibaba  | √        | √        |                |          |
| dbsyncer | 个人开源 | √        | √        |                |          |



### 制定迁移方案

​       使用 yugong ALL模式 提前一个月同步数据





