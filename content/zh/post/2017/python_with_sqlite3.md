+++
title = "用python 来操控 sqlite3"
date = 2017-11-12T07:05:00+08:00
lastmod = 2022-02-24T16:00:09+08:00
tags = ["python", "sqlite"]
categories = ["python"]
draft = false
toc = true
+++

python 与嵌入式关系数据库 sqlite3的邂逅

`SQLite` 是一个非常优秀的嵌入式数据库，非常轻量，可以与 Mysql, PostgreSQL 这样的 大型数据库互补使用. 而 Python 标准库中的 `sqlite3` 模块实现了兼容 SQLite 的 [Python DB-API 2.0](https://www.python.org/dev/peps/pep-0249/)接口, 因此我们可以很方 便地使用 `sqlite3` 模块来操作 `SQLite`


## <span class="section-num">1</span> 入门 {#入门}


### <span class="section-num">1.1</span> 创建数据库 {#创建数据库}

`SQLite` 数据库是存储在文件系统的单个文件上的，所以如果数据库文件不存在，那么在第一次访问这个数据库，就会创建相应的数据库文件。

```python
import os
import sqlite3

db_filename = 'sqlite3_demo.db'
db_exist = os.path.exists(db_filename)
conn = sqlite3.connect(db_filename)
if db_exist:
    print('Database exists')
else:
    print("Database does not exist")
conn.close()
```

上面的例子会在连接数据库之前检查一下数据库文件是否存在，然后使用 `connect()` 函数连接数据库。

你在执行该代码之前查看一下当前目录的话，如果不存在 `sqlite3_demo.db` 的话，那么跑完这段代码，你应该会看到 `sqlite3_demo.db`
文件的.

这段代码本身是没有做多少事，我只是用它来阐述一下 `SQLite` 的原理


### <span class="section-num">1.2</span> 创建表 {#创建表}

那么，现在，让我们用 `SQLite` 来做点数据库的本份工作。先创建一张表，接下来的操作都会围绕着这张表进行。

`user.sql`:

```sql
create table role(
name         text primary key,
description  text
);
create table user (
id           integer primary key autoincrement not null,
name         text,
phone_number integer,
birthday     date,
role      text not null references role(name)
);
```

然后使用 `Connection` 对象的 `executescript()` 函数来创建表以及插入对应的数据

```python
import os
import sqlite3

db_filename = 'sqlite3_demo.db'
schema_filename = 'user.sql'

db_exists = os.path.exists(db_filename)
with sqlite3.connect(db_filename) as conn:
    if db_exists:
	print('Creating schema')
	with open(schema_filename, 'rt') as f:
	    schema = f.read()
	conn.executescript(schema)

	print('Inserting initial data')
	conn.executescript("""
	insert into role (name,description)
	values ('student','This is a student');

	insert into role (name,description)
	values ('teacher','This is a teacher');

	insert into user (id,name,phone_number,birthday,role)
	values (1,'Samray',12345678,'2017-11-10','student');

	insert into user (id,name,phone_number,birthday,role)
	values (2,'Paul',3231546,'2017-11-11','student');

	insert into user (id,name,phone_number,birthday,role)
	values (3,'Trump',13254768,'2017-11-12','teacher');
			   """)
```


### <span class="section-num">1.3</span> 检索数据 {#检索数据}

如果想要使用检索存储在 `user` 表中的数据，那么就需要从数据库连接对象 `Connection` 中创建一个 `Cursor`对象。

而`Cursor` 对象负责与数据库进行交互并获取 数据

```python
import sqlite3

db_filename = 'sqlite3_demo.db'
with sqlite3.connect(db_filename) as conn:
    cursor = conn.cursor()
    cursor.execute("""
    select id,name,phone_number,birthday from user
    where role='student'
		   """)
    for row in cursor.fetchall():
	id, name, phone_number, birthday = row
    print('{:2d} {} {:<10} [{:<8}]'.format(
	id, name, phone_number, birthday))
```

`SQLite3` 数据库的查询分成两步。首先，使用 `Cursor` 对象的 `execute()` 对象执行查询语句，告诉数据库引擎我们需要什么样的数据，然后，使用 `fetchall()` 函数把数据集从数据库的返回结果中取出来。

返回结果是包含着一系列 `tuple` 的列表，而`tuple` 中对应着的数据就是 `select` 语句指定返回的字段值。

`fetchall()`函数是把所有符合 条件的结果一次性返回，如果需要的话，我们可以使用`fetchone()`函数返回单条记录， 或者使用`fetchmany()`返回固定数量的记录

```python
import sqlite3

db_filename = 'sqlite3_demo.db'
with sqlite3.connect(db_filename) as conn:
    cursor = conn.cursor()
    cursor.execute("""
    select name, description from role
    where name='teacher'
		   """)
    name, description = cursor.fetchone()
    print('Role details for {} ({}) \n'.format(name, description))

    cursor.execute("""
    select id,name,phone_number,birthday from user
    where role='student'
		   """)
    print('/nNext 10 tasks:')
    for row in cursor.fetchmany(10):
	id, name, phone_number, birthday = row
	print('{:2d} {} {:<10} [{:<8}]'.format(
	    id, name, phone_number, birthday))
```

使用 `fetchmany()` 函数需要注意的是，当你指定的数量超过了符合条件的全部记录的数量的时候，`fetchmany()`只会返回全部记录的数量。

例如上面的代码里面，我想要 `fetchmany()` 返回10条记录，但是我的数据库只有2条符合条件的数据，而 `fetchmany()` 之后返回两条记录


### <span class="section-num">1.4</span> Row 对象 {#row-对象}

在先前的内容内，我已经提到，数据库返回的数据行都是以 `tuple`的形式返回的，所以 程序调用者必须知道查询语句字段的顺序，然后在`tuple`取出记录的时候把字段名和变量名一一对应上，例如
`name, description = cursor.fetchone()`.

查询语句中字段不多的时候或许还能记住，但是如果字段值多了起来，就很容易出现问题.

如果可以像`value=dict['key']` 那样使用键值对的形式获取数据，那样就方便很多.

而`sqlite3`也有为你提供这样便利的操作，诀窍就在使用 `Row` 对象。`sqlite3` 可以把查询结果映 射到 Row 对象，然后我们就可以通过`Row[字段名']` 这种方式来获取指定字段对应的值。

```nil
import sqlite3

db_filename = 'sqlite3_demo.db'
with sqlite3.connect(db_filename) as conn:
    conn.row_factory = sqlite3.Row
    cursor = conn.cursor()
    cursor.execute("""
    select name, description from role
    where name='teacher'
		   """)
    name, description = cursor.fetchone()
    print('Role details for {} ({}) \n'.format(name, description))

    cursor.execute("""
    select id,name,phone_number,birthday from user
    where role='student'
		   """)
    print('/nNext 10 tasks:')
    for row in cursor.fetchmany(10):
	print('{:2d} {} {:<10} [{:<8}]'.format(
	    row['id'], row['name'], row['phone_number'], row['birthday']))
```

通过指定 `Connection` 对象的 `row_factory` 属性就可以控制查询结果集返回的对象。

在上面的代码，我们使用了 `Row` 对象而不是 `tuple` 来获取数据，而程序的执行结果都是相同，但是程序的健壮性就得到了提高。


### <span class="section-num">1.5</span> 在查询中使用变量 {#在查询中使用变量}

我们上面的代码里面的查询语句都是硬编码的，不利于扩展。如果你希望可以使用更灵活的查询语句，你可能会去用字符串拼接查询语句。

但是这样的做法是不被提倡的，因为很容易出现安全问题，比如说 SQL 注入. 比较提倡的方式是在执行 `execute()` 函数的时候进行 变量替换，使用变量替换可以避免SQL注入攻击，因为那些不被信任的代码没办法被解析。

```python
import sqlite3

db_filename = 'sqlite3_demo.db'
with sqlite3.connect(db_filename) as conn:
    conn.row_factory = sqlite3.Row
    cursor = conn.cursor()
    sql = """
    select id,name,phone_number,birthday from user
    where role=:role_name
		   """
    cursor.execute(sql, {'role_name': 'student'})
    print('/nNext 10 tasks:')
    for row in cursor.fetchmany(10):
	print('{:2d} {} {:<10} [{:<8}]'.format(
	    row['id'], row['name'], row['phone_number'], row['birthday']))
```

如上面的代码所示，使用 `:role_name` 占位符来表示 `role_name`变量, 然后在执行 SQL 语句的时候把 `role_name`的值传到 SQL 语句里面去。


### <span class="section-num">1.6</span> 批量插入 {#批量插入}

我们之前提到的插入都是使用 `execute()` 函数逐条插入的，但是 `sqlite3` 也是支持批 量插入的, 使用 `executemany()`函数就可以实现一次插入批量的数据，而函数的底层也 是对插入多条数据的循环进行了优化的，这些就无需调用者操心了。

user.csv

```csv
birthday,name,id,phone_number
2018-11-30,Torres,22,98564311
2010-08-10,Messi,12,81582236
2018-11-21,Saul,9,23564548
```

```python
import csv
import sqlite3

db_filename = 'sqlite3_demo.db'
data_filename = 'users.csv'
SQL = """
insert into user (id,name,phone_number,birthday,role)
values (:id,:name,:phone_number,:birthday,'student')
    """
with open(data_filename, 'rt') as csv_file:
    csv_reader = csv.DictReader(csv_file)
    with sqlite3.connect(db_filename) as conn:
	cursor = conn.cursor()
	cursor.executemany(SQL, csv_reader)
```

我们从 csv 文件中批量导入数据，而Python 的标准库也内置了 CSV 的解析器，使用 `DictReader` 就是将 csv 文件解析成
`{'id':22,'birthday':'2018-11-30','name':'Torres','phone_number':98564311}`的形式
然后配合上面提到的命名变量，把所有数据插入到数据库。


## <span class="section-num">2</span> 进阶 {#进阶}

自定义数据库列类型 `SQLite` 的数据列原生支持整型(integer), 浮点数(floating point), 文本类型 (text), 并且由 `sqlite3` 转换成 Python内置的数据类型。

例如：数据库的整型可以转 换成Python 的 `int` 或者是 `long`, 具体取决于值的大小；文本类型默认会转换成 `str`
类型，除非我们修改了 `Connection` 对象的 `text_factory` 属性。

虽然 `SQLite` 内部支持的数据类型不多，但是得益于 `sqlite3` 的内置机制的支持，我们可以 定义程序自己的数据列。

```python
import sqlite3
import sys

db_filename = 'sqlite3_demo.db'
sql = 'select id,name,birthday from user'


def show_birthday(conn):
    conn.row_factory = sqlite3.Row
    cursor = conn.cursor()
    cursor.execute(sql)
    row = cursor.fetchone()
    for col in ['id', 'name', 'birthday']:
	print('{:<8} {:<10} {}'.format(col, row[col], type(row[col])))
    return


print('Without type detection:')
with sqlite3.connect(db_filename) as conn:
    show_birthday(conn)
print('\nWith type detection:')
with sqlite3.connect(db_filename, detect_types=sqlite3.PARSE_DECLTYPES,) as conn:
    show_birthday(conn)
```

如上面的代码所示，如果你想在Python 数据类型和 `SQLite` 数据列转换的时候使用 `SQLite` 原本不支持的类型，你可以在调用 `connect()` 函数的时候，传一个 `detect_types` 参数进去，而 `PARSE_DECLTYPES` 的意思是指转换成字段声明时候的类型， 比如 `birthday` 声明成 `datetime`类型，但是没有指定成 `PAESE_DECLTYPES` 的时候， 转换成 `str`, 指定后，转换成 `datetime`.

现在我们就来说说怎么定义自己的数据列类型:

我们需要注册两个函数，一个函数把 Python 对象转换成 `byte string` 存储到数据 库里面去，这个函数被称为 `adapter(适配器)`; 既然有从Python 对象转换到数据库存储 对象的函数，那么自然就有从数据库存储转换成 Python 对象的函数，这个函数被称为 `converter(转换器)`.

然后就需要使用 `register_adapter()` 函数将一个函数注册成 `adapter` 函数，至于`register_converter()`函数，也是同理可得了。

```python
import pickle
import sqlite3

db_filename = 'sqlite3_demo.db'


def adapter_func(obj):
    """Covert from python to sqlite3 representation
    """
    print('adapter_func({})\n'.format(obj))
    return pickle.dumps(obj)


def converter_func(data):
    """Convert from sqlite3 to python representation
    """
    print('converter_func({})'.format(data))
    return pickle.loads(data)

# custom type


class MyObj:
    def __init__(self, arg):
	self.arg = arg

    def __str__(self):
	return 'MyObj({!r})'.format(self.arg)


# Register functions
sqlite3.register_adapter(MyObj, adapter_func)
sqlite3.register_converter("MyObj", converter_func)

# Create some objects to save
to_save = [
    (MyObj('this is a value to save'),),
    (MyObj(42),)
]
with sqlite3.connect(db_filename, detect_types=sqlite3.PARSE_DECLTYPES) as conn:
    conn.execute("""
     create table if not exists obj (
	id    integer primary key autoincrement not null,
	data  MyObj
    )
		 """)
    cursor = conn.cursor()
    cursor.executemany("insert into obj (data) values (?)", to_save)
    # Query the database for the objects just saved
    cursor.execute("select id, data from obj")
    for obj_id, obj in cursor.fetchall():
	print('Retrieved', obj_id, obj)
	print('  with type', type(obj))
	print()
```

上面的例子使用了Python 标准库的 `pickle` 模块，将一个 Python 对象转换成可以保存 到数据库的字符串，然后使用 `pickle`
把字符串转换成Python 对象。

这就基本实现了自定义的数据类型。不过我们自己实现的这种自定义数据类型是有局限的，我们只能把整个 Python 对象当作字符串来查询，而没办法针对 Python 对象的属性进行查询，如果你感兴趣的话，你可以看看 Python ORM
框架是怎么实现这些功能的。


### <span class="section-num">2.1</span> 事务 {#事务}

谈及关系型数据库，必不可少的一定是事务。对于事务的见解，网上的资料都已经浩如烟海 了，那么，就要我们直接来说一下 `SQLite` 事务的使用


#### <span class="section-num">2.1.1</span> commit {#commit}

对数据库的修改操作，无论是新增(insert) 还是更新 (update), 都需要调用 `commit()` 来保存。

```python
import sqlite3

db_filename = 'sqlite3_demo.db'


def show_role(conn):
    cursor = conn.cursor()
    cursor.execute('select name, description from role')
    for name, description in cursor.fetchall():
	print('  ', name)


with sqlite3.connect(db_filename) as conn1:
    print('Before changes:')
    show_role(conn1)

    # Insert in one cursor
    cursor1 = conn1.cursor()
    cursor1.execute("""
    insert into role (name, description)
    values ('president','well, this is a president')
		    """)

    print('\nAfter changes in conn1:')
    show_role(conn1)

    # 在没有提交事务之前，使用其它的数据库连接进行查询
    print('\nBefore commit:')
    with sqlite3.connect(db_filename) as conn2:
	show_role(conn2)

    # 提交事务，然后使用另外的数据库连接进行查询
    conn1.commit()
    print('\nAfter commit:')
    with sqlite3.connect(db_filename) as conn3:
	show_role(conn3)
```

`commit()` 函数的调用结果可以被使用若干个数据库连接的程序查询到，在第一个数据库连接插入了一行新的数据，另外两个数据库连接尝试读取到新插入的数据。

当 `show_role()` 函数在 `conn1` 提交事务之前被调用，返回结果就取决于调用 `show_role()` 是哪个数据连接了。

因为是通过 `conn1`来修改数据库，所以它可以看到修改后的数据，但是 `conn2`看不到。在提交事务之后(`commit()`) ,通过其他的数据库连接 (conn3)也可以看到修改结果了


#### <span class="section-num">2.1.2</span> rollback {#rollback}

未提交的修改可以通过调用`rollback()` 函数全部丢弃。通常 `commit()` 和 `rollback()` 函数都是在 `try-except` 语句块的不同地方被调用的，例如错误异常触发, 事务回滚(rollback)

```python
import sqlite3

db_filename = 'sqlite3_demo.db'


def show_role(conn):
    cursor = conn.cursor()
    cursor.execute('select name, description from role')
    for name, description in cursor.fetchall():
	print('  ', name)


with sqlite3.connect(db_filename) as conn1:
    print('Before changes:')
    show_role(conn1)

    try:
	# Delete
	cursor1 = conn1.cursor()
	cursor1.execute("""
	delete from role where name='president'
			""")

	print('\nAfter delete')
	show_role(conn1)

	# 模拟接下来的操作出现了错误
	raise RuntimeError('This is an error')
    except Exception as error:
	# 丢弃之前的修改
	print('Error:', error)
	conn1.rollback()
    else:
	# 保存修改，提交事务
	conn1.commit()
    print('\nAfter rollback:')
    show_role(conn1)
```

在调用 `rollback()` 函数回滚事务之后，对数据库的修改都丢弃了。


### <span class="section-num">2.2</span> 内存型数据库 {#内存型数据库}

正如我们先前提到的，`SQLite` 是文件型数据库，它通过文件系统来管理数据库。但是 `SQLite` 也可以把整个数据库放到内存中去。

```python
import sqlite3

schema_filename = 'user.sql'

with sqlite3.connect(':memory:') as conn:
    conn.row_factory = sqlite3.Row

    print('Creating schema')
    with open(schema_filename, 'rt') as f:
	schema = f.read()
    conn.executescript(schema)

    print('Inserting initial data')
    conn.execute("""
    insert into role (name,description)
    values ('Admin', 'wow, administrator'
	    )
    """)
    data = [
	('Xi', 119, '1910-10-03','president'),
	('Jiang', 110, '2020-10-10','president'),
	('Mao', 10086, '2010-10-17','president'),
    ]
    conn.executemany("""
    insert into user (name, phone_number, birthday,role)
    values (?, ?, ?,?)
    """, data)

    print('Dumping:')
    for text in conn.iterdump():
	print(text)
```

想要把 `SQLite` 当作内存型数据库，只需在调用 `connect()` 函数的时候，使用 `:memory:` 参数而不是数据库文件的文件名。

需要注意的是，每一个 `connect()` 函数都会打开新建一个数据库实例，所以在一个数据库连接上的修改是不会影响其它的连接的。

而 `iterdump()` 函数会返回一个迭代器，输出一系列对数据库修改的 SQL.

最后需要注意的是，使用内存型的数据库是有风险的，要切记这一点。


### <span class="section-num">2.3</span> 在SQL 使用 Python 函数 {#在sql-使用-python-函数}

`SQLite` 支持在查询的时候使用注册了的 Python函数的，这个特性就使我们在可以获取到 查询结果之前先对数据进行加工，或者调用Python 函数实现那些 纯SQL 力所不能及的功能

```python
import codecs
import sqlite3

db_filename = 'sqlite3_demo.db'


def encrypt(s):
    print('Encrypting {!r}'.format(s))
    return codecs.encode(s, 'rot-13')


def decrypt(s):
    print('Decrypting {!r}'.format(s))
    return codecs.encode(s, 'rot-13')


with sqlite3.connect(db_filename) as conn:
    conn.create_function('encrypt', 1, encrypt)
    conn.create_function('decrypt', 1, decrypt)
    cursor = conn.cursor()

    # Raw values
    print('Original values:')
    query = "select id, name from user"
    cursor.execute(query)
    for row in cursor.fetchall():
	print(row)

    print('\nEncrypting...')
    query = "update user set name = encrypt(name)"
    cursor.execute(query)

    print('\nRaw encrypted values:')
    query = "select id, name from user"
    cursor.execute(query)
    for row in cursor.fetchall():
	print(row)

    print('\nDecrypting in query...')
    query = "select id, decrypt(name) from user"
    cursor.execute(query)
    for row in cursor.fetchall():
	print(row)

    print('\nDecrypting...')
    query = "update user set name = decrypt(name)"
    cursor.execute(query)
```

通过 `create_function()` 注册了两个可供 SQL 使用的函数，而 `create_function()`
的参数分别是定义函数的名字，函数传递的参数的个数，以及源函数


## <span class="section-num">3</span> 总结 {#总结}

虽说 `SQLite` 只是一个嵌入式的轻量数据库，但是麻雀虽小，五脏俱全嘛。

内置的 `sqlite3` 库为Python 和 `SQLite` 的沟通构建了一个便捷的桥梁，但是这个桥梁只是个木桥，如果你希望使用斜拉索跨海大桥的话，你就需要去了解 [sqlalchemy](http://www.sqlalchemy.org/), 那是一个功能完善的 ORM 框架 :)


## <span class="section-num">4</span> 参考 {#参考}

-   [sqlalchemy](http://www.sqlalchemy.org/)
-   [sqlite](http://www.sqlite.org/)
-   [sqlite3](https://docs.python.org/3.5/library/sqlite3.html)
-   [python3 module of week](https://pymotw.com/3/index.html)