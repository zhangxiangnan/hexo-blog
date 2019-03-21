### 基础规范
- 绝大部分场景存储引擎必须使用InnoDB
    支持事务
- 字符集编码必须使用utf8mb4字符集(支持emoji表情以及部分不常见汉字)
  utf8mb4对应字符的比较排序常用排序字符集utf8mb4_unicode_ci、utf8mb4_general_ci。utf8mb4_unicode_ci能在各种语言精准排序；utf8mb4_general_ci性能好一点，但对于特殊字符排序不是非常准确，两者都可。建议utf8mb4_unicode_ci。
- 数据表、字段必须加注释
- 不要使用触发器、存储过程、视图、外键约束
    考虑业务层处理，可扩展性强（数据库一般一主多从，主节点没法横向扩展），便于调试
- 不要存文件、图片等大数据及TEXT、BLOB类型
    使用文件服务器、图片服务器等文件系统存储，数据库里可以存储url

### 命名规范
- dev、test环境可以使用ip直连数据库，stag&prod需使用内网域名连接，其他类似尽可能线上使用域名连接，有利于依赖方对自身服务机器的运维
- 数据库名称尽量合项目或业务名称一样，各个环境的库名最好一样
- 库名、表名、字段名要用小写，下划线风格，<=32个字符，名称和业务绑定要有意义，不得拼音英文混合。
- 索引名，唯一索引名为uk_字段名，普通索引idx_字段名

### 表设计规范
- 表必须要有主键id
  - 表碎片率低，顺序insert效率高；
  - 推荐unsigned整数（BIGINT），值自增或者业务设置
  - 不要用varchar、char、uuid作为主键，变为随机写入，效率低；
  - 无主键表说是删除表时，对于row模式主从架构，从库会挂住。
- 禁止使用外键，影响性能，可能会死锁，造成表与表耦合
  
### 字段设计规范
- 必须把字段定义为NOT NULL并提供默认值
  - NULL字段很难查询优化，表及索引都需要额外空间标示，而且会使复合索引无效，mysql针对其特殊处理导致性能下降
  - 比较只能IS NULL，IS NOT NULL，不能用＝、in、<>、!=、not in，如name != 'xx'结果不会包含name为null的记录
  - 使列索引、值比较、统计时都复杂，难优化
- 禁止使用text、blob类型，影响性能
- 货币、钱不能使用浮点数存储，用整数单位为分，或者用bigdecimal，注意精度。
- 手机号存储使用varchar(20)
  - 支持模糊查询，查询188开头的
  - 手机号可能会有国家号、区号
- 禁止用enum类型，用tinyint代替
  - 增加新enum类型，要执行ddl语句
### 参考
- https://www.kancloud.cn/wubx/mysql-sql-standard/600511
- https://www.verynull.com/2017/02/18/MySQL%E4%BA%92%E8%81%94%E7%BD%91%E4%B8%9A%E5%8A%A1%E6%95%B0%E6%8D%AE%E5%BA%93%E8%AE%BE%E8%AE%A1%E8%A7%84%E8%8C%83/
- https://www.jianshu.com/p/7c4f1efbc059
- https://www.kancloud.cn/wubx/mysql-sql-standard/600517