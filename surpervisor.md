### 检查进程：

ps aux | grep supervisord

### 查看运行状态

supervisorctl status



|               命令               |                             说明                             |
| :------------------------------: | :----------------------------------------------------------: |
|  supervisorctl stop programxxx   |    停止某一进程（programxxx）为[program:beepkg]里配置的值    |
|  supervisorctl start programxxx  |                         启动某一进程                         |
| supervisorctl restart programxxx |                         重启某个进程                         |
|      supervisorctl  status       |                         查看进程状态                         |
|  supervisorctl stop groupworker  | 重启所有属于名为 groupworker 这个分组的进程(start,restart 同理) |
|      supervisorctl stop all      |                         停止全部进程                         |
|       supervisorctl reload       | 载入最新的配置文件，停止原有进程并按新的配置启动、管理所有进程 |
|       supervisorctl update       | 根据最新的配置文件，启动新配置或有改动的进程，配置没有改动的进程不会受影响而重启 |

### 自动重启配置文件(.conf)

```python
[program:your_program_name]#设置进程的名称，使用supervisorctl来管理进程时需要使用该进程名，这里的进程名是`your_program_name`
directory = /home/...#执行command之前，先切换到工作目录
command = python3  ... .py
autostart = true ;#如果设置为true，当supervisord启动的时候，进程会自动重启
user = root #使用root用户来启动该进程
autorestart = true #程序崩溃时自动重启，重启次数是有限制的，默认为3次
startsecs = 5 #启动5秒之后没有异常退出，就当做已经正常启动了
redirect_stderr = ture #错误日志重定向到标准输出
priority = 999 #程序操作的优先级，例如在start all/ stop  all ,高优先级的程序会先关闭和重启

```

### 使用supervisorctl管理进程

* 停止某一个进程，program_name为[program:x] 里的x
  * supervisorctl stop program_name
* 启动某个进程
  * supervisorctl start program_name
* 重启某个进程
  * supervisorctl restart program_name
* 停止全部进程。注：start . restart  ,stop 都不会载入最新的配置文件
  * supervisorctl stop all
* 载入最新的配置文件，停止原有进程并按新的配置启动，管理所有进程
  * supervisorctl reload
* 根据最新的配置文件，启动新配置或有改动的进程，配置没有改动的进程不会受影响而重启
  * supervisorctl update

### clush命令

| 命令                       | 说明                                                         | 举例                                              |
| -------------------------- | ------------------------------------------------------------ | ------------------------------------------------- |
| clush -a 全部              | 等于 clush -g  all                                           |                                                   |
| clush -g 指定组            |                                                              |                                                   |
| clush -w 操作主机名字      | 多个主机用逗号分开                                           |                                                   |
| clush  -g 组名 -c  -- dest | --dest 前面表示本地要复制的文件或目录路径，后面表示远程机器的存放路径 | clush -g 组名  -c  /root/test.file --dest  /root/ |

### supervisor管理nginx

- 这个命令默认是后台启动，但是supervisor不能监控后台程序，所以supervisor就一直执行这个命令
  - command = /usr/local/bin/nginx
- 加上-g  'daemon off' 这个参数可解决这个问题，这个参数的意思是在前台运行
  - command = /usr/local/bin/nginx -g  'daemon off'
- nginx 默认运行状态是后台运行，supervisord 不能管理此类程序需要，需要修改nginx主配置文件。在nginx最外层加入daemon off