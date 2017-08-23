### MongoDB备份与恢复

mongodb中有工具mongodump和mongorestore提供了非常方便的对数据库备份与恢复功能。可以在命令后面加--help选项查看两个工具的帮助文档。

```
/ # mongodump --help
Usage:
  mongodump <options>

Export the content of a running server into .bson files.

Specify a database with -d and a collection with -c to only dump that database or collection.

See http://docs.mongodb.org/manual/reference/program/mongodump/ for more information.

general options:
      --help                                                print usage
      --version                                             print the tool version and exit

verbosity options:
  -v, --verbose=<level>                                     more detailed log output (include multiple times for more verbosity, e.g. -vvvvv, or specify a numeric value, e.g. --verbose=N)
      --quiet                                               hide all log output  
  ...
  ...
```


#### MongoDB备份

mongodump备份命令语法: 

```
> mongodump -h dbhost -d dbname -o dbdirectory
```

* `-h` : MongDB所在服务器地址，例如：127.0.0.1，当然也可以指定端口号：127.0.0.1:27017
* `-d` : 需要备份的数据库实例，例如：test
* `-o` ：备份的数据存放位置，例如：c:\data\dump，当然该目录需要提前建立

备份指定数据库：

```
> mongodump -h 127.0.0.1:27017 -d local -o D:\Test\aatt
```

备份所有数据库：

```
> mongodump --host 127.0.0.1 --port 27017
```

***

#### MongoDB恢复

mongorestore 恢复备份命令语法：

```
> mongorestore -h <hostname><:port> -d dbname <path>
```

* `--host <:port>, -h <:port>` : MongoDB所在服务器地址，默认为： localhost:27017
* `--db , -d` ：需要恢复的数据库实例，该名称与备份时的名称可以不一致
* `--drop` : 恢复的时候，先删除当前数据，然后恢复备份的数据。
* `<path>` ：mongorestore 最后的一个参数，设置备份数据所在位置，例如：c:\data\dump\test。
你不能同时指定 <path> 和 --dir 选项，--dir也可以设置备份目录。
* `--dir` : 指定备份的目录

恢复备份数据到指定的服务器数据库中：

```
> mongorestore -h 127.0.0.1:27017 -d test2 D:\Test\aatt\local
```

***

#### Alpine系统下的MongoDB备份与恢复

Alpine系统下使用MongoDB，需要安装MongoDB包: ` apk add mongodb` 。如果要使用备份与恢复功能，需要安装 `mongodb-tools` 包：`apk add mongodb-tools`。

```
alpine:edge

$ echo http://dl-4.alpinelinux.org/alpine/edge/testing >> /etc/apk/repositories

$ apk add --no-cache mongodb mongodb-tools

$ ls /usr/bin/
```
