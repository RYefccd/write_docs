

> Written with [StackEdit](https://stackedit.io/).


#  postgresql 


##  安装

16.04
```shell
# postgresql setup
echo "deb http://apt.postgresql.org/pub/repos/apt/ xenial-pgdg main" >> /etc/apt/sources.list.d/pgdg.list
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo apt-get update
apt-get install postgresql-10
```

14.04
```shell
# postgresql setup
echo "deb http://apt.postgresql.org/pub/repos/apt/ trusty-pgdg main" >> /etc/apt/sources.list.d/pgdg.list
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo apt-get update
apt-get install postgresql-10

```





##  操作

    默认的数据库示例
    service postgresql start
    service postgresql stop
    service postgresql restart
    
    新建一个数据库
    在 /var/lib/postgresql/ 新建 pipelinedb 文件夹
    mkdir /var/lib/postgresql/pipelinedb
    # 初始化数据库(配置和相关数据库元信息)
    sudo -i -u postgres /usr/lib/postgresql/10/bin/initdb -D /var/lib/postgresql/pipelinedb
    开启数据库实例
    sudo -i -u postgres /usr/lib/postgresql/10/bin/pg_ctl -D /var/lib/postgresql/pipelinedb/ start
    关闭数据库实例
    sudo -i -u postgres /usr/lib/postgresql/10/bin/pg_ctl -D /var/lib/postgresql/pipelinedb/ stop
    重启数据库实例
    sudo -i -u postgres /usr/lib/postgresql/10/bin/pg_ctl -D /var/lib/postgresql/pipelinedb/ restart



##  psql

###  登录
 
 psql  是用来操作数据的相关命令操作

    初始数据库的账号是以 postgres 用户登录, 数据库用户也是 postgres, 无密码
    sudo -i -u postgres psql 
    或者
    psql -U postgres

    如果是以网络连接访问, 需要密码(以上面的postgres 用户免密登录, 然后按照下面修改密码)
    psql -U postgres -h  10.0.0.30


##  修改密码
```psql
#修改密码
postgres=# \password 
Enter new password: 
Enter it again: 
postgres=# 
```

##   配置允许通过网络连接

http://docs.pipelinedb.com/installation.html#configuration



##   查看状态

###  查看当前登录用户
```psql
postgres=# \c
You are now connected to database "postgres" as user "postgres".

```

###  查看数据库

```psql

postgres=# \d
Did not find any relations.

```

###  查看配置文件数据目录

```shell
postgres=# show config_file;
               config_file               
-----------------------------------------
 /etc/postgresql/10/main/postgresql.conf
(1 row)

postgres=# SHOW data_directory;
       data_directory        
-----------------------------
 /var/lib/postgresql/10/main
(1 row)


```

###  查看扩展

```psql
postgres=# \dx
                   List of installed extensions
    Name    | Version |   Schema   |         Description          
------------+---------+------------+------------------------------
 pipelinedb | 1.0.0   | public     | PipelineDB
 plpgsql    | 1.0     | pg_catalog | PL/pgSQL procedural language
(2 rows)

```

###  查看表大小

 1. \dt+  查看当前库的表的大小
 2. \di+  查看当前库的表的索引的大小
 3. 用 sql 语句查询
```sql
--数据库中单个表的大小（不包含索引）
select pg_size_pretty(pg_relation_size('表名'));

--查出所有表（包含索引）并排序
SELECT table_schema || '.' || table_name AS table_full_name, pg_size_pretty(pg_total_relation_size('"' || table_schema || '"."' || table_name || '"')) AS size
FROM information_schema.tables
ORDER BY
pg_total_relation_size('"' || table_schema || '"."' || table_name || '"') DESC limit 20

--查出表大小按大小排序并分离data与index
SELECT
table_name,
pg_size_pretty(table_size) AS table_size,
pg_size_pretty(indexes_size) AS indexes_size,
pg_size_pretty(total_size) AS total_size
FROM (
SELECT
table_name,
pg_table_size(table_name) AS table_size,
pg_indexes_size(table_name) AS indexes_size,
pg_total_relation_size(table_name) AS total_size
FROM (
SELECT ('"' || table_schema || '"."' || table_name || '"') AS table_name
FROM information_schema.tables
) AS all_tables
ORDER BY total_size DESC
) AS pretty_sizes limit 20;

```

##  DML

###  insert

    插入有自增序列的用法 nextval('legou_user_id_day_v3_seq')
    insert into legou_user_id_day_v3_mrel values ('2019-02-17 00:00:00+08', 'geetest', 'user_id2', 12,12,0,0,nextval('legou_user_id_day_v3_seq'));


##  MVCC & VACUUM

###  mvcc
todo


###  vacuum

####  config

postgresql.conf 中


####  dynamic change

```psql

ALTER TABLE shema.table SET (autovacuum_vacuum_scale_factor = 0.0);
ALTER TABLE shema.table SET (autovacuum_vacuum_threshold = 1000);
ALTER TABLE shema.table SET (autovacuum_analyze_scale_factor = 0.0);
ALTER TABLE shema.table SET (autovacuum_analyze_threshold = 1000);

```

**HINT:** The following query provides information about the last time the Novell schema tables were vacuumed or analyzed:

```psql

select relname,last_vacuum, last_autovacuum, last_analyze, 
vacuum_count, autovacuum_count, last_autoanalyze 
from pg_stat_user_tables 
where schemaname = 'public' 
order by relname ASC;

```




#  pipelinedb

首先安装好 **postgresql 10**  数据库, 步骤如上.
 
 
##    安装

    # 添加源
    curl -s http://download.pipelinedb.com/apt.sh | sudo bash

    sudo apt-get install pipelinedb-postgresql-10
 

pipelinedb 安装完之后,  会出现如下的提示:

```shell

Setting up pipelinedb-postgresql-10 (1.0.0-6~ubuntu14) ...

    ____  _            ___            ____  ____
   / __ \(_)___  ___  / (_)___  ___  / __ \/ __ )
  / /_/ / / __ \/ _ \/ / / __ \/ _ \/ / / / __  |
 / ____/ / /_/ /  __/ / / / / /  __/ /_/ / /_/ /
/_/   /_/ .___/\___/_/_/_/ /_/\___/_____/_____/
       /_/

PipelineDB successfully installed. To get started, initialize a
PostgreSQL database directory:

  initdb -D <data directory>

where <data directory> is a nonexistent directory where you'd
like all of your database files to live.

Next, edit <data directory>/postgresql.conf and set:

  shared_preload_libraries = 'pipelinedb'
  max_worker_processes = 128

Once your database is running, create the pipelinedb extension:

  psql -c 'CREATE EXTENSION pipelinedb'

```


按照上面的提示

 1. 先初始化数据库目录

```shell

    # 初始化数据库
    # To get started, initialize a PostgreSQL database directory
    sudo -i -u postgres /usr/lib/postgresql/10/bin/initdb -D /var/lib/postgresql/pipelinedb
```

 2. 然后再修改配置

```shell
    # 编辑 /var/lib/postgresql/pipelinedb/postgresql.conf
    shared_preload_libraries = 'pipelinedb'
    max_worker_processes = 128
```

 3. 创建扩展
 ```shell
psql -U postgres -c "CREATE EXTENSION pipelinedb"
```



#  sql
     todo: sql statment
