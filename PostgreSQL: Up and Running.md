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
  * SELECT ARRAY [2001,2002,2003] AS yrs;
  * 将一个字符串格式书写的数组转变为真正的数组
    * SELECT '{Alex, Sonia}'::text[] As name, '{46, 43}':: smallint[] As age;
  * 将一个分隔符格式的字符串转换为数组
    * SELECT string_to_array('ca.ma,tx', '.') AS estados;
  * array_agg是一个聚合函数，可用于将一组任何类型的数据转换为数组
    * SELECT array_agg(f.t)
      FROM (VALUES ('{Alex,Sonia}'::text[]), ('{46,43}'::text[]))
      AS f(t);
  * 将数组元素展开为记录行
    * SELECT unnest('{XOX,OXO,XOX}'::char(3)[]) AS tic_tac_toe;

  * 区间类型
    * int4range, int8range
      * 整数型离散区间
    * numrange
      * 连续区间，包括decimal, float, double
    * daterange
      * 不带时区信息的日期离散区间
    * tsrange, tstzrange
      * 时间戳类型的连续区间
  * 定义包含区间类型字段的表
    * CREATE TABLE employment
      (id SERIAL PRIMARY KEY,
      empolyee varchar(20),
      period daterange);
      CREATE INDEX ix_employment_period
      ON employment USING gist (period);
      INSERT INTO employment (employee, period)
      VALUES ('Alex', '[2012-04-24, infinity)'::daterange),
            ('Sonia', '[2011-04-24, 2012-06-01]'::daterange),
            ('Leo', '[2012-06-20, 2013-04-20]'::daterange),
            ('Regina', '[2012-06-20, 2013-06-20]'::daterange);
    * 查询谁与谁同时在公司工作过
      * SELECT e1.employee,
        string_agg(DISTINCT e2.employee, ', ' ORDER BY e2.employee)
        AS colleagues
        FROM employment e1
        JOIN employment e2
        ON e1.period && e2.period
        WHERE e1.employee != e2.employee
        GROUP BY e1.employee;
    * 查询当前在职
      * SELECT employee FROM employment
        WHERE period @> CURRENT_DATE
        group by employee
      * @>运算符
        * 第一个参数是区间，第二个参数是待判定的值
        * 如果第二个参数的值落在第一个参数的区间内，运算符就返回true, 否则返回false
* JSON数据类型
  * 插入JSON数据
    * 只需建一个json类型的字段
      * CREATE TABLE persons
        (id SERIAL PRIMARY KEY,
        person json);
  * INSERT INTO persons (person)
    VALUES ('{
      "name": "Sonia",
      "spouse": {
          "name": "Alex",
          "parents": {
              "father":"Rafael",
              "mother":"Ofelia"
          },
          "phones": [
            {
              "type":"work",
              "number":"619-722-6719"
            },
            {
              "type": "cell",
              "number": "619-852-5083"
            }
          ]
        },
        "children":[
          {
            "name":"Brandon",
            "gender":"M"
          },
          {
            "name":"Azaleah",
            "girl":true,
            "phones":[]
          }
        ]
      }'
    );
  * 查询JSON数据
    * SELECT person->'name' FROM persons;
      SELECT person->'spouse'->'parents'->'father' FROM persons;
      SELECT person#>array['spouse','parents','father'] FROM persons;
      SELECT person->'children'->0->'name' FROM persons;
  * 将结果转换为string
    * SELECT person->'spouse'->'parents'->>'father' FROM persons;
  * 用json_array_elements展开JSON数组
    * SELECT json_array_elements(person->'children')->>'name' AS name FROM persons;
  * 输出JSON数据
    * 将多条记录转换为单个JSON对象
      * SELECT row_to_json(f) AS x
        FROM (
            SELECT id, json_array_elements(person->'children')->>'name'
            AS cname FROM persons
              )
        AS f;
    * 将表中所有记录行整体打包转换一个json对象
      * SELECT row_to_json(f) AS jsoned_row
        FROM persons AS f;
* JSON 类型的二进制版本：JSONB
  * 存储原始文本解析后生成的二进制数据结构
  * 会对原始内容做一些转换
  * 不允许键值重复
  * 可以在字段上建立GIN索引

* 全文检索


### 第六章 表、约束和索引
* 基本建表操作
  * serial
    * 自增长的数字类型。建表时如果有一个serial类型的字段，那么系统会自动在schema中同时创建一个对应的序列号生成器。
    * 对于特别大的表，应该使用bigserial类型
  * IDENTITY
    * 也可将字段定义为自增序列号类型
    * 底层不再依赖自动生成的序列号生成器
    * CREATE TABLE logs (
      log_id int GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY ,
      user_name varchar(50),
      desription text,
      log_ts timestamp with time zone NOT NULL DEFAULT current_timestamp
      );
* 继承表
  * CREATE TABLE logs_2011(PRIMARY KEY (log_id)) INHERITS (logs);

* 原生分区表支持
  * CREATE TABLE .. PARTITION BY RANGE ..
  * 查询父表时，所有不符合条件的子分区都会被跳过

* 无日志表
  * 写入记录的速度远远超过普通的有日志表，10-15倍左右

* 约束机制
  * 外键约束
  * 唯一性约束
  * check约束
    * 查询规划器会利用check约束来优化执行速度
    * ALTER TABLE logs ADD CONSTRAINT chk CHECK(user_name = lower(user_name));
  * 排他性约束
    * 扩展了唯一性比较的算法机制，可以使用更多的运算符来进行比较运算

* 索引
  * B-树索引
  * BRIN索引(block range index)
    * 针对超大表做索引
    * 查询速度相对较慢
    * 不能用于确保唯一主键
  * GiST(generalized search tree)
    * 适用于全文检索以及空间数据、科学数据、非结构化数据和层次化数据的搜索
    * 不能保障字段唯一性，用于排他性约束就可以保证唯一性
    * 有损索引，它不存储被索引字段的值，仅存储一个sample，意味着需要一个额外查找才能找到真正的值
  * GIN索引(generalized inverted index, 通用逆序索引)
    * 适用于全文搜索引擎以及二进制json数据类型
    * 无损索引
  * SP-GiST索引(space-partitioning trees)
    * 与GiST索引的适用领域相同
    * 对于某些特定领域的数据算法效率更高
  * 散列索引
    * 某些场景中比B-树更快
  * 基于B-树算法的GiST和GIN索引