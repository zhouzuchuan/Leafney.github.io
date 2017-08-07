### Linux系统实时监控-Glances


#### Ubuntu系统下安装

可以通过 apt-get 方式来安装：

```
$ sudo apt-get update
$ sudo apt-get install glances
```

还可以通过 pip 方式来安装：

```
$ sudo pip install python-dev
$ sudo pip install glances
```

建议通过 `pip` 的方式来安装，因为我通过 `apt-get` 方式安装的 `glances` 版本为 `Glances version 1.7.3 with PsUtil 1.2.1` ，而通过 `pip` 安装的为 `Glances v2.7.1 with psutil v4.3.1` ，旧版本的功能比较简单。

在安装过程中出现报错：

```
warning: no previously-included files matching '*' found under directory 'docs/_build'
```

首先执行如下命令并尝试再次安装：

```
sudo apt-get install libpq-dev python-dev
```

还可以通过官方给出的方式安装：

```
$ curl -L https://bit.ly/glances | /bin/bash
```

or:

```
$ wget -O- https://bit.ly/glances | /bin/bash
```

***

#### Alpine系统下安装

执行如下命令安装

```
$ apk update
$ apk add python-dev py-pip py2-psutil
$ pip install glances
```

***

#### 使用 

安装完成后，可以执行下面的命令启动 Glances：

```
$ glances
```

可以看到类似下面的输出：

```
MyServer (Ubuntu 14.04 64bit / Linux 4.4.0-38-generic)                                            Uptime: 1:41:59

CPU  [  3.0%]   CPU       3.0%  nice:     0.0%  ctx_sw:   193   MEM      6.5%   SWAP      0.0%   LOAD    2-core
MEM  [  6.5%]   user:     1.3%  irq:      0.0%  inter:    266   total:  1.95G   total:   2.24G   1 min:    0.00
SWAP [    0%]   system:   1.3%  iowait:   0.0%  sw_int:   372   used:    129M   used:        0   5 min:    0.00
                idle:    97.1%  steal:    0.2%                  free:   1.82G   free:    2.24G   15 min:   0.00

NETWORK     Rx/s   Tx/s   TASKS 114 (140 thr), 1 run, 113 slp, 0 oth sorted automatically
docker0       0b     0b
eth0        75Kb   44Kb     CPU%  MEM%  VIRT   RES   PID USER        NI S     TIME+   R/s   W/s Command 
lo            0b     0b      4.7   1.2  377M 23.2M 11918 tiger        0 R   0:01.90     0     0 /usr/bin/python /
                             0.3   0.1  250M 2.63M   556 syslog       0 S   0:00.18     0     0 rsyslogd
DISK I/O     R/s    W/s      0.0   0.0     0     0    18 root       -20 S   0:00.00     0     0 perf
dm-0           0    19K      0.0   0.1 23.1M 2.12M   915 root         0 S   0:00.00     0     0 cron
dm-1           0      0      0.0   0.2 42.4M 3.22M   630 root         0 S   0:00.10     0     0 /lib/systemd/syst
xvda1          0      0      0.0   0.0     0     0    19 root         0 S   0:00.00     0     0 xenwatch
xvda2          0      0      0.0   0.0     0     0    81 root       -20 S   0:00.00     0     0 bioset
xvda5          0    19K      0.0   0.0     0     0     2 root         0 S   0:00.00     0     0 kthreadd
xvdb           0      0      0.0   0.0     0     0    71 root       -20 S   0:00.00     0     0 bioset

```

要退出 `Glances` 终端，按 `ESC` 键或 `Ctrl + C`。




通过browser查看

```
$ sudo pip install bottle

$ glances -w
Glances web server started on http://0.0.0.0:61208/
```

然后在浏览器端输入网址即可查看，默认会监听 `61208` 端口。

#### 

* https://github.com/nicolargo/glances
* https://glances.readthedocs.io/en/stable/index.html
