---
title: 解决pip install时unsupported locale setting错误
date: 2018-09-05 14:09:36
updated: 2018-09-05 14:21:17
tags:
    - Python
description: pip install locale.Error unsupported locale setting
top: false
---

今天在安装 `docker-compose` 时，使用 `pip install` 命令出现了下面这个错误：

```
➜  sudo pip install -U docker-compose
Traceback (most recent call last):
  File "/usr/bin/pip", line 11, in <module>
    sys.exit(main())
  File "/usr/lib/python2.7/dist-packages/pip/__init__.py", line 215, in main
    locale.setlocale(locale.LC_ALL, '')
  File "/usr/lib/python2.7/locale.py", line 581, in setlocale
    return _setlocale(category, locale)
locale.Error: unsupported locale setting
```

后来查询到是语言配置错误导致的：

```
➜  locale -a
locale: Cannot set LC_CTYPE to default locale: No such file or directory
C
C.UTF-8
en_US
en_US.iso88591
en_US.utf8
POSIX
```

解决方法：

执行如下命令即可。

```
➜  export LC_ALL=C
```

再次尝试安装：

```
➜  sudo pip install -U docker-compose
The directory '/home/ubuntu/.cache/pip/http' or its parent directory is not owned by the current user and the cache has been disabled. Please check the permissions and owner of that directory. If executing pip with sudo, you may want sudo's -H flag.
The directory '/home/ubuntu/.cache/pip' or its parent directory is not owned by the current user and caching wheels has been disabled. check the permissions and owner of that directory. If executing pip with sudo, you may want sudo's -H flag.
Collecting docker-compose
  Downloading https://files.pythonhosted.org/packages/67/03/b833b571595e05c933d3af3685be3b27b1166c415d005b3eadaa5be80d25/docker_compose-1.22.0-py2.py3-none-any.whl (126kB)
    100% |################################| 133kB 143kB/s
Collecting websocket-client<1.0,>=0.32.0 (from docker-compose)
  Downloading https://files.pythonhosted.org/packages/09/12/d21872b618befc489cabde794c7af281d12fa2e194e279132ef1f04a3b07/websocket_client-0.52.0-py2.py3-none-any.whl (198kB)
    100% |################################| 204kB 34kB/s

...
...
Successfully installed PyYAML-3.11 backports.ssl-match-hostname-3.5.0.1 cached-property-1.4.3 certifi-2018.8.24 chardet-2.3.0 docker-3.5.0 docker-compose-1.22.0 docker-pycreds-0.3.0 dockerpty-0.4.1 docopt-0.6.2 enum34-1.1.2 functools32-3.2.3.post2 idna-2.0 ipaddress-1.0.16 jsonschema-2.6.0 requests-2.9.1 six-1.10.0 texttable-0.9.1 urllib3-1.13.1 websocket-client-0.52.0
You are using pip version 8.1.1, however version 18.0 is available.
You should consider upgrading via the 'pip install --upgrade pip' command.
```

相关参考：

* [解决pip install时unsupported locale setting错误 | 阿阿燃](http://linfuyan.com/locale_error_unsupported_locale_setting/)
