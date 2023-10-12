# 故障处理方法

## 故障处理三板斧

1. 精准定位
2. 及时重启
3. 谨慎切换

补充步骤

- 沟通协调：
联系值班经理，系统负责人，部门经理
- 保留现场：
在重启前，收集网络、数据库、中间件、bpc、线程数、连接池、堆栈等关键信息。
- 事后记录：
梳理时间线，整理故障报告，总结后续优化、做的好的地方和不足

## 应用隔离

- F5 BIG-IP 切流量（详细步骤见 科技知识库-《F5负载均衡切换操作步骤》）
- 水星微服务-星云空间站隔离
- nginx 负载均衡
- 参数调整

## 应用重启

`/home/cloud/APPLICATION_NAME+ENV/APPLICATION_NAME/go.sh`

```shell
#!usr/bin/env bash
projectName=$1
target=$2
port=$3
health_url="http://127.0.0.1:$port/actuator/health"
up_url="http://127.0.0.1:$port/actuator/service-registry?status=UP"
down_url="http://127.0.0.1:$port/actuator/service-registry?status=DOWN"
current=`date "+%Y-%m-%d-%H%M%S"`

skywalking_agent_dir="./skywalking/agent-msp/agent8.4"
skywalking_service_ip=$4
service_name=$1

#something...
```

Java 应用手动重启：
```shell
nohup java  -jar JVM_PARAMETERS LOG_PARAMETERS MICRO_SERVICES_PARAMETERS 
		APPLICATON—VERSION.jar &
```

- nohup 指忽略所有挂断（SIGHUP）信号
- & 指在后台运行

## 版本回退

在科管下载历史更新包，或在自动化部署的备份目录找到历史包，替换现有包后启动。

## 修改配置

- Apollo 修改，如之前未使用过 Apollo 需要先赋权；
- vim 直接进入 jar 包修改

## Linux 基础

- man / which
- 用户权限（Access Permission）

## 磁盘空间不足

系统管理员会对系统磁盘空间状况进行日常巡检，如磁盘使用率超过75%，会在科管上对系统负责人发起异常巡检流程。

如磁盘使用率达到80%以上，触发8级故障。

### 查看硬盘

查看服务器硬盘使用情况：

```shell
df -h
```

查看服务器硬盘使用分布：
```shell
du -h --max-depth=1
```
查看目录下有多少文件：
```shell
ll | wc -l
```

### 删除文件

删除10GB内文件使用 nbrm 命令，nbrm 根据用户权限提供相应的功能：
- 普通用户：放弃删除 / 远端备份后删除
- 高权用户：放弃删除 / 远端备份后删除 / 本地备份后删除 /直接删除
交互选择时须二次确认选择项，两次选择一致，删除功能才生效；若前后选择不一致，需重新选择

!> 由于nbrm命令受网络限制，远程备份速度较慢，加上备份盘容量有限，如果删除超过10GB文件，建议在确定文件可被清除后，使用rm命令清除。

- 将rm命令写在脚本里，然后执行脚本。注意在编写应用变更方案时，要加上给脚本赋权的语句，默认权限无法执行脚本。

- 按命名匹配删除：`rm -rf /folder/2019*`
- 按最后一次修改日期删除：`find /path/file* -mtime +5 -exec rm {} \;`

#### 删除海量文件

如果要删除文件夹内的部分文件，且文件夹中存在海量文件（数十万），执行上述命令会报参数过长错误，有几种做法：

- `find /path/dir/2019* -delete`
- 创建空文件夹 deletetmp，然后`rsync -a --delete deletetmp /path/dir`
- 在脚本里面写循环删除，一次删除几条

#### 删除后空间不释放

rm 命令只是把链接解除，进程仍可占有已删除的文件，所以并不释放磁盘空间。使用 `lsof | grep deleted` 命令查看被删除但其实未真正释放的文件，找到进程号以后杀掉进程。

后续配置定期清理策略，有些中间件自带清理配置，优先修改自带配置，再考虑编写 Cron 脚本。

## CPU过高

CPU 过高的原因可能包括业务逻辑问题（死循环）、循环以及上下文切换过多等。

需了解服务器的核心数量(`/proc/cpuinfo`)，对于几十上百核CPU的服务器来说，CPU占用百分之几百可能是正常现象。

- `top` 命令，然后按 P，查看 CPU 使用率高的进程；
- `top -p pid -H` 发现问题线程 pid；
- `printf '%x\n' pid` 将 pid 转换成16进制得到 nid；
- 在 jstack 中找到相应的堆栈信息 `jstack pid | grep 'nid' -C5 –color`，主要关注 WAITING 、 TIMED_WAITING状态。

### 查看GC情况

jstat -gcutil pid 1000

- 1000：采样间隔(ms)，
- S0C/S1C、S0U/S1U：两个Survivor区， C for Capacity，U for Used，
- EC/EU：Eden区，
- OC/OU：老年代，
- MC/MU：元空间，
- YGC/YGT：YoungGc， C for Count，T for Total，
- FGC/FGCT：FullGc

## 内存不足

使用 `free -m` 查看内存使用情况，注意关注 available 这列，而不是 free。

![free 命令结果](../_media/ops_memory.jpeg ':size=90%')

查看是否有 heapdump 文件生产，如没有，使用 `jmap -dump:live,format=b,file=heap.hprof <pid>` 生产 heapdump 文件
，使用 mat(Eclipse Memory Analysis Tools) 导入 dump 文件进行分析。

在 Eclipse/MemoryAnalyzer.ini 修改启动参数，在 info.plist 修改 java 路径。

### OutOfMemory

```shell
Exception in thread "main" java.lang.OutOfMemoryError: unable to create new native thread
```

没有足够的内存空间给线程分配 Java 栈，基本上是线程池代码写的有问题，比如说忘记 shutdown ，所以说应该首先从代码层面来寻找问题，使用 jstack 或者 jmap。如果一切都正常，JVM 方面可以通过指定 Xss 来减少单个 thread stack 的大小。另外也可以在系统层面，可以通过修改 /etc/security/limits.conf的 nofile 和 nproc 来增大 os 对线程的限制：

![nofile & nproc](../_media/ops_limits.png ':size=40%')
```shell
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
```

堆内存占用已达到 -Xmx 设置的最大值。解决思路仍然是先应该在代码中找问题，怀疑存在内存泄漏，通过jstack 和 jmap 定位。如果一切正常，通过调整 Xmx 的值来扩大内存

### Stack Overflow

```shell
Exception in thread "main" java.lang.StackOverflowError
```

线程栈需要的内存大于 Xss 值，同样也是先进行排查，参数方面通过 Xss 来调整，但调整的太大可能又会引起OOM。

## 前端页面报错

尝试在前端复现错误，使用 F12 查看报错接口信息。

特殊场景：点击打开一个新的页面，报错出现在新的页面，来不及点 F12 查看
打开开发者工具，进设置，在 Global 栏选上 Auto-open DevTools for popups。

## 日志查询

- 在对应日志界面查找日志：大数据运维平台，Logstash 等，注意搜索时可能会触发分词
- 在服务器上查询日志

## VIM

Vim的设计以大多数时间都花在阅读、浏览和进行少量编辑改动为基础，因此它具有多种操作模式：

- 正常模式：在文件中四处移动光标进行修改
- 插入模式：插入文本
- 替换模式：替换文本
- 可视化（一般，行，块）模式：选中文本块
- 命令模式：用于执行命令

