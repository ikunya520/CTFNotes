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
#### SVN泄露：
#### HG泄露：
### 密码口令：
弱口令：
默认口令：
字典爆破：
### SQL注入：
整数型注入：
字符型注入：
报错注入：
布尔型注入：
时间盲注：
mysql结构：
cookie注入：
UA注入：
refer注入：
空格注入：
update注入：
insert注入：
### 文件上传：
无验证：
前端验证：
.htaccess:
MIME绕过：
00截断：
双写绕过：
文件头部检查：
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
