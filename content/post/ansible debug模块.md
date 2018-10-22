+++
title = "ansible debug模块"
date = "2018年9月19日"
author = "vhlv"
cover = "shell.png"
description = "首先确认ip和端口，例如ip为127.0.0.1, 端口为1080, 打开终端，运行以下命令："

+++



### ansible debug模块

ansible playbook可以将多个命令组合来执行，但是很多时候我们需要接收服务器的反馈，所以debug模块就非常重要了。

# 模块说明

调试模块，用于在调试中输出信息
 常用参数：
 msg：调试输出的消息
 var：将某个任务执行的输出作为变量传递给debug模块，debug会直接将其打印输出
 verbosity：debug的级别（默认是0级，全部显示）

例程：

```SH
- name: Print debug infomation eg1 
   hosts: test2 
   gather_facts: F 
  vars: 
    user: jingyong 
  tasks: 
  - name: Command run line 
    shell: date 
    register: result 
  - name: Show debug info 
    debug: var=result verbosity=0
```

程序是将命令date返回信息使用debug模块打印出来。

返回结果如下：

```sh
PLAY [Print debug infomation eg] *********************************************** 

TASK [Show debug info] ********************************************************* 
ok: [192.168.0.1] ={ 
"result": { 
"changed": true, 
"cmd": "date", 
"delta": "0:00:00.002400", 
"end": "2016-08-27 13:42:16.502629", 
"rc": 0, 
"start": "2016-08-27 13:42:16.500229", 
"stderr": "", 
"stdout": "2016年 08月 27日 星期六 13:42:16 CST", 
"stdout_lines": [ 
"2016年 08月 27日 星期六 13:42:16 CST" 
], 
"warnings": [] 
} 
} 
ok: [192.168.0.2] ={ 
"result": { 
"changed": true, 
"cmd": "date", 
"delta": "0:00:00.003847", 
"end": "2002-01-12 03:08:37.493383", 
"rc": 0, 
"start": "2002-01-12 03:08:37.489536", 
"stderr": "", 
"stdout": "2002年 01月 12日 星期六 03:08:37 CST", 
"stdout_lines": [ 
"2002年 01月 12日 星期六 03:08:37 CST" 
], 
"warnings": [] 
} 
} 

PLAY RECAP ********************************************************************* 
192.168.0.1 : ok=2changed=1unreachable=0failed=0   
192.168.0.1 : ok=2changed=1unreachable=0failed=0   
```

可以看到debug不光输出了date命令结果，还返回了很多相关调试信息，只需要date返回值，可以使用变量属性过滤 如：result.stdout 就是命令的返回值。

程序改成：

```sh
- name: Print debug infomation eg 
  hosts: test2 
  gather_facts: F 
  tasks: 
  - name: Command run line 
    shell: date 
    register: result 
  - name: Show debug info 
    debug: var=result.stdout verbosity=0
```

运行结果：

```sh
PLAY [Print debug infomation eg] *********************************************** 

TASK [Command run line] ******************************************************** 
changed: [192.168.0.1] 
changed: [192.168.0.2] 

TASK [Show debug info] ********************************************************* 
ok: [192.168.0.1] ={ 
"result.stdout": "2002年 01月 12日 星期六 03:16:26 CST" 
} 
ok: [192.168.0.2] ={ 
"result.stdout": "2016年 08月 27日 星期六 13:50:05 CST" 
} 

PLAY RECAP ********************************************************************* 
192.168.0.1  : ok=3changed=1unreachable=0failed=0   
192.168.0.2  : ok=3changed=1unreachable=0failed=0
```

