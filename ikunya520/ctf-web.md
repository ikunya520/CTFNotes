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
#### 前端JS验证：
1）火狐浏览器输入about config禁用javascript  2）F12开发者模式修改JS代码后上传  3）burp抓包修改
#### 服务端效验：
##### content-Type绕过：
bp抓包修改成image/jpeg  

##### 文件名绕过：
 Apache HTTP 服务器，可以尝试phtml,php3,php4,php5,pht等后缀名，但前提是配置文件中有 AddType application/x-httpd-php .php .phtml .phps .php5 .pht .htaccess

#### .htaccess绕过:
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
#### 00截断：
原理：0x00是字符串的结束标识符，攻击者可以利用手动添加字符串标识符的方式来将后面的内容进行截断，而后面的内容又可以帮助我们绕开检测。上传路径%00截断:save_path改成/upload/11.php%00，最后保存下来就是11.php
条件：php<5.3.29，且GPC关闭
**POST**：%00在GET中被url解码后是空字符，但是在POST中%00不会被url解码。所以只能通过burpsuite修改hex值为00进行截断
#### 双写绕过：
文件名改成xx.phphpp，适用与检测到非法后缀就直接删除
#### 文件头部检查：
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
