# SQL

最近在写一些古老的数据维护脚本，主要使用 Python 2.7 + MySQL-python 1.2.5，没有 ORM，手写 SQL 步步精心，以下是一些随意的笔记。

## Python 中使用不定长字典构造 SQL 语句问题

> 转载自 [Python 中使用不定长字典构造 sql 语句问题 - V2EX](https://www.v2ex.com/t/656843)

@todd7zhang: 

``` python
d = {'name': 123, 'age': 456}
keys = d.keys()
sql = 'insert into (%s) value (%s)' % (','.join('%s' % k for k in keys), ','.join('%%s' for _ in keys))
cr.execute(sql, tuple(d.values()))


后面是 %s for_ in keys
关键是 构造出 'insert into (name, age) values (%s, %s)' 就行了

```

@sikong31:

``` python
def insert(self, table, **kw):
fields, values = zip(*kw.items())
fields_str = ','.join(fields)
values_str = ','.join(['?'] * len(fields))
sql = f"INSERT INTO {table} ({fields_str}) values ({values_str})"
self.con.execute(sql, values)
```

@xpresslink:
``` python
>>> d = {'name': 123, 'age': 456}
>>> 'insert into table{0} values{1};'.format(tuple(d.keys()),tuple(d.values()))
"insert into table('name', 'age') values(123, 456);"
>>>


但是不建议一条一条的插入，最好用批量插入，数据量大了效率差万倍。
建一个插入模板
>> sql = 'insert into (name, age) values (%s, %s)'
>> conn.executemany(sql, list(d.values()))
大多数 python 的 mysql 驱动接口都带 executemany()方法。

```

@Trim21:

![](https://tva4.sinaimg.cn/large/bd69bf14ly1gd8pov927oj21j01uydu0.jpg)

## executemany

templet : sql模板字符串,

例如

``` sql
insert into table(id,name) values(%s,%s)
```

args: 模板字符串的参数，是一个列表，列表中的每一个元素必须是元组！

例如：  [(1,'小明'),(2,'zeke'),(3,'琦琦'),(4,'韩梅梅')

*e.g.*

``` python
#coding=utf-8
import MySQLdb
import traceback

tmp = "insert into exch_no_rand_auto(stkcode) values(%s);"   #SQL模板字符串
l_tupple = [(i,) for i in range(100)]   #生成数据参数，list里嵌套tuple

class mymysql(object):
    def __init__(self):
        self.conn = MySQLdb.connect(
            host='127.0.0.1',
            port = 3306,
            user = 'root',
            passwd = '123456',
            db = 'xtp3')

    def insert_sql(self,temp,data):
        cur = self.conn.cursor()
        try:
            cur.executemany(temp,data)
            self.conn.commit()
        except:
            self.conn.rollback()
            traceback.print_exc()
        finally:
            cur.close()

if __name__ == '__main__':
    m = mymysql()
    m.insert_sql(tmp,l_tupple)
#coding=utf-8
import MySQLdb
import traceback

tmp = "insert into exch_no_rand_auto(stkcode) values(%s);"   #SQL模板字符串
l_tupple = [(i,) for i in range(100)]   #生成数据参数，list里嵌套tuple

class mymysql(object):
    def __init__(self):
        self.conn = MySQLdb.connect(
            host='127.0.0.1',
            port = 3306,
            user = 'root',
            passwd = '123456',
            db = 'xtp3')

    def insert_sql(self,temp,data):
        cur = self.conn.cursor()
        try:
            cur.executemany(temp,data)
            self.conn.commit()
        except:
            self.conn.rollback()
            traceback.print_exc()
        finally:
            cur.close()

if __name__ == '__main__':
    m = mymysql()
    m.insert_sql(tmp,l_tupple)
```
