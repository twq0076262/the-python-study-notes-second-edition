#第三部分 扩展库
#A. Fabric
通过 SSH 进行软件部署或系统管理。

**设置**

在连接目标主机之前，必须提供足够的配置信息。

- env.hosts: 目标主机列表。格式: ip, user@ip, user@ip:port。
- env.host_string: 单机，格式同 hosts。
- env.roledefs: 按角色定义主机列表。格式: {name:[host, ...]}。
- env.passwords: 主机密码字典。格式: {host: password}，和 hosts 中保持格式一致。
- env.user: 默认用户名。
- env.port: 默认端口。
- env.password: 默认密码。
- env.parallel: 是否并行执行任务。
- env.skip_bad_hosts: 是否跳过无法连接的主机。
- env.timeout: 连接超时，默认 10 秒。
- env.warn_only: 出错时是否仅显示警告信息。默认 False 终止任务。

**任务**

任务就是些普通函数，可以直接用 execute 执行，或在命令行调用。

- 默认: 所有 env.hosts 或 host_string。
- 主机: host 单主机，host 多主机列表。
- 角色: role 单个角色，roles 多个角色名列表。

也可用装饰器设置这些参数。另外，最好显式关闭连接。

```
from fabric.api import *
from fabric.network import *

def rcmd(s): run(s)

env.user = "root"
env.password = "123456"
env.hosts = ["192.168.1.1", "192.168.1.2", "192.168.1.3"]
env.roledefs = {"A": ["192.168.1.1", "192.168.1.2"], "B":["192.168.1.3"] }

try:
	execute(rcmd, "uname -a", roles = ["A", "B"])
finally:
	disconnect_all()
```

**颜色**

fabric.colors 提供了一些颜色包装函数，可配合 print 显示一些需要特别注意的信息。

```
from fabric.colors import *
print(green("This text is green"))
```

**上下文**

fabric.context_managers 为命令提供区域设置。

- cd: 切换主机工作目录。
- lcd: 切换本地目录。
- hide: 隐藏输出信息。
- quiet: 安静模式，hide('everything'), warn_only=True。
- path: 修改 PATH 环境变量，可选 append 或 prepend。

切换到合适的工作目录，减少命令行输入。

```
with cd("/etc"):
run("pwd")   # /etc
with cd ("init.d"):
run("pwd")   # /etc/init.d
```

**操作**

fabric.operations 包含用于任务的一些命令。

- get: 下载文件。
- put: 上传文件。
- run: 运行远程命令。
- sudo: 以超级用户执行命令。
- local: 运行本地命令。
- prompt: 用户输入。
- open_shell: 远程交互式环境。
- reboot: 重启主机。

最常用的是 run 命令，它执行远程命令，返回输出结果字符串。该字符串对象还有 failed、succeeded、 return_code、command、real_command 等属性用来检查运行结果。

```
out = run("uname -a")
print out.succeeded, out.failed, out.return_code
print out.command, out.real_command
```

**其他**

fabric.utils 提供了一些辅助操作。

- abort: 引发异常，终止任务执行。
- warn: 显示警告信息，但不终止。
- indent: 获取一个缩进字符串。
- fastprint, puts: 显示信息。

fabric.contrib.console 提供 confirm 函数，让用户输入 [Y/n] 确认信息。

```
if confirm("Continue", default = False):
	out = run("uname -a")
```

fabric.contrib.files 提供了远程文件操作功能。

- append: 添加信息。
- comment: 按条件注释掉某些内容。
- contains: 是否包含特定内容。
- exists: 路径是否存在。

问题

(1) 对后台运行命令 nohup 支持不好，可以考虑用 screen、pexpect 等代替。

```
$ screen -d -m -S <session_name> [cmd args]; sleep 5
```

注意用 sleep 暂停，避免 screen 尚未运行，会话就结束。

(2) 用 open_shell 进入交互模式，Ctrl+C 导致 fabric 进程退出，而不是远程进程。