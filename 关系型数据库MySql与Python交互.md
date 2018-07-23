---
title:关系型数据库MySql与Python交互
---

##  关系型数据库MySql与Python交互

### 安装引入模块

- 安装mysql模块

```shell
sudo apt-get install python-mysql
```

- 在文件中引入模块

```python
import Mysqldb
```

### Connection对象

- 用于建立与数据库的连接
- 创建对象：调用connect()方法

```python
conn=connect(参数列表)
```

- 参数host：连接的mysql主机，如果本机是'localhost'
- 参数port：连接的mysql主机的端口，默认是3306
- 参数db：数据库的名称
- 参数user：连接的用户名
- 参数password：连接的密码
- 参数charset：通信采用的编码方式，默认是'gb2312'，要求与数据库创建时指定的编码一致，否则中文会乱码

#### 对象的方法

- close()关闭连接
- commit()事务，所以需要提交才会生效
- rollback()事务，放弃之前的操作
- cursor()返回Cursor对象，用于执行sql语句并获得结果

### Cursor对象

- 执行sql语句
- 创建对象：调用Connection对象的cursor()方法

```python
cursor1=conn.cursor()
```

#### 对象的方法

- close()关闭
- execute(operation [, parameters ])执行语句，返回受影响的行数
- fetchone()执行查询语句时，获取查询结果集的第一个行数据，返回一个元组
- next()执行查询语句时，获取当前行的下一行
- fetchall()执行查询时，获取结果集的所有行，一行构成一个元组，再将这些元组装入一个元组返回
- scroll(value[,mode])将行指针移动到某个位置
  - mode表示移动的方式
  - mode的默认值为relative，表示基于当前行移动到value，value为正则向下移动，value为负则向上移动
  - mode的值为absolute，表示基于第一条数据的位置，第一条数据的位置为0

#### 对象的属性

- rowcount只读属性，表示最近一次execute()执行后受影响的行数
- connection获得当前连接对象

### 增加

- 创建testInsert.py文件，向学生表中插入一条数据

```python
#encoding=utf-8
import MySQLdb
try:
    conn=MySQLdb.connect(host='localhost',port=3306,db='test1',user='root',passwd='mysql',charset='utf8')
    cs1=conn.cursor()
    count=cs1.execute("insert into students(sname) values('张良')")
    print count
    conn.commit()
    cs1.close()
    conn.close()
except Exception,e:
    print e.message
```

### 修改

- 创建testUpdate.py文件，修改学生表的一条数据

```python
#encoding=utf-8
import MySQLdb
try:
    conn=MySQLdb.connect(host='localhost',port=3306,db='test1',user='root',passwd='mysql',charset='utf8')
    cs1=conn.cursor()
    count=cs1.execute("update students set sname='刘邦' where id=6")
    print count
    conn.commit()
    cs1.close()
    conn.close()
except Exception,e:
    print e.message
```

### 删除

- 创建testDelete.py文件，删除学生表的一条数据

```python
#encoding=utf-8
import MySQLdb
try:
    conn=MySQLdb.connect(host='localhost',port=3306,db='test1',user='root',passwd='mysql',charset='utf8')
    cs1=conn.cursor()
    count=cs1.execute("delete from students where id=6")
    print count
    conn.commit()
    cs1.close()
    conn.close()
except Exception,e:
    print e.message
```

### sql语句参数化

- 创建testInsertParam.py文件，向学生表中插入一条数据

```python
#encoding=utf-8
import MySQLdb
try:
    conn=MySQLdb.connect(host='localhost',port=3306,db='test1',user='root',passwd='mysql',charset='utf8')
    cs1=conn.cursor()
    sname=raw_input("请输入学生姓名：")
    params=[sname]
    count=cs1.execute('insert into students(sname) values(%s)',params)
    print count
    conn.commit()
    cs1.close()
    conn.close()
except Exception,e:
    print e.message
```

### 其它语句

- cursor对象的execute()方法，也可以用于执行create table等语句
- 建议在开发之初，就创建好数据库表结构，不要在这里执行

### 查询一行数据

- 创建testSelectOne.py文件，查询一条学生信息

```python
#encoding=utf8
import MySQLdb
try:
    conn=MySQLdb.connect(host='localhost',port=3306,db='test1',user='root',passwd='mysql',charset='utf8')
    cur=conn.cursor()
    cur.execute('select * from students where id=7')
    result=cur.fetchone()
    print result
    cur.close()
    conn.close()
except Exception,e:
    print e.message
```

### 查询多行数据

- 创建testSelectMany.py文件，查询一条学生信息

```python
#encoding=utf8
import MySQLdb
try:
    conn=MySQLdb.connect(host='localhost',port=3306,db='test1',user='root',passwd='mysql',charset='utf8')
    cur=conn.cursor()
    cur.execute('select * from students')
    result=cur.fetchall()
    print result
    cur.close()
    conn.close()
except Exception,e:
    print e.message
```

### 封装

- 观察前面的文件发现，除了sql语句及参数不同，其它语句都是一样的
- 创建MysqlHelper.py文件，定义类

```python
#encoding=utf8
import MySQLdb

class MysqlHelper():
    def __init__(self,host,port,db,user,passwd,charset='utf8'):
        self.host=host
        self.port=port
        self.db=db
        self.user=user
        self.passwd=passwd
        self.charset=charset

    def connect(self):
        self.conn=MySQLdb.connect(host=self.host,port=self.port,db=self.db,user=self.user,passwd=self.passwd,charset=self.charset)
        self.cursor=self.conn.cursor()

    def close(self):
        self.cursor.close()
        self.conn.close()

    def get_one(self,sql,params=()):
        result=None
        try:
            self.connect()
            self.cursor.execute(sql, params)
            result = self.cursor.fetchone()
            self.close()
        except Exception, e:
            print e.message
        return result

    def get_all(self,sql,params=()):
        list=()
        try:
            self.connect()
            self.cursor.execute(sql,params)
            list=self.cursor.fetchall()
            self.close()
        except Exception,e:
            print e.message
        return list

    def insert(self,sql,params=()):
        return self.__edit(sql,params)

    def update(self, sql, params=()):
        return self.__edit(sql, params)

    def delete(self, sql, params=()):
        return self.__edit(sql, params)

    def __edit(self,sql,params):
        count=0
        try:
            self.connect()
            count=self.cursor.execute(sql,params)
            self.conn.commit()
            self.close()
        except Exception,e:
            print e.message
        return count
```

#### 添加

- 创建testInsertWrap.py文件，使用封装好的帮助类完成插入操作

```python
#encoding=utf8
from MysqlHelper import *

sql='insert into students(sname,gender) values(%s,%s)'
sname=raw_input("请输入用户名：")
gender=raw_input("请输入性别，1为男，0为女")
params=[sname,bool(gender)]

mysqlHelper=MysqlHelper('localhost',3306,'test1','root','mysql')
count=mysqlHelper.insert(sql,params)
if count==1:
    print 'ok'
else:
    print 'error'
```

#### 查询一个

- 创建testGetOneWrap.py文件，使用封装好的帮助类完成查询最新一行数据操作

```python
#encoding=utf8
from MysqlHelper import *

sql='select sname,gender from students order by id desc'

helper=MysqlHelper('localhost',3306,'test1','root','mysql')
one=helper.get_one(sql)
print one
```

### 实例：用户登录

#### 创建用户表userinfos

- 表结构如下
  - id
  - uname
  - upwd
  - isdelete
- 注意：需要对密码进行加密
- 如果使用md5加密，则密码包含32个字符
- 如果使用sha1加密，则密码包含40个字符，推荐使用这种方式

```sql
create table userinfos(
id int primary key auto_increment,
uname varchar(20),
upwd char(40),
isdelete bit default 0
);
```

#### 加入测试数据

- 插入如下数据，用户名为123,密码为123,这是sha1加密后的值

```sql
insert into userinfos values(0,'123','40bd001563085fc35165329ea1ff5c5ecbdbbeef',0);
```

#### 接收输入并验证

- 创建testLogin.py文件，引入hashlib模块、MysqlHelper模块
- 接收输入
- 根据用户名查询，如果未查到则提示用户名不存在
- 如果查到则匹配密码是否相等，如果相等则提示登录成功
- 如果不相等则提示密码错误

```sql
#encoding=utf-8
from MysqlHelper import MysqlHelper
from hashlib import sha1

sname=raw_input("请输入用户名：")
spwd=raw_input("请输入密码:")

s1=sha1()
s1.update(spwd)
spwdSha1=s1.hexdigest()

sql="select upwd from userinfos where uname=%s"
params=[sname]

sqlhelper=MysqlHelper('localhost',3306,'test1','root','mysql')
userinfo=sqlhelper.get_one(sql,params)
if userinfo==None:
    print '用户名错误'
elif userinfo[0]==spwdSha1:
    print '登录成功'
else:
    print '密码错误'
```