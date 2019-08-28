# 前言 {#_1}

以前觉得 sqlmap 自己玩得挺溜了，结果最近有一个任务，需要调用 sqlmap api 接口来验证存在 sql 注入漏洞的站点，一开始听到这个任务觉得完了，可能完成不了了，后来我去网上搜了搜相关的资料，发现关于这方面的资料确实挺少的，于是参观了一下sqlmap 的源码，大致摸清楚了如何调用 api 接口。因此，笔者打算写一篇完整些的文章，才有了本文。

**笔者技术有限，有错误或者写的不好的地方敬请谅解！**

# 为什么要使用sqlmap API？ {#sqlmap-api}

由于SQLMAP每检测一个站点都需要开启一个新的命令行窗口或者结束掉上一个检测任务。虽然 -m 参数可以批量扫描URL，但是模式也是一个结束扫描后才开始另一个扫描任务。 通过 api 接口，下发扫描任务就简单了，无需开启一个新的命令行窗口。

# 下载与安装 {#_2}

如果您需要使用 sqlmap api接口或者没安装 sqlmap 工具的，您需要下载安装sqlmap程序。而sqlmap是基于Python 2.7.x 开发的，因此您需要下载Python 2.7.x。

* Python 2.7.x 下载地址：
  [https://www.python.org/downloads/release/python-2715/](https://www.python.org/downloads/release/python-2715/)
* sqlmap 下载地址：
  [https://github.com/sqlmapproject/sqlmap/zipball/master](https://github.com/sqlmapproject/sqlmap/zipball/master)

安装过程我这里就不详细说了，不会的话可以问问度娘\([https://www.baidu.com/s?wd=sqlmap%E5%AE%89%E8%A3%85%E6%95%99%E7%A8%8B](https://www.baidu.com/s?wd=sqlmap安装教程)\)

sqlmap的目录结构图如下：

![](/assets/sqlmap-1.png)

sqlmap 安装完成后，输入以下命令，返回内容如下图一样，意味着安装成功：

```
python sqlmap.py -h
```

![](/assets/sqlmap-2.png)

# sqlmap api {#sqlmap-api_1}

说了那么多，到底 api 如何使用呢？在下载安装 SQLMAP 后，你会在 sqlmap 安装目录中找到一个 sqlmapapi.py 的文件，这个 sqlmapapi.py 文件就是 sqlmmap api。 sqlmap api 分为服务端和客户端，sqlmap api 有两种模式，一种是基于 HTTP 协议的接口模式，一种是基于命令行的接口模式。

## sqlmapapi.py 的使用帮助 {#sqlmapapipy}

通过以下命令获取 sqlmapapi.py 的使用帮助：

```bash
python sqlmapapi.py -h
```

返回的信息：

```bash
Usage: sqlmapapi.py [options]

Options:
? -h, --help? ? ? ? ? ? 显示帮助信息并退出
? -s, --server? ? ? ? ? 做为api服务端运行
? -c, --client? ? ? ? ? 做为api客户端运行
? -H HOST, --host=HOST? 指定服务端IP地址 (默认IP是 "127.0.0.1")
? -p PORT, --port=PORT? 指定服务端端口 (默认端口8775)
? --adapter=ADAPTER? ? ?服务端标准接口 (默认是 "wsgiref")
? --username=USERNAME? ?可空，设置用户名
? --password=PASSWORD? ?可空，设置密码
```

## 开启api服务端 {#api}

无论是基于 HTTP 协议的接口模式还是基于命令行的接口模式，首先都是需要开启 api 服务端的。 通过输入以下命令即可开启 api 服务端:

```py
python sqlmapapi.py -s
```

命令成功后，在命令行中会返回一些信息。以下命令大概的意思是 api 服务端在本地 8775 端口上运行，admin token 为ff053d6517b86f06dec73bda814a70a1，IPC 数据库的位置在/var/folders/fz/16wqcnkd6477gz4wcr\_63xcc0000gn/T/sqlmapipc-N0m7P7，api 服务端已经和 IPC 数据库连接上了，正在使用 bottle 框架 wsgiref 标准接口。

![](/assets/sqlmap-3.png)但是通过上面的这种方式开启 api 服务端有一个缺点，当服务端和客户端不是一台主机会连接不上，因此如果要解决这个问题，可以通过输入以下命令来开启 api 服务端:

```py
python sqlmapapi.py -s -H "0.0.0.0" -p 8775
```

命令成功后，远程客户端就可以通过指定远程主机 IP 和端口来连接到 API 服务端。

### 固定 admin token {#admin-token}

如果您有特殊的需求需要固定 admin token 的话，可以修改文件 api.py，该文件在 sqlmap 目录下的`/lib/utils/`中，修改该文件的第 661 行代码，以下是源代码：

```py
DataStore.admin_token = hexencode(os.urandom(16))
```

##  {#sqlmap-api_2}

## sqlmap api 的两种模式 {#sqlmap-api_2}

### 命令行接口模式 {#_3}

输入以下命令，可连接 api 服务端，进行后期的指令发送操作：

```
python sqlmapapi.py -c
```

如果是客户端和服务端不是同一台计算机的话，输入以下命令：

```py
python sqlmapapi.py -c -H "192.168.1.101" -p 8775
```

输入以上命令后，会进入交互模式，如下图所示：

![](/assets/sqlmap-4.png)

#### 命令行接口模式的相关命令 {#_4}

通过在交互模式下输入 help 命令，获取所有命令，以下是该接口模式的所有命令：

```py
api> help
help? ? ? ? ? ?显示帮助信息
new ARGS? ? ? ?开启一个新的扫描任务
use TASKID? ? ?切换taskid
data? ? ? ? ? ?获取当前任务返回的数据
log? ? ? ? ? ? 获取当前任务的扫描日志
status? ? ? ? ?获取当前任务的扫描状态
option OPTION? 获取当前任务的选项
options? ? ? ? 获取当前任务的所有配置信息
stop? ? ? ? ? ?停止当前任务
kill? ? ? ? ? ?杀死当前任务
list? ? ? ? ? ?显示所有任务列表
flush? ? ? ? ? 清空所有任务
exit           退出客户端        t? ? ? ? ? ?
```

既然了解了命令行接口模式中所有命令，那么下面就通过一个 sql 注入来演示该模式接口下检测 sql 注入的流程。

#### 检测 GET 型注入 {#get}

通过输入以下命令可以检测 GET 注入

```
new -u "url"
```

虽然我们仅仅只指定了`-u`参数，但是从返回的信息中可以看出，输入 new 命令后，首先先请求了`/task/new`，来创建一个新的`taskid`，后又发起了一个请求去开始任务，因此可以发现该模式实质也是基于 HTTP 协议的。

![](/assets/sqlmap-5.png)

通过输入 status 命令，来获取该任务的扫描状态，若返回内容中的 status 字段为terminated，说明扫描完成，若返回内容中的 status 字段为 run，说明扫描还在进行中。

下图是扫描完成的截图：

![](/assets/sqlmap-6.png)

通过输入 data 命令，来获取扫描完成后注入出来的信息，若返回的内容中 data 字段不为空就说明存在注入。

下图是存在 SQL 注入返回的内容，可以看到返回的内容有数据库类型、payload、注入的参数等等。  
![](/assets/sqlmap-7.png)

####  {#post-cookieua}

#### 检测 POST 型、cookie、UA 等注入 {#post-cookieua}

通过输入以下命令，在 data.txt 中加入星号，指定注入的位置，来达到检测 POST、cookie、UA 等注入的目的：

```
new -r data.txt
```

![](/assets/sqlmap-9.png)

### 基于HTTP协议的接口模式 {#http}

下列都是基于 HTTP 协议 API 交互的所有方法：

提示：“@get” 就说明需要通过 GET 请求的，“@post” 就说明需要通过 POST 请求的；POST 请求需要修改 HTTP 头中的 Content-Type 字段为`application/json`。

```py
#辅助
@get('/error/401')    
@get("/task/new")
@get("/task/<taskid>/delete")

#Admin 命令
@get("/admin/list")
@get("/admin/<token>/list")
@get("/admin/flush")
@get("/admin/<token>/flush")

#sqlmap 核心交互命令
@get("/option/<taskid>/list")
@post("/option/<taskid>/get")
@post("/option/<taskid>/set")
@post("/scan/<taskid>/start")
@get("/scan/<taskid>/stop")
@get("/scan/<taskid>/kill")
@get("/scan/<taskid>/status")
@get("/scan/<taskid>/data")
@get("/scan/<taskid>/log/<start>/<end>")
@get("/scan/<taskid>/log")
@get("/download/<taskid>/<target>/<filename:path>")
```

#### @get\('/error/401'\) {#geterror401}

该接口在我的理解表明首先需要登录（Admin token），不然会返回状态码 401。 具体代码如下：

```py
response.status = 401
return response
```

#### @get\("/task/new"\) {#gettasknew}

该接口用于创建一个新的任务，使用后会返回一个随机的 taskid。

下图是调用该接口的截图

![](/assets/sqlmap-10.png)图10

#### @get\("/task/&lt;taskid&gt;/delete"\) {#gettaskdelete}

该接口用于删除 taskid。在调用时指定 taskid，不指定 taskid 会有问题。

下图是调用该接口的截图：

![](/assets/sqlmap-11.png)

#### @get\("/admin/list"\) {#getadminlistgetadminlist}

该接口用于返回所有 taskid。在调用时指定 taskid，不指定 taskid 会有问题。

下图是调用该接口的截图：

![](/assets/sqlmap-12.png)

#### @get\("/admin/flush"\) {#getadminflushgetadminflush}

该接口用于删除所有任务。在调用时指定admin token，不指定admin token可能会有问题。下图是调用该接口的截图：

![](/assets/sqlmap-13.png)13

#### @get\("/option/&lt;taskid&gt;/list"\) {#getoptionlist}

该接口可获取特定任务ID的列表选项，调用时请指定taskid，不然会出现问题。

下图是调用该接口的截图：

![](/assets/sqlmap-14.png)

#### @post\("/option/&lt;taskid&gt;/get"\) {#postoptionget}

该接口可获取特定任务ID的选项值，调用时请指定taskid，不然会出现问题。

下图是调用该接口的截图：

![](/assets/sqlmap-15.png)

#### @post\("/option/&lt;taskid&gt;/set"\) {#postoptionset}

该接口为特定任务 ID 设置选项值，调用时请指定 taskid，不然会出现问题。

下图是调用该接口的截图：16

![](/assets/sqlmap-16.png)

#### @post\("/scan/&lt;taskid&gt;/start"\) {#postscanstart}

该接口定义开始扫描特定任务，调用时请指定 taskid，不然会出现问题。

下图是调用该接口的截图：

![](/assets/sqlmap-17.png)

#### @get\("/scan/&lt;taskid&gt;/stop"\) {#getscanstop}

该接口定义停止扫描特定任务，调用时请指定 taskid，不然会出现问题。

下图是调用该接口的截图：

![](/assets/sqlmap-18.png)

#### @get\("/scan/&lt;taskid&gt;/kill"\) {#getscankill}

该接口可杀死特定任务，需要指定 taskid，不然会出现问题。

#### @get\("/scan/taskid/status"\) {#getscanstatus}

该接口可查询扫描状态，调用时请指定 taskid，不然会出现问题。 下图是调用该接口的截图：19

![](/assets/sqlmap-19.png)

#### @get\("/scan/&lt;taskid&gt;/data"\) {#getscandata}

该接口可获得到扫描结果，调用时请指定 taskid，不然会出现问题。

下图是调用该接口的截图：

存在 SQL 注入的返回结果，返回的内容包括 payload、数据库类型等等。

![](/assets/sqlmap-20.png)

不存在注入的返回结果

![](/assets/sqlmap-21.png)

#### @get\("/scan/&lt;taskid&gt;/log"\) {#getscanlog}

该接口可查询特定任务的扫描的日志，调用时请指定 taskid，不然会出现问题。

### 准备

使用该模式接口需要用到 python 的两个库文件，一个是 requests 库，一个是 json 库。

下图是导入库的操作：

![](/assets/sqlmap-22.png)

#### 检测 GET 型注入 {#get_1}

下面是完整的一次 API 接口访问，"从创建任务 ID，到发送扫描指令，再到查询扫描状态，最后查询结果”的过程。

```py
>>> r = requests.get("http://127.0.0.1:8775/task/new")  创建一个新的扫描任务
>>> r.json()
{'taskid': 'c87dbb00644ed7b7', 'success': True} 获取响应的返回内容
>>> r = requests.post('http://127.0.0.1:8775/scan/c87dbb00644ed7b7/start', data=json.dumps({'url':'http://192.168.1.104/sql-labs/Less-2/?id=1'}), headers={'Content-Type':'application/json'})  开启一个扫描任务
>>> r = requests.get("http://127.0.0.1:8775/scan/c87dbb00644ed7b7/status")  查询任务的扫描状态
>>> r.json()
{'status': 'terminated', 'returncode': 0, 'success': True}
>>> r = requests.get("http://127.0.0.1:8775/scan/c87dbb00644ed7b7/data")  
获取扫描的结果
>>> r.json()
{'data': [{'status': 1, 'type': 0, 'value': {'url': 'http://192.168.1.104:80/sql-labs/Less-2/', 'query': 'id=1', 'data': None}}, {'status': 1, 'type': 1, 'value': [{'dbms': 'MySQL', 'suffix': '', 'clause': [1, 8, 9], 'notes': [], 'ptype': 1, 'dbms_version': ['>= 5.0'], 'prefix': '', 'place': 'GET', 'data': {'1': {'comment': '', 'matchRatio': 0.957, 'title': 'AND boolean-based blind - WHERE or HAVING clause', 'trueCode': 200, 'templatePayload': None, 'vector': 'AND [INFERENCE]', 'falseCode': 200, 'where': 1, 'payload': 'id=1 AND 8693=8693'}..., 'success': True, 'error': []}
```

可能您会被最后返回的结果好奇，ptype、suffix、clause等等都是什么意思呢？ 下面我给出部分字段的含义：

![](/assets/sqlmap-23.png)

#### 检测 POST注入、COOKIE、UA等注入 {#postcookieua}

检测 POST 注入和检测 GET 注入类似，但是还是有一定区别的，与 GET 注入检测区别如下，流程上是一样的，不同的是开启扫描任务的时候，多提交一个 data 字段。

```py
requests.post('http://127.0.0.1:8775/scan/cb9c4b4e4f1996b5/start', data=json.dumps({'url':'http://192.168.1.104/sql/sql/post.php','data':'keyword=1'}), headers={'Content-Type':'application/json'})
```

下面是一次完整的 POST 注入检测过程

![](/assets/sqlmap-24.png)

具体输入输出代码如下：

```py
>>> r = requests.get("http://127.0.0.1:8775/task/new")
>>> r.json()
{'taskid': 'cb9c4b4e4f1996b5', 'success': True}
>>> r = requests.post('http://127.0.0.1:8775/scan/cb9c4b4e4f1996b5/start', data=json.dumps({'url':'http://192.168.1.104/sql/sql/post.php','data':'keyword=1'}), headers={'Content-Type':'application/json'})
>>> r.json()
{'engineid': 9682, 'success': True}
>>> r = requests.get("http://127.0.0.1:8775/scan/cb9c4b4e4f1996b5/status")
>>> r.json()
{'status': 'terminated', 'returncode': 0, 'success': True}
>>> r = requests.get("http://127.0.0.1:8775/scan/cb9c4b4e4f1996b5/data")
>>> r.json()
{'data': [{'status': 1, 'type': 0, 'value': {'url': 'http://192.168.1.104:80/sql/sql/post.php', 'query': None, 'data': 'keyword=1'}}, {'status': 1, 'type': 1, 'value': [{'dbms': 'MySQL', 'suffix': '', 'clause': [1, 8, 9], 'notes': [], 'ptype': 1, 'dbms_version': ['>= 5.0.12'], 'prefix': '', 'place': 'POST', 'os': None, 'conf': {'code': None, 'string': 'Title=FiveAourThe??', 'notString': None, 'titles': None, 'regexp': None, 'textOnly': None, 'optimize': None}, 'parameter': 'keyword', 'data': {'1': {'comment': '', 'matchRatio': 0.863, 'trueCode': 200, 'title': 'AND boolean-based blind - WHERE or HAVING clause', 'templatePayload': None, 'vector': 'AND [INFERENCE]', 'falseCode': 200, 'where': 1, 'payload': 'keyword=1 AND 3424=3424'}...], 'success': True, 'error': []}
```

那么如何检测 COOKIE 注入、UA 注入这些呢？下面笔者将列出 api 接口可接收的所有字段，若要检测 COOKIE 注入的话，我们只要在`@post("/scan/<taskid>/start")`接口中，传入 cookie 字段；若要检测 referer 注入的话，我们只要在`@post("/scan/<taskid>/start")`接口中，传入 referer 字段。

若要从注入点中获取数据库的版本、数据库的用户名这些，只要在`@post("/scan/<taskid>/start")`接口中，传入 getBanner 字段，并设置为 True，传入 getUsers 字段，并设置为 True。

```
crawlDepth: None
osShell: False
getUsers: False
getPasswordHashes: False
excludeSysDbs: True
ignoreTimeouts: False
regData: None
fileDest: None
prefix: None
code: None
googlePage: 1
skip: None
query: None
randomAgent: False
osPwn: False
authType: None
safeUrl: None
requestFile: None
predictOutput: False
wizard: False
stopFail: False
forms: False
uChar: None
secondReq: None
taskid: 630f50607ebf91dc
pivotColumn: None
preprocess: None
dropSetCookie: False
smart: False
paramExclude: None
risk: 1
sqlFile: None
rParam: None
getCurrentUser: False
notString: None
getRoles: False
getPrivileges: False
testParameter: None
tbl: None
charset: None
trafficFile: None
osSmb: False
level: 1
dnsDomain: None
outputDir: None
skipWaf: False
timeout: 30
firstChar: None
torPort: None
getComments: False
binaryFields: None
checkTor: False
commonTables: False
direct: None
tmpPath: None
titles: False
getSchema: False
identifyWaf: False
paramDel: None
safeReqFile: None
regKey: None
murphyRate: None
limitStart: None
crawlExclude: None
flushSession: False
loadCookies: None
csvDel: ,
offline: False
method: None
tmpDir: None
fileWrite: None
disablePrecon: False
osBof: False
testSkip: None
invalidLogical: False
getCurrentDb: False
hexConvert: False
proxyFile: None
answers: None
host: None
dependencies: False
cookie: None
proxy: None
updateAll: False
regType: None
repair: False
optimize: False
limitStop: None
search: False
shLib: None
uFrom: None
noCast: False
testFilter: None
ignoreCode: None
eta: False
csrfToken: None
threads: 1
logFile: None
os: None
col: None
skipStatic: False
proxyCred: None
verbose: 1
isDba: False
encoding: None
privEsc: False
forceDns: False
getAll: False
api: True
url: http://10.20.40.95/sql-labs/Less-4/?id=1
invalidBignum: False
regexp: None
getDbs: False
freshQueries: False
uCols: None
smokeTest: False
udfInject: False
invalidString: False
tor: False
forceSSL: False
beep: False
noEscape: False
configFile: None
scope: None
authFile: None
torType: SOCKS5
regVal: None
dummy: False
checkInternet: False
safePost: None
safeFreq: None
skipUrlEncode: False
referer: None
liveTest: False
retries: 3
extensiveFp: False
dumpTable: False
getColumns: False
batch: True
purge: False
headers: None
authCred: None
osCmd: None
suffix: None
dbmsCred: None
regDel: False
chunked: False
sitemapUrl: None
timeSec: 5
msfPath: None
dumpAll: False
fileRead: None
getHostname: False
sessionFile: None
disableColoring: True
getTables: False
listTampers: False
agent: None
webRoot: None
exclude: None
lastChar: None
string: None
dbms: None
dumpWhere: None
tamper: None
ignoreRedirects: False
hpp: False
runCase: None
delay: 0
evalCode: None
cleanup: False
csrfUrl: None
secondUrl: None
getBanner: False
profile: False
regRead: False
bulkFile: None
db: None
dumpFormat: CSV
alert: None
harFile: None
nullConnection: False
user: None
parseErrors: False
getCount: False
data: None
regAdd: False
ignoreProxy: False
database: /tmp/sqlmapipc-lI97N8
mobile: False
googleDork: None
saveConfig: None
sqlShell: False
tech: BEUSTQ
textOnly: False
cookieDel: None
commonColumns: False
keepAlive: False
```

### 总结

基于 HTTP 的接口模式用起来可能比较繁琐，但是对于程序调用接口还是很友善的。总之该模式的流程是：

1、通过GET请求[http://ip:port](http://ip:port)/task/new 这个地址，创建一个新的扫描任务；

2、通过POST请求[http://ip:port](http://ip:port)/scan//start 地址，并通过json格式提交参数，开启一个扫描；通过GET请求[http://ip:port/](http://ip:port/)scan//status 地址，即可获取指定的taskid的扫描状态。这个返回值分为两种，一种是 run 状态（扫描未完成），一种是 terminated 状态（扫描完成）；

3、扫描完成后获取扫描的结果。

# 使用 Python3 编写 sqlmapapi 调用程序 {#python3-sqlmapapi}

下面就来编写一个 sqlmapapi 调用程序，首先我们得再次明确一下流程：

1、通过 sqlmapapi.py -s -H "0.0.0.0" 开启sqlmap api的服务端。服务端启动后，在服务端命令行中会返回一个随机的admin token值，这个token值用于管理taskid（获取、清空操作），在这个流程中不需要amin token这个值，可以忽略。之后，服务端会处于一个等待客户端的状态。

2、通过GET请求[http://ip:port](http://ip:port)/task/new 这个地址，即可创建一个新的扫描任务，在响应中会返回一个随机的taskid。这个taskid在这个流程中尤为重要，因此需要通过变量存储下来，方便后面程序的调用。

3、通过POST请求[http://ip:port](http://ip:port)/scan//start 地址，并通过json格式提交参数\(待扫描的HTTP数据包、若存在注入是否获取当前数据库用户名\)，即可开启一个扫描任务，该请求会返回一个enginedid。

4、通过GET请求[http://ip:port/](http://ip:port/)scan//status 地址，即可获取指定的taskid的扫描状态。这个返回值分为两种，一种是run状态（扫描未完成），一种是terminated状态（扫描完成）。

5、判断扫描状态，如果扫描未完成，再次请求[http://ip:port/](http://ip:port/)scan//status 地址 ，直到扫描完成。

6、扫描完成后获取扫描的结果，是否是SQL注入，若不存在SQL注入，data字段为空，若存在SQL注入，则会返回数据库类型、payload等等。

明确了流程后，为了可维护性好和 main.py 文件代码量少，笔者首先是写了一个类，代码如下：

```py
#!/usr/bin/python
# -*- coding:utf-8 -*-
# wirter:En_dust
import requests
import json
import time

class Client():
    def __init__(self,server_ip,server_port,admin_token="",taskid="",filepath=None):
        self.server = "http://" + server_ip + ":" + server_port
        self.admin_token = admin_token
        self.taskid = taskid
        self.filepath = ""
        self.status = ""
        self.scan_start_time = ""
        self.scan_end_time = ""
        self.engineid=""
        self.headers = {'Content-Type': 'application/json'}



    def create_new_task(self):
        '''创建一个新的任务，创建成功返回taskid'''
        r = requests.get("%s/task/new"%(self.server))
        self.taskid = r.json()['taskid']
        if self.taskid != "":
            return self.taskid
        else:
            print("创建任务失败!")
            return None

    def set_task_options(self,url):
        '''设置任务扫描的url等'''
        self.filepath = url



    def start_target_scan(self,url):
        '''开始扫描的方法,成功开启扫描返回True，开始扫描失败返回False'''
        r = requests.post(self.server + '/scan/' + self.taskid + '/start',
                      data=json.dumps({'url':url,'getCurrentUser':True,'getBanner':True,'getCurrentDb':True}),
                      headers=self.headers)
        if r.json()['success']:
            self.scan_start_time = time.time()
            #print(r.json())
            #print(r.json()['engineid'])
            return r.json()['engineid']
        else:
            #print(r.json())
            return None

    def get_scan_status(self):
        '''获取扫描状态的方法,扫描完成返回True，正在扫描返回False'''
        self.status = json.loads(requests.get(self.server + '/scan/' + self.taskid + '/status').text)['status']
        if self.status == 'terminated':
            self.scan_end_time = time.time()
            #print("扫描完成!")
            return True
        elif self.status == 'running':
            #print("Running")
            return False
        else:
            #print("未知错误！")
            self.status = False



    def get_result(self):
        '''获取扫描结果的方法，存在SQL注入返回payload和注入类型等，不存在SQL注入返回空'''
        if(self.status):
            r = requests.get(self.server + '/scan/' + self.taskid + '/data')
            if (r.json()['data']):
                return r.json()['data']
            else:
                return None

    def get_all_task_list(self):
        '''获取所有任务列表'''
        r = requests.get(self.server + '/admin/' + self.admin_token + "/list")
        if r.json()['success']:
            #print(r.json()['tasks'])
            return r.json()['tasks']
        else:
            return None

    def del_a_task(self,taskid):
        '''删除一个任务'''
        r = requests.get(self.server + '/task/' + taskid + '/delete')
        if r.json()['success']:
            return True
        else:
            return False

    def stop_a_scan(self,taskid):
        '''停止一个扫描任务'''
        r = requests.get(self.server + '/scan/' + taskid + '/stop')
        if r.json()['success']:
            return True
        else:
            return False

    def flush_all_tasks(self):
        '''清空所有任务'''
        r =requests.get(self.server + '/admin/' + self.admin_token + "/flush")
        if r.json()['success']:
            return True
        else:
            return False

    def get_scan_log(self):
        '''获取log'''
        r = requests.get(self.server + '/scan/' + self.taskid + '/log')
        return r.json()
```

main.py

```py
#!/usr/bin/python
# -*- coding:utf-8 -*-
# wirter:En_dust
from Service import Client
import time
from threading import Thread

def main():
    '''实例化Client对象时需要传递sqlmap api 服务端的ip、port、admin_token和HTTP包的绝对路径'''
    print("————————————————Start Working！—————————————————")
    target = input("url:")
    task1 = Thread(target=set_start_get_result,args=(target,))
    task1.start()



def time_deal(mytime):
     first_deal_time = time.localtime(mytime)
     second_deal_time = time.strftime("%Y-%m-%d %H:%M:%S", first_deal_time)
     return  second_deal_time


def set_start_get_result(url):
    #/home/cheng/Desktop/sqldump/1.txt
    current_taskid =  my_scan.create_new_task()
    print("taskid: " + str(current_taskid))
    my_scan.set_task_options(url=url)
    print("扫描id:" + str(my_scan.start_target_scan(url=url)))
    print("扫描开始时间：" + str(time_deal(my_scan.scan_start_time)))
    while True:
        if my_scan.get_scan_status() == True:
            print(my_scan.get_result())
            print("当前数据库:" + str(my_scan.get_result()[-1]['value']))
            print("当前数据库用户名:" + str(my_scan.get_result()[-2]['value']))
            print("数据库版本:" + str(my_scan.get_result()[-3]['value']))
            print("扫描结束时间：" + str(time_deal(my_scan.scan_end_time)))
            print("扫描日志：\n" + str(my_scan.get_scan_log()))
            break




if __name__ == '__main__':
    my_scan = Client("127.0.0.1", "8775", "c88927c30abb1ef6ea78cb81ac7ac6b0")
    main()
```

Github 地址：

[https://github.com/FiveAourThe/sqlmap\_api\_demo](https://github.com/FiveAourThe/sqlmap_api_demo)

原文地址：[https://paper.seebug.org/940/](https://paper.seebug.org/940/)

