# mysql

1、在写sql语句时，不管字段为什么类型，占位符统一使用`%s`,且不能加上引号（占位符 `%s` 只能出现在值的地方）。例如

```python
sql="insert into tablename (id,name) values (%s,%s)"
```


2、添加的数据的格式必须为list[tuple(),tuple(),tuple()]或者tuple(tuple(),tuple(),tuple())例如

```python
values=[(1,"zhangsan"),(2,"lisi")]
或者
values=((1,"zhangsan"),(2,"lisi"))
```


最后，通过executemany插入

```
cursor.executemany(sql,values) 
```





MySQL本身有个load data infile的方法（速度比executemany更快），格式类似这样：

```sql
load data infile 'D:/Python workspace/user.txt' into table user(username, salt, pwd)
```



传入一个字典来插入数据的方法，不需要再去修改SQL语句和插入操作

```python
data = {
    'id': '20120001',
    'name': 'Bob',
    'age': 20
}
table = 'students'
keys = ', '.join(data.keys())
values = ', '.join(['%s'] * len(data))
sql = 'INSERT INTO {table}({keys}) VALUES ({values})'.format(table=table, keys=keys, values=values)
try:
   if cursor.execute(sql, tuple(data.values())):
       print('Successful')
       db.commit()
except:
    print('Failed')
    db.rollback()
db.close()
```



值得注意的是，需要执行`db`对象的`commit()`方法才可实现数据插入，这个方法才是真正将语句提交到数据库执行的方法。对于数据插入、更新、删除操作，都需要调用该方法才能生效。

接下来，我们加了一层异常处理。如果执行失败，则调用`rollback()`执行数据回滚，相当于什么都没有发生过。

这里涉及事务的问题。事务机制可以确保数据的一致性，也就是这件事要么发生了，要么没有发生。比如插入一条数据，不会存在插入一半的情况，要么全部插入，要么都不插入，这就是事务的原子性。另外，事务还有3个属性——一致性、隔离性和持久性。这4个属性通常称为ACID特性，具体如下表所示。

| 属性                  | 解释                                                         |
| --------------------- | ------------------------------------------------------------ |
| 原子性（atomicity）   | 事务是一个不可分割的工作单位，事务中包括的诸操作要么都做，要么都不做 |
| 一致性（consistency） | 事务必须使数据库从一个一致性状态变到另一个一致性状态。一致性与原子性是密切相关的 |
| 隔离性（isolation）   | 一个事务的执行不能被其他事务干扰，即一个事务内部的操作及使用的数据对并发的其他事务是隔离的，并发执行的各个事务之间不能互相干扰 |
| 持久性（durability）  | 持续性也称永久性（permanence），指一个事务一旦提交，它对数据库中数据的改变就应该是永久性的。接下来的其他操作或故障不应该对其有任何影响 |

插入、更新和删除操作都是对数据库进行更改的操作，而更改操作都必须为一个事务，所以这些操作的标准写法就是：

```
try:
    cursor.execute(sql)
    db.commit()
except:
    db.rollback()复制代码
```

这样便可以保证数据的一致性。这里的`commit()`和`rollback()`方法就为事务的实现提供了支持。



`fetchall()`方法的内部实现有一个偏移指针用来指向查询结果，最开始偏移指针指向第一条数据，取一次之后，指针偏移到下一条数据，这样再取的话，就会取到下一条数据了。我们最初调用了一次`fetchone()`方法，这样结果的偏移指针就指向下一条数据，`fetchall()`方法返回的是偏移指针指向的数据一直到结束的所有数据



如果数据存在，则更新数据；如果数据不存在，则插入数据。另外，这种做法支持灵活的字典传值。示例如下：

```python
data = {
    'id': '20120001',
    'name': 'Bob',
    'age': 21
}

table = 'students'
keys = ', '.join(data.keys())
values = ', '.join(['%s'] * len(data))

sql = 'INSERT INTO {table}({keys}) VALUES ({values}) ON DUPLICATE KEY UPDATE'.format(table=table, keys=keys, values=values)
update = ','.join([" {key} = %s".format(key=key) for key in data])
sql += update
try:
    if cursor.execute(sql, tuple(data.values())*2):
        print('Successful')
        db.commit()
except:
    print('Failed')
    db.rollback()
db.close()
```

这里构造的SQL语句其实是插入语句，但是我们在后面加了`ON DUPLICATE KEY UPDATE`。这行代码的意思是如果主键已经存在，就执行更新操作。比如，我们传入的数据`id`仍然为`20120001`，但是年龄有所变化，由20变成了21，此时这条数据不会被插入，而是直接更新`id`为`20120001`的数据。完整的SQL构造出来是这样的：

```mysql
INSERT INTO students (id, name, age) VALUES (%s, %s, %s) ON DUPLICATE KEY UPDATE id = %s, name = %s, age = %s
```

这里就变成了6个`%s`。所以在后面的`execute()`方法的第二个参数元组就需要乘以2变成原来的2倍。

如此一来，我们就可以实现主键不存在便插入数据，存在则更新数据的功能了。



如果INSERT多行记录(假设 a 为主键或 a 是一个 UNIQUE索引列):

```
1.INSERT INTO TABLE (a,c) VALUES (1,3),(1,7) ON DUPLICATE KEY UPDATE c=c+1;
```

执行后, c 的值会变为 4 (第二条与第一条重复, c 在原值上+1).

```mysql
2.INSERT INTO TABLE (a,c) VALUES (1,3),(1,7) ON DUPLICATE KEY UPDATE c=VALUES(c);
```

执行后, c 的值会变为 7 (第二条与第一条重复, c 在直接取重复的值7).

注意：ON DUPLICATE KEY UPDATE只是MySQL的特有语法，并不是SQL标准语法！



https://stackoverflow.com/questions/26337065/mysqldb-returns-not-all-arguments-converted-with-on-duplicate-key-update

```python
query = "INSERT INTO g4_money (money, loginname, api_name) VALUES (%s, %s, %s) ON DUPLICATE KEY UPDATE money = money + %s"
with DB.cursor() as cur:
    cur.executemany(query, data)
    DB.commit()
会报错：TypeError: not all arguments converted during string formatting
query会执行insert或者update操作，不能批量。
正则匹配query时忽略了ON DUPLICATE KEY UPDATE之后的内容，属于pymysql（等）的bug。
```



