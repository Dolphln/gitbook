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

命令成功后，在命令行中会返回一些信息。以下命令大概的意思是 api 服务端在本地 8775 端口上运行，admin token 为

ff053d6517b86f06dec73bda814a70a1，IPC 数据库的位置在/var/folders/fz/16wqcnkd6477gz4wcr\_63xcc0000gn/T/sqlmapipc-N0m7P7，api 服务端已经和 IPC 数据库连接上了，正在使用 bottle 框架 wsgiref 标准接口。

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

```
? ? response.status = 401
? ? return response
```

#### @get\("/task/new"\) {#gettasknew}

该接口用于创建一个新的任务，使用后会返回一个随机的 taskid。 具体代码如下：

```py
def task_new():
    """
    Create a new task
    """
    taskid = hexencode(os.urandom(8))
    remote_addr = request.remote_addr
    DataStore.tasks[taskid] = Task(taskid, remote_addr)
    logger.debug("Created new task: '%s'" % taskid)
    return jsonize({"success": True, "taskid": taskid})
```

下图是调用该接口的截图

![](/assets/sqlmap-10.png)图10

#### @get\("/task//delete"\) {#gettaskdelete}

该接口用于删除 taskid。在调用时指定 taskid，不指定 taskid 会有问题。 具体代码如下：

```py
def task_delete(taskid):
    """
    Delete an existing task
    """
    if taskid in DataStore.tasks:
        DataStore.tasks.pop(taskid)
        logger.debug("(%s) Deleted task" % taskid)
        return jsonize({"success": True})
    else:
        response.status = 404
        logger.warning("[%s] Non-existing task ID provided to task_delete()" % taskid)
        return jsonize({"success": False, "message": "Non-existing task ID"})
```

下图是调用该接口的截图：![](/assets/sqlmap-11.png)

#### @get\("/admin/list"\)/@get\("/admin//list"\) {#getadminlistgetadminlist}

该接口用于返回所有 taskid。在调用时指定 taskid，不指定 taskid 会有问题。 具体代码如下：

```py
def task_list(token=None):
    """
    Pull task list
    """
    tasks = {}
    for key in DataStore.tasks:
        if is_admin(token) or DataStore.tasks[key].remote_addr == request.remote_addr:
            tasks[key] = dejsonize(scan_status(key))["status"]
    logger.debug("(%s) Listed task pool (%s)" % (token, "admin" if is_admin(token) else request.remote_addr))
    return jsonize({"success": True, "tasks": tasks, "tasks_num": len(tasks)})
```

下图是调用该接口的截图：![](/assets/sqlmap-12.png)

#### @get\("/admin/flush"\)/@get\("/admin//flush"\) {#getadminflushgetadminflush}

该接口用于删除所有任务。在调用时指定admin token，不指定admin token可能会有问题。 具体代码如下：

```py
def task_flush(token=None):
    """
    Flush task spool (delete all tasks)
    """
    for key in list(DataStore.tasks):
        if is_admin(token) or DataStore.tasks[key].remote_addr == request.remote_addr:
            DataStore.tasks[key].engine_kill()
            del DataStore.tasks[key]
    logger.debug("(%s) Flushed task pool (%s)" % (token, "admin" if is_admin(token) else request.remote_addr))
    return jsonize({"success": True})
```

下图是调用该接口的截图：![](/assets/sqlmap-13.png)13

#### @get\("/option//list"\) {#getoptionlist}

该接口可获取特定任务ID的列表选项，调用时请指定taskid，不然会出现问题。 具体代码如下：

```py
def option_list(taskid):
    """
    List options for a certain task ID
    """
    if taskid not in DataStore.tasks:
        logger.warning("[%s] Invalid task ID provided to option_list()" % taskid)
        return jsonize({"success": False, "message": "Invalid task ID"})
    logger.debug("(%s) Listed task options" % taskid)
    return jsonize({"success": True, "options": DataStore.tasks[taskid].get_options()})
```

下图是调用该接口的截图：

![](/assets/sqlmap-14.png)

#### @post\("/option//get"\) {#postoptionget}

该接口可获取特定任务ID的选项值，调用时请指定taskid，不然会出现问题。 具体代码如下：

```py
def option_get(taskid):
    """
    Get value of option(s) for a certain task ID
    """
    if taskid not in DataStore.tasks:
        logger.warning("[%s] Invalid task ID provided to option_get()" % taskid)
        return jsonize({"success": False, "message": "Invalid task ID"})
    options = request.json or []
    results = {}
    for option in options:
        if option in DataStore.tasks[taskid].options:
            results[option] = DataStore.tasks[taskid].options[option]
        else:
            logger.debug("(%s) Requested value for unknown option '%s'" % (taskid, option))
            return jsonize({"success": False, "message": "Unknown option '%s'" % option})
    logger.debug("(%s) Retrieved values for option(s) '%s'" % (taskid, ",".join(options)))
    return jsonize({"success": True, "options": results})
```

下图是调用该接口的截图：![](/assets/sqlmap-15.png)

#### @post\("/option//set"\) {#postoptionset}

该接口为特定任务 ID 设置选项值，调用时请指定 taskid，不然会出现问题。 具体代码如下：

```py
def option_set(taskid):
    """
    Set value of option(s) for a certain task ID
    """
    if taskid not in DataStore.tasks:
        logger.warning("[%s] Invalid task ID provided to option_set()" % taskid)
        return jsonize({"success": False, "message": "Invalid task ID"})
    if request.json is None:
        logger.warning("[%s] Invalid JSON options provided to option_set()" % taskid)
        return jsonize({"success": False, "message": "Invalid JSON options"})
    for option, value in request.json.items():
        DataStore.tasks[taskid].set_option(option, value)
    logger.debug("(%s) Requested to set options" % taskid)
    return jsonize({"success": True})
```

下图是调用该接口的截图：16![](/assets/sqlmap-16.png)

#### @post\("/scan//start"\) {#postscanstart}

该接口定义开始扫描特定任务，调用时请指定 taskid，不然会出现问题。 具体代码如下:

```py
def scan_start(taskid):
    """
    Launch a scan
    """
    if taskid not in DataStore.tasks:
        logger.warning("[%s] Invalid task ID provided to scan_start()" % taskid)
        return jsonize({"success": False, "message": "Invalid task ID"})
    if request.json is None:
        logger.warning("[%s] Invalid JSON options provided to scan_start()" % taskid)
        return jsonize({"success": False, "message": "Invalid JSON options"})
    # Initialize sqlmap engine's options with user's provided options, if any
    for option, value in request.json.items():
        DataStore.tasks[taskid].set_option(option, value)
    # Launch sqlmap engine in a separate process
    DataStore.tasks[taskid].engine_start()
    logger.debug("(%s) Started scan" % taskid)
    return jsonize({"success": True, "engineid": DataStore.tasks[taskid].engine_get_id()})
```

下图是调用该接口的截图：![](/assets/sqlmap-17.png)

#### @get\("/scan//stop"\) {#getscanstop}

该接口定义停止扫描特定任务，调用时请指定 taskid，不然会出现问题。 具体代码如下：

```py
def scan_stop(taskid):
    """
    Stop a scan
    """
    if (taskid not in DataStore.tasks or DataStore.tasks[taskid].engine_process() is None or DataStore.tasks[taskid].engine_has_terminated()):
        logger.warning("[%s] Invalid task ID provided to scan_stop()" % taskid)
        return jsonize({"success": False, "message": "Invalid task ID"})
    DataStore.tasks[taskid].engine_stop()
    logger.debug("(%s) Stopped scan" % taskid)
    return jsonize({"success": True})
```

下图是调用该接口的截图：![](/assets/sqlmap-18.png)

#### @get\("/scan//kill"\) {#getscankill}

该接口可杀死特定任务，需要指定 taskid，不然会出现问题。 具体代码如下：

```py
def scan_kill(taskid):
    """
    Kill a scan
    """
    if (taskid not in DataStore.tasks or DataStore.tasks[taskid].engine_process() is None or DataStore.tasks[taskid].engine_has_terminated()):
        logger.warning("[%s] Invalid task ID provided to scan_kill()" % taskid)
        return jsonize({"success": False, "message": "Invalid task ID"})
    DataStore.tasks[taskid].engine_kill()
    logger.debug("(%s) Killed scan" % taskid)
    return jsonize({"success": True})
```

####  {#getscanstatus}

#### @get\("/scan//status"\) {#getscanstatus}

该接口可查询扫描状态，调用时请指定 taskid，不然会出现问题。 具体代码如下：

```py
def scan_status(taskid):
    """
    Returns status of a scan
    """
    if taskid not in DataStore.tasks:    
        logger.warning("[%s] Invalid task ID provided to scan_status()" % taskid)
        return jsonize({"success": False, "message": "Invalid task ID"})
    if DataStore.tasks[taskid].engine_process() is None:
        status = "not running"
    else:
        status = "terminated" if DataStore.tasks[taskid].engine_has_terminated() is True else "running"
    logger.debug("(%s) Retrieved scan status" % taskid)
    return jsonize({
        "success": True,
        "status": status,
        "returncode": DataStore.tasks[taskid].engine_get_returncode()
    })
```

下图是调用该接口的截图：![](/assets/sqlmap-19.png)

#### @get\("/scan//data"\) {#getscandata}

该接口可获得到扫描结果，调用时请指定 taskid，不然会出现问题。 具体代码如下：

```py
def scan_data(taskid):
    """
    Retrieve the data of a scan
    """
    json_data_message = list()
    json_errors_message = list()
    if taskid not in DataStore.taskslogger.warning("[%s] Invalid task ID provided to scan_data()" % taskid)
        return jsonize({"success": False, "message": "Invalid task ID"})
# Read all data from the IPC database for the taskid
    for status, content_type, value in DataStore.current_db.execute("SELECT status, content_type, value FROM data WHERE taskid = ? ORDER BY id ASC", (taskid,)):
        json_data_message.append({"status": status, "type": content_type, "value": dejsonize(value)})
        # Read all error messages from the IPC database
    for error in DataStore.current_db.execute("SELECT error FROM errors WHERE taskid = ? ORDER BY id ASC", (taskid,)):
        json_errors_message.append(error)
        logger.debug("(%s) Retrieved scan data and error messages" % taskid)
    return jsonize({"success": True, "data": json_data_message, "error": json_errors_message})
```



