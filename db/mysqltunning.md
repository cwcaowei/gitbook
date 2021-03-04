#### SQL优化

- 小表驱动大表

通过explain查看一个涉及多表的关联查询的执行计划，发现id列可能不一样，也可能一样，ID不同的按大小顺序，ID相同的按从上到下顺序，所以要把查询的中间结果较少的表放在前面。

- 通过执行计划分析是否走了正确的索引

- 用join代替子查询

- not exist转换成left join IS NULL

- OR改成UNION

- 结果集允许重复的情况下，用UNION ALL代替UNION

#### 表结构设计

- 固定长度的用char，不要用varchar，varchar需要额外的字节存储长度

- 选用合适的int类型，比如tinyint

- 字段尽量NOT NULL

- 不使用外键、触发器、视图

- 大文件只存路径

#### 连接配置优化

- 服务端

  - 增加可用连接数

    修改max_connections的大小

  -  及时释放不活动的连接

    修改wait_timeout，默认是28800秒

- 客户端

  连接池

#### 硬件

- SSD，搭建磁盘阵列，特定的CPU

- 重启，show processlist查看线程，连接等是否正常

#### 其他

Redis缓存，ES，读写分离，分库分表，分流，限流，MQ削峰填谷，降级
