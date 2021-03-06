# AgentSmith-HIDS

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;--The name of this project was inspired by the movie - The Matrix

[![License](https://img.shields.io/badge/License-GPL%20v2-blue.svg)](https://github.com/DianrongSecurity/AgentSmith-HIDS/blob/master/LICENSE)

English | [简体中文](README-zh_CN.md)




### About AgentSmith-HIDS

Technically, AgentSmith-HIDS is not a Host-based Intrusion Detection System (HIDS) due to lack of rule engine and detection function. However, it can be used as a high performance 'Host Information Collect Agent' as part of your own HIDS solution.
The comprehensiveness of information which can be collected by this agent was one of the most important metrics during developing this project, hence it was built to function in the kernel stack and achieve huge advantage comparing to those function in user stack, such as:

* **Better performance**, Information needed are collected in kernel stack to avoid additional supplement actions such as traversal of '/proc'; and to enhance the performance of data transportation, data collected is transferred via shared ram instead of netlink.
* **Hard to be bypassed**, Information collection was powered by specifically designed kernel drive, makes it almost impossible to bypass the detection for malicious software like rootkit, which can deliberately hide themselves.
* **Easy to be integrated**，The AgentSmith-HIDS was built to integrate with other applications and can be used not only as security tool but also a good monitoring tool, or even a good detector of your assets. The agent is capable of collecting the users, files, processes and internet connections for you, so let's imagine when you integrate it with CMDB, you could get a comprehensive map consists of your network, host, container and business (even dependencies). What if you also have a Database audit tool at hand? The map can be extended to contain the relationship between your DB, DB User, tables, fields, applications, network, host and containers etc. Thinking of the possibility of integration with network intrusion detection system and/or threat intelligence etc., higher traceability could also be achieved. It just never gets old.
* **Kernel stack + User stack**，AgentSmith-HIDS also provide user stack module, to further extend the functionality when working with kernel stack module.



### Major abilities of AgentSmith-HIDS：

* Kernel stack module hooks **execve, connect, process inject, create file, DNS query, load LKM** behaviors via Kprobe，and is also capable of monitoring containers by being compatible with Linux namespace.
* User stack module utilize built in detection functions including queries of **User List**，**Listening ports list**，**System RPM list**，**Schedule jobs**
* **AntiRootkit**，From: [Tyton](https://github.com/nbulischeck/tyton) ,for now add **PROC_FILE_HOOK**，**SYSCALL_HOOK**，**LKM_HIDDEN**，**INTERRUPTS_HOOK** feature，but only wark on Kernel > 3.10.
* Cred Change monitoring (sudo/su/sshd except)



### About the compatibility with Kernel version

* Kernel > 2.6.25
* AntiRootKit > 3.10


### About the compatibility with Containers

| Source | Nodename       |
| ------ | -------------- |
| Host   | hostname       |
| Docker | container name |
| k8s    | pod name       |




### Composition of AgentSmith-HIDS

* **Kernel stack module (LKM)**
    Hook key functions via Kprobe to capture information needed.
* **User stack module** 
    Collect data capatured by kernel stack module, perform necessary process and send it to Kafka; 
    Keep sending heartbeat packet to server so process integrity can be identitied; 
    Execute commands received from server.
* **Agent Server**(Optional)
    Send commands to user stack module and monitoring the status of user stack module.

### Execve Hook

Achieved by hooking **sys_execve()/sys_execveat()**, example:

```json
{
    "uid":"0",
    "data_type":"59",
    "run_path":"/root/AgentSmith-HIDS/agent/target/release",
    "exe":"/usr/bin/ls",
    "argv":"ls --color=auto --indicator-style=classify ",
    "pid":"6265",
    "ppid":"1941",
    "pgid":"6265",
    "tgid":"6265",
    "comm":"fish",
    "nodename":"test",
    "stdin":"/dev/pts/0",
    "stdout":"/dev/pts/0",
    "sessionid":"1",
    "user":"root",
    "time":"1575721900051",
    "local_ip":"192.168.165.153",
    "hostname":"test",
    "exe_md5":"a0c32dd6d3bc4d364380e2e65fe9ac64"
}
```



### Connect Hook

Achieved by hooking **sys_connect()**, example:

```json
{
    "uid":"0",
    "data_type":"42",
    "sa_family":"4",
    "fd":"4",
    "dport":"1025",
    "dip":"180.101.49.11",
    "exe":"/usr/bin/ping",
    "pid":"6294",
    "ppid":"1941",
    "pgid":"6294",
    "tgid":"6294",
    "comm":"ping",
    "nodename":"test",
    "sip":"192.168.165.153",
    "sport":"45524",
    "res":"0",
    "sessionid":"1",
    "user":"root",
    "time":"1575721921240",
    "local_ip":"192.168.165.153",
    "hostname":"test",
    "exe_md5":"735ae70b4ceb8707acc40bc5a3d06e04"
}
```



### DNS Query Hook

Achieved by hooking **sys_recvfrom()**, example:

```json
{
    "uid":"0",
    "data_type":"601",
    "sa_family":"4",
    "fd":"4",
    "sport":"53",
    "sip":"192.168.165.2",
    "exe":"/usr/bin/ping",
    "pid":"6294",
    "ppid":"1941",
    "pgid":"6294",
    "tgid":"6294",
    "comm":"ping",
    "nodename":"test",
    "dip":"192.168.165.153",
    "dport":"53178",
    "qr":"1",
    "opcode":"0",
    "rcode":"0",
    "query":"www.baidu.com",
    "sessionid":"1",
    "user":"root",
    "time":"1575721921240",
    "local_ip":"192.168.165.153",
    "hostname":"test",
    "exe_md5":"39c45487a85e26ce5755a893f7e88293"
}
```



### Create File Hook

Achieved by hooking **security_inode_create()**, example:

```json
{
    "uid":"0",
    "data_type":"602",
    "exe":"/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.232.b09-0.el7_7.x86_64/jre/bin/java",
    "file_path":"/tmp/kafka-logs/replication-offset-checkpoint.tmp",
    "pid":"3341",
    "ppid":"1",
    "pgid":"2657",
    "tgid":"2659",
    "comm":"kafka-scheduler",
    "nodename":"test",
    "sessionid":"3",
    "user":"root",
    "time":"1575721984257",
    "local_ip":"192.168.165.153",
    "hostname":"test",
    "exe_md5":"215be70a38c3a2e14e09d637c85d5311",
    "create_file_md5":"d41d8cd98f00b204e9800998ecf8427e"
}
```



### Process Inject Hook

Achieved by hooking **sys_ptrace()**, example:

```json
{
    "uid":"0",
    "data_type":"101",
    "ptrace_request":"4",
    "target_pid":"7402",
    "addr":"00007ffe13011ee6",
    "data":"-a",
    "exe":"/root/ptrace/ptrace",
    "pid":"7401",
    "ppid":"1941",
    "pgid":"7401",
    "tgid":"7401",
    "comm":"ptrace",
    "nodename":"test",
    "sessionid":"1",
    "user":"root",
    "time":"1575722717065",
    "local_ip":"192.168.165.153",
    "hostname":"test",
    "exe_md5":"863293f9fcf1af7afe5797a4b6b7aa0a"
}
```


### Load LKM File Hook

Achieved by hooking **load_module()**, example:

```json
{
    "uid":"0",
    "data_type":"603",
    "exe":"/usr/bin/kmod",
    "lkm_file":"/root/ptrace/ptrace",
    "pid":"29461",
    "ppid":"9766",
    "pgid":"29461",
    "tgid":"29461",
    "comm":"insmod",
    "nodename":"test",
    "sessionid":"13",
    "user":"root",
    "time":"1577212873791",
    "local_ip":"192.168.165.152",
    "hostname":"test",
    "exe_md5":"0010433ab9105d666b044779f36d6d1e",
    "load_file_md5":"863293f9fcf1af7afe5797a4b6b7aa0a"
}
```


### Cred Change Hook

Achieved by Hook **commit_creds()**，example：

```json
{
    "uid":"0",
    "data_type":"604",
    "exe":"/tmp/tt",
    "pid":"27737",
    "ppid":"26865",
    "pgid":"27737",
    "tgid":"27737",
    "comm":"tt",
    "old_uid":"1000",
    "nodename":"test",
    "sessionid":"42",
    "user":"root",
    "time":"1578396197131",
    "local_ip":"192.168.165.152",
    "hostname":"test",
    "exe_md5":"d99a695d2dc4b5099383f30964689c55"
}
```


### PROC File Hook Alert
```json
{
    "uid":"-1",
    "data_type":"700",
    "module_name":"autoipv6",
    "hidden":"0",
    "time":"1578384987766",
    "local_ip":"192.168.165.152",
    "hostname":"test"
}
```


### Syscall Hook Alert
```json
{
    "uid":"-1",
    "data_type":"701",
    "module_name":"diamorphine",
    "hidden":"1",
    "syscall_number":"78",
    "time":"1578384927606",
    "local_ip":"192.168.165.152",
    "hostname":"test"
}
```


### LKM Hidden Alert
```json
{
    "uid":"-1",
    "data_type":"702",
    "module_name":"diamorphine",
    "hidden":"1",
    "time":"1578384927606",
    "local_ip":"192.168.165.152",
    "hostname":"test"
}
```


### Interrupts Hook Alert
```json
{
    "uid":"-1",
    "data_type":"703",
    "module_name":"syshook",
    "hidden":"1",
    "interrupt_number":"2",
    "time":"1578384927606",
    "local_ip":"192.168.165.152",
    "hostname":"test"
}
```


### About Performance of AgentSmith-HIDS

Testing Environment:

| CPU       | Intel(R) Core(TM) i7-4870HQ CPU @ 2.50GHz 2 Core |
| --------- | ------------------------------------------------ |
| RAM       | 2GB                                              |
| OS/Kernel | Centos7  /  3.10.0-1062.7.1.el7.x86_64           |

Testing Result:

| Hook Handler           | Average Delay(us) |
| ---------------------- | ----------------- |
| execve_entry_handler   | 10.4              |
| connect_handler        | 7.5               |
| connect_entry_handler  | 0.06              |
| recvfrom_handler       | 9.2               |
| recvfrom_entry_handler | 0.17              |
| fsnotify_post_handler  | 0.07              |

Original Testing Data:

[Benchmark Data](https://github.com/EBWi11/AgentSmith-HIDS/tree/master/benchmark_data)



### Documents for deployment and testing purpose:

[Quick Start](https://github.com/EBWi11/AgentSmith-HIDS/blob/master/doc/AgentSmith-HIDS-Quick-Start.md)




### Special Thanks(Not in order)

[yuzunzhi](https://github.com/yuzunzhi)

[hapood](https://github.com/hapood)

[HF-Daniel](https://github.com/HF-Daniel)




### Wechat of developer

<img src="doc/wechat.jpg" width="50%" height="50%"/>


### Wechat channel of '灾难控制局'

We would constantly provide information about the functionalities of AgentSmith-HIDS via this channel, a good place to receive the most updated news:)

<img src="doc/SecDamageControl.jpg" width="50%" height="50%"/>


## License

AgentSmith-HIDS kernel module are distributed under the GNU GPLv2 license.
