### 第一章 基础知识
* PostgreSQL数据库对象
  * database
    * 每个PostgreSQL服务可以包含多个独立的database
  * schema
    * ANSI SQL标准中database的下一层逻辑结构
    * 创建新的database时，PostgreSQL会自动创建一个public的schema
    * 如果为设置search_path变量，PostgreSQL会将创建的所有对象放入默认的public schema中
  * table
    * 属于某个schema
    * PostgreSQL的表支持继承
    * 系统会自动为表创建一种对应的自定义数据类型
  * 视图
    * 基于表的一种抽象，通过它可以实现一次性查询多张表，也可以实现通过复杂运算来构造出虚拟字段。
  * 扩展包
    * 可通过该机制将一组相关的函数、数据类型、数据类型转换器、用户自定义索引，表以及属性变量等对象打包成一个功能扩展包。
  * 函数
    * 可编写自定义函数来对数据进行新增、修改、删除和复杂计算等操作。
  * 内置编程语言
  * 运算符
  * 外部表和外部数据封装器
  * 触发器和触发函数
  * catalog
    * 系统级的schema, 用于存储系统函数和系统元数据。
  * 类型
  * 全文检索
  * 数据类型转换器
  * 序列号生成器
  * 规则
  
### 第二章 数据库管理
* 配置文件
  * postgresql.conf
    * 通用设置，内存分配，新建db默认存储位置，ip地址、日志的位置等
  * pg_hba.conf
    * 用于控制PostgreSQL服务器的访问权限
  * pg_ident.conf
    * 如果该文件存在，则系统会基于文件内容将当前登陆的操作系统用户映射为一个PostgreSQL数据库内部用户的身份来登陆。
  * SELECT name, setting from pg_settings WHERE category = 'File Locations';
* 让配置文件生效
  * context属性
    * postmaster: 需要重启
    * user: 重新加载配置文件即可
  * 重新加载配置文件
    * pg_ctl reload -D 你的数据目录
    * linux: service postgresql-9.5 reload
    * SELECT pg_reload_conf
  * 重启
    * service postgresql-9.5 restart
    * pg_ctl restart -D 你的数据目录
  * postgresql.conf
    * SELECT name, context, unit, setting, boot_val, reset_val
      FROM pg_settings
      WHERE name in ('listen_addresses', 'deadlock_timeout', 'shared_buffers', 'effective_cache_size', 'work_mem', 'maintenance_work_mem')
      ORDER BY context, name;

* 连接管理
  * 查出活动连接列表及其进程ID
    * SELECT * FROM pg_stat_activity;
  * 取消连接上的活动查询
    * SELECT pg_cancel_backend(1234);
  * 终止该连接
    * SELECT pg_terminate_backend(1234);

* 角色
  * 可登录角色
    * 拥有登陆数据库权限的角色
  * 组角色
    * 拥有成员角色的角色

* 备份与恢复
  * pg_dump
    * 支持精确到表的备份，适合每日备份
  * pg_dumpall
    * 全局备份，只支持SQL格式
  * pg_basebackup

* 禁止的行为
  * 不要删除PostgreSQL系统文件
  * 不要把操作系统管理员权限授予PostgreSQL的系统账号
  * 不要把shared_buffers缓存区设置的过大
  * 不要将PostgreSQL服务器的侦听端口设为一个已被其他程序占用的端口

* 第五章 数据类型
* 数值类型
  * serial类型
    * 序列自身是一个数据库资产
    * 多表共享同一个现存的序列号生成器
      * CREATE SEQUENCE s START 1;
      * CREATE TABLE stuff(id bigint default nextval('s') PRIMARY KEY, name text);
  * 文本类型
    * char
    * varchar
    * text
  * 字符串函数
    * 填充
      * lpad
      * rpad
    * 修整空白
      * rtrim
      * ltrim
      * trim
      * btrim
    * 提取子串
      * substring
    * 连接
      * ｜｜
  * 将字符串拆分为数组、表或者子串
    * split_part
    * unnest
  * 正则和模式匹配

* 时间类型
  * date
    * 仅存储月、日、年，没有时区、小时、分和秒的信息
  * time
    * 仅存储小时、分和秒，没有时区和日期信息
  * timestamp
    * 月、日、年、小时、分和秒，没有时区
  * timestampz
    * 月、日、年、小时、分和秒，有时区
    * 以UTC标准时间储存，当查询显示时，会根据服务器的时区进行转换

* 数组类型
  *  