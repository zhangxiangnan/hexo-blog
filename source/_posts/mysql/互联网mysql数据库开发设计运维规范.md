layout: title
title: 互联网mysql数据库开发设计运维规范
date: 2019-03-23 23:07:10
tags:
- mysql
categories:
- mysql
---

### 基础规范
- 绝大部分场景存储引擎必须使用InnoDB
    支持事务
- 字符集编码必须使用utf8mb4字符集(支持4字节emoji表情以及部分不常见汉字，utf8只支持3字节)，是utf8的超集
  utf8mb4对应字符的比较排序常用排序字符集utf8mb4_unicode_ci、utf8mb4_general_ci。utf8mb4_unicode_ci能在各种语言精准排序；utf8mb4_general_ci性能好一点，但对于特殊字符排序不是非常准确，两者都可。建议utf8mb4_unicode_ci。
- 数据表、字段必须加注释
- 不要使用触发器、存储过程、视图、外键约束、events、udf
    - 能放业务层处理的逻辑都放业务层处理；
    - 业务层可扩展性强（数据库一般一主多从，主节点没法横向扩展），且便于调试，查问题
    - 为了高并发
- 不要存文件、图片等大数据及TEXT、BLOB类型
    - 使用文件服务器、图片服务器等文件系统存储，数据库里可以存储url
    - 非用不可则拆到单独表，使用主键做关联 
- 线下环境服务禁止直连线上数据库，各环境数据库隔离
- 禁止针对线上数据库进行压力测试，影响性能及正常业务，可能导致故障
- 库备份可以以类似backup为前缀＋数据库名＋日期为后缀，从库以-s为后缀，备库以-ss为后缀，只要能标示即可。
  
### 命名规范
- dev、test环境可以使用ip直连数据库，stag&prod需使用内网域名连接，其他类似尽可能线上使用域名连接，有利于依赖方对自身服务机器的运维
- 数据库名称尽量合项目或业务名称一样，各个环境的库名最好一样
- 库名、表名、字段名要用小写，下划线风格，<=32个字符，名称和业务绑定要有意义，不得拼音英文混合。
- 索引名，唯一索引名为uk_字段名，普通索引idx_字段名，小写
- 表名单数
  
### 表设计规范
- 表必须要有主键id
  - 表碎片率低，顺序insert效率高；
  - 推荐unsigned整数（BIGINT），值自增或者业务设置
  - 不要用varchar、char、uuid、md5、hash作为主键，变为随机写入，效率低；
  - 无主键表说是删除表时，对于row模式主从复制架构，从库的数据是以主键为依据进行更新，导致从库会挂住
- 禁止使用外键，影响性能，可能会死锁，造成表与表耦合
- 大字段，冷门字段可以拆分到单独表，分离冷热数据

### 字段设计规范
- 必须把字段定义为NOT NULL并提供默认值
  - 表及索引都需要额外空间标示，而且会使索引无效
  - mysql针对其特殊处理导致性能下降
  - 比较只能IS NULL，IS NOT NULL，不能用＝、in、<>、!=、not in，如name != 'xx'结果不会包含name为null的记录
  - 使列索引、值比较、统计时都复杂，难优化
- 禁止使用text、blob类型，影响性能
- 货币、钱不能使用浮点数存储会损失精度，用整数单位为分，或者用bigdecimal，注意精度。
- 手机号存储使用varchar(20)，不要用整数
  - 支持模糊查询，查询188开头的
  - 手机号可能会有国家号、区号
- 禁止用enum类型，用tinyint代替
  - 增加新enum类型，要执行ddl语句
- 根据业务区分使用tinyint/int/bigint，占用1/4/8字节，合理的类型减少空间浪费
- 根据业务区分使用char/varchar
  - 长度固定，或近似使用char，减少内存碎片，提升查询性能
  - 字段长度相差大，或更新较少，用varchar，节省磁盘及内存空间；字符长度小于5000使用varchar，否则text类型，拆单独表存储，避免影响效率
- 根据业务区分使用datetime/timestamp
  - 存储年用YEAR，日期用DATE，时间datetime
- 使用INT UNSIGNED存储IPv4，不要用char(15)，减少存储空间，查询时使用INET_ATON()、INET_NTOA()进行转换
- 每个表都建议添加id、create_time（创建时间，datetime类型）、update_time（更新时间，datetime类型）、creator（创建者）、modifier（修改者）
- 严禁明文存储密码、身份证、信用卡号、银行卡号等机密数据，要加密，存储密文，进行数据脱敏
- 整型字段，若为非负，必须UNSIGNED类型，扩大数值范围

### 索引设计规范
- 单表的索引建议<=5，包括唯一id索引
  - 太多索引导致索引占用太多存储空间，且插入更新操作会更新相关索引，影响性能
  - 太多索引可能导致mysql选择不出最优索引
  - 复杂条件查询建议使用es等存储引擎查询
- 单个索引的字段个数不能超过5个
  - 太多字段导致存储空间增大，影响性能；
  - 另一方面说明选取字段区分度不高，值域较少，不能有效过滤数据
- 区分度不高、更新频繁的字段禁止建立索引
  - 区分度不高的字段不能有效进行数据过滤，如性别字段
  - 更新频繁的字段每次会更新索引，更新b＋树，影响性能
- 避免使用join查询
  - 尽量放到业务层处理，多次表查询
  - 必须join的话，需join的字段类型必须相同，且建立索引，否则全表扫描;最多不超过3个表join
- 组合索引最左前缀原则，不要重复建立索引和已有单列索引重复，复合索引如(a,b,c)，相当于建立(a)，（a,b）,(a,b,c)三个索引
- 对字符串使用前缀索引时，前缀索引长度不超过8个字符，太多字符导致索引存储空间大，影响性能；有时可以通过增加冗余列来减少索引长度，提升效率
- 复合索引把区分度高，去重后个数多的字段放索引前面
- 使用explain进行sql性能分析，是否使用索引，避免extra出现：use file sort，use tempory，导致性能降低
- 具有唯一特性的一个或多个字段建立唯一索引，避免脏数据
- sql explain时语句type至少达到range级别（基于索引范围搜索），ref好些(使用索引)，constants(匹配单条记录)最好。
  
### SQL使用规范
- sql要尽可能简单，拆分
- 事务要简单，禁止包含非数据库操作逻辑
- 禁止使用SELECT ＊，只获取必要的字段
  - 读取多余列增加CPU、IO、NET消耗，影响性能、带宽、IO
  - 不能有效利用覆盖索引(假如查询的列都在索引里，直接从索引获取数据，而不必再次读取数据行)
  - SELECT * 容易在增加删除字段等表结构变更后后出现问题
  
- 禁止使用INSERT INTO t_xxx VALUES(xxx)，必须显示指定插入的列属性，容易在增加或者删除字段等表结构变更后出现程序BUG，字段顺序错乱导致的值错乱&sql报错等
- 禁止使用属性隐式转换
    如seelct uid, name from user where phone_num = 18810231321语句不会使用索引（phone_num使用varchar存储且该列建了索引）,导致全表扫描
- 禁止where条件的属性上使用函数或者表达式，导致索引不生效，全局扫描;在值上可以使用
    如select uid from user where age - 2 > 10可改为 age > 12;
- 反向、负向查询以及％开头的模糊查询
  - 负向查询如NOT、!=、<>、!<、!>、NOT IN、NOT LIKE等会导致全局扫描；但如果where uid = xx and status != 1类似先执行了uid过滤后进行负向查询的话还可以
  - 针对字符，左前缀模糊查询导致索引不生效，全表扫描；其他类型后面模糊也不走索引
- 禁止大表使用join查询，及使用子查询
    会产生临时表，消耗内存、cpu，影响性能；避免临时表
- union all不去重，少了排序操作；union去重，性能不如union all
- 禁止使用同一字段上的OR查询，改为IN,in也要尽量避免使用，in里个数控制在500或1000个内，OR使索引失效；不同字段的or用union all代替
- 应用程序要捕获sql异常，进行处理，便于定位问题
- 使用预编译prepared sql，防止注入，提升性能
- 重要sql要被索引
  - where条件里的字段
  - ORDER BY、GROUP BY、DISTINCT的字段
- sql语句建议全部小写，或者全部大写，小写容易读
- 避免使用count(*)，会全表扫描，计数实时较强的场景可以用redis等分布式缓存，不强可以用单独的统计表定时更新
- 禁止update语句时set a = x ,b = y;写成a = x and b = y，意义不一样
- 尽量减少与数据库交互次数，能批量则批量，能一次不2次
- insert使用batch提交时insert into table values (),()..，小于500个，根据数据量酌情定
- sql语句不用不确定值函数及随机函数，如rand()、now()
- 减少、避免排序，group by如果不需要排序可以加order by null
- 多使用limit n，少用limit m,n，m比较大时影响性能
- 使用INSERT ... ON DUPLICATE KEY update (INSERT IGNORE)或者replace语句来避免不必要的查询
- 尽量使用主键update、delete操作
- 用where子句代替having子句，having可能不同执行计划结果不同
- 不要一次大量删除、更新或导入导出数据，分批，太多容易出问题
- 多表连接查询数据量小的表的条件紧跟where语句后
- count(列名)不会统计列为null的行，所以列要not null
- count(distinct col1)统计col1不为null的不重复数量，列要not null
- count(distinct col1, col2)，col1或col2为null，则返回0，所以列要not null
- count(col)若col列全为null返回0，但是sum(col)则返回null，npe异常，列要not null
- sql delte update语句执行时要先select查看影响条数，及diff语句前后的影响数据

### 流程规范
- 建表时提前考虑可能的查询场景，并建立对应合适的索引
- 表结构更改、加索引需要提前周知dba，梳理表的查询sql，评估风险
- 大量批量导入、导出数据及统计类查询，不得使用主库，线上在线的从库
- 活动推广，新功能要评估数据库压力
- 业务高峰期不做表结构更改，加索引等有风险的操作

### 开发规范技巧
- 注意整数字段的溢出，业务做参数校验
- 注意字符串自动转换为整数的溢出，校验参数
- 分页技巧
  limit m, n,m越大，性能越慢，需要先扫描前m条记录，然后往后找n条
  - id方式，where id > xx limit 10;
  - 子查询优化
    ```
    Select * from table WHERE id >= ( select id from table limit 10000,1 ) limit 10;
    ```
  - join优化
      ```
      SELECT * FROM table INNER JOIN (SELECT id FROM table LIMIT 10000,10) USING (id) ;
      ```
  - 先取id，再次执行in (id1,id2)查询
  - 缓存前若干页热门数据

### mysql特点
- mysql的查询缓存适用于更新频率非常低，查询频率非常高的场景，否则建议关闭查询缓存
- 简单使用mysql，不要使用外键、分区表、存储过程等特性

### 分库分表
- 通常根据表的某个字段如订单号、userid来进行hash或者range拆分
- 单表数据量建议超过千万级别或单表容量>2GB才推荐分库分表，不过有的表数据量亿级容量也可以轻松应对，看情况

### 读写分离
- 主从复制分异步复制，半同步复制
- 建议交易环节所有请求都走主库

### 对象关系映射
- domain类boolean属性字段不要以is为前缀，有坑;数据库boolean字段加is前缀
- domain类不能直接返回到展示层，需进行到VO或DTO的转换
- mybatis的mapper里不能使用${}，要用#{}，防止sql注入，提升性能，预编译
- mybatis的queryForList(String statementName,int start,int size)不要使用，底层查询所有数据，然后内存截取
  
### 总结
- 互联网业务一般高并发，大数据量，然而数据库连接有效，查询&写入必须快速，否则就影响性能，长时间占有连接则影响并发。提升性能则查询尽可能走索引，不要导致全局扫描、文件排序。
- 针对数据的统计分析，或者数据的多维度搜索、聚合，建议使用ETL或者搜索引擎

### mysql各数据类型范围
| 类型      |      占用字节      |  有符号范围        | 无符号范围 |
|-         |:-:                |-:                 |-          |
| TINYINT  |  1                | (-128，127)       |	(0，255) |
| SMALLINT |  2                | (-3万，3万)     | (0，6万) |
| MEDIUMINT| 3                 | (-800万，800万) | 	(0，约1600万) |
| INT或INTEGER	| 4            | 约21亿| 约42亿 |
| BIGINT| 8                    |    (-922亿亿, 922亿亿)     | (0,1844亿亿) |
| DECIMAL| 对DECIMAL(M,D) ，如果M>D，为M+2否则为D+2|    依赖于M和D的值     | 依赖于M和D的值 |

### 参考
- https://www.kancloud.cn/wubx/mysql-sql-standard/600511
- https://www.verynull.com/2017/02/18/MySQL%E4%BA%92%E8%81%94%E7%BD%91%E4%B8%9A%E5%8A%A1%E6%95%B0%E6%8D%AE%E5%BA%93%E8%AE%BE%E8%AE%A1%E8%A7%84%E8%8C%83/
- https://www.jianshu.com/p/7c4f1efbc059
- https://www.kancloud.cn/wubx/mysql-sql-standard/600517
- https://my.oschina.net/leejun2005/blog/476839
