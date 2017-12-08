---
title: cURL-命令行下工作的文件传输工具
date: 2016-05-04 22:16:00
tags:
    - Linux
categories: 
    - 开发笔记
description: cURL-命令行下工作的文件传输工具
---

cURL是一种命令行工具，作用是发出网络请求，然后得到和提取数据，显示在"标准输出"（stdout）上面。

#### Linux下安装cURL

```
$ sudo apt-get install -y curl
```

#### Windows下安装cURL

1. Windows下默认没有cURL命令，需要安装后才能使用
2. 到网址 [curl download](https://curl.haxx.se/download/trash/) 下载curl安装包，我下载的是 `curl-7.33.0-win64-ssl-sspi.zip 2013-10-16 04:24 698K` 。
3. 解压安装包，只有一个curl.exe文件，在 `curl.exe` 所在目录下打开 `cmd` 命令行窗口，就可以直接使用 `curl` 命令了。
4. 为了让 `curl` 支持访问 `https` 的网址，需要下载 `OpenSSL` ,到 [Win32OpenSSL](http://slproweb.com/products/Win32OpenSSL.html) 下载 `Win32 OpenSSL v1.0.1t Light` 文件。
5. 上面的 2、3、4 步，如果你觉得太麻烦的话，也可以到 [cURL - Download](https://curl.haxx.se/download.html#Win64) 下载 `Win64 x86_64 7zip    7.49.0    binary    SSL  SSH  Viktor Szakáts` 这一项，下载后压缩包里的bin目录下有三个文件：`curl.exe`，`curl-ca-bundle.crt`，`libcurl.dll`，里面自带了SSL的证书文件。

***

#### cURL常用命令整理

格式： `curl [参数] [URL地址]`

##### <span id='demo_top'>参数介绍</span>

| 缩写 | 完整命令 | 示例 | 释意|
|:-----:|:-----|:----:|:-----|
| `-A` | `--user-agent <agent string>` | [demo](#demo_user_agent) | 指定User-Agent的值 |
| `-b` | `--cookie <name=data/file>` | [demo](#demo_b) | cookie字符串或文件读取位置，使用option来把上次的cookie信息追加到http request里面去 | 
| `-c` | `--cookie-jar <file name>` | [demo](#demo_c) | 操作结束后把cookie写入到这个文件中 | 
| `-C` | `--continue-at <offset>` | [demo](#demo_continue) | 断点续传 | 
| `-d` | `--data <data>` | [demo](#demo_data) | HTTP POST方式传送数据 `application/x-www-url-encoded` | 
| `-D` | `--dump-header <file>` | [demo](#demo_dump) | 将请求返回的响应信息保存到指定的文件中 | 
| `-e` | `--referer <URL>` | [demo](#demo_referer) | 指定引用地址，表示从哪里跳转过来的 | 
| `-E` | `--cert <certificate[:password]>` |  |  |
| `-F` | `--form <name=content>` | [demo](#demo_form) | HTTP 表单方式提交数据 `multipart/form-data` | 
| `-G` | `--get` |   | 使用GET方式请求，并将参数拼接在URL的`?`后面 | 
| `-H` | `--header <header>` | [demo](#demo_header) |  指定请求头参数 | 
| `-I` | `--head` | [demo](#demo_head) | 仅返回头部信息，使用`HEAD`请求 | 
| `-L` | `--location` | [demo](#demo_location) | 执行重定向操作 `3xx` | 
| ` ` | `--limit-rate <speed>` |   | 限制最大传输率 | 
| `-m` | `--max-time <seconds>` |   | 指定处理的最大时长 | 
| ` ` | `--max-filesize <bytes>` |  | 指定要下载的文件的最大长度，如果超过bytes值，下载并不开始、返回退出码63 |
| `-o ` | `--output <file>` | [demo](#demo_output)  |  将文件保存为命令行中指定的文件名的文件中 |
| `>` | `>` |    | 等同于`-o`,对输出进行转向输出 |
| `-O` | `--remote-name` | [demo](#demo_remote)  | 使用URL中默认的文件名保存文件到本地 |
| `-r` | `--range <range>` | [demo](#demo_range) | 检索来自HTTP/1.1或FTP服务器字节范围;分块下载 | 
| `-s` | `--silent` | [demo](#demo_silent) | 减少输出信息,比如进度显示或错误信息 | 
| ` ` | `--ssl` |    | 使用SSL/TLS方式创建连接请求 | 
| `-T` | `--upload-file <file>` | [demo](#demo_upload) | 使用PUT方法上传，`-T`参数指定上传文件 | 
| `-u` | `--user <user:password>` |  [demo](#demo_user)  | 设置服务器的用户和密码 | 
| `-v` | `--verbose` | [demo](#demo_verbose) | 小写的v参数，用于打印更多信息，包括发送的请求信息，这在调试脚本是特别有用 | 
| ` ` | `--trace` | [demo](#demo_trace) | 查看更详细的通信过程 | 
| ` ` | `--trace-ascii` | [demo](#demo_trace) | 查看更详细的通信过程 | 
| `-w` | `--write-out <format>` | [demo](#demo_write_out)  |  | 
| `-x` | `--proxy <[protocol://][user:password@]proxyhost[:port]>` | [demo](#demo_proxy) | 指定代理服务器地址和端口，端口默认为1080 | 
| `-X` | `--request <command>` | [demo](#demo_request) | 指定GET,HEAD,POST,PUT,DELETE等请求协议 | 
| `-h` | `--help` |    | 使用帮助 | 
| `-V` | `--version` |   | 版本信息 | 
| ` `  | `--retry <num>` |   | 指定重试次数 |

***

#### 示例

<i id='demo_user_agent'></i>  `-A` 指定User-Agent的值  [Back To Top](#demo_top)   
```
$ curl -A "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.0)" -o page.html http://www.www.baidu.com
```

<i id='demo_b'></i>  `-b` 使用cookie文件(当前目录下的cookie.txt文件)  [Back To Top](#demo_top) 

```
// 指定 cookie信息 访问
$ curl -b "name=data" http://www.baidu.com
// 使用 本地文件中的cookie信息 访问
$ curl -b cookie.txt http://www.xxxx.com/api
```

<i id='demo_c'></i> `-c` 将请求得到的cookie信息保存到本地的 cookie.txt文件中  [Back To Top](#demo_top)

```
$ curl -c cookie.txt http://www.alibaba.com
```

<i id='demo_continue'></i>  `-C` 选项可对大文件使用断点续传功能  [Back To Top](#demo_top)

```
// 当文件在下载完成之前结束该进程
$ curl -O http://www.xxx.com/gettext.html
// ############# 20.1%
// 通过添加-C选项继续对该文件进行下载，已经下载过的文件不会被重新下载
$ curl -C -O http://www.xxx.com/gettext.html
// ############# 20.1%
```

<i id='demo_data'></i>  `-d` 通过 `application/x-www-url-encoded` 方式发送POST请求，-d参数以`name=value`的方式指定参数内容  [Back To Top](#demo_top)

```
//多个参数用&连接
$ curl -d "q=hello&param2=test" http://www.google.com  
//多个参数分别指定    
$ curl -d "action=del" -d "id=12" http://localhost/action.php
```

**注意：`-d` 后面`post` 的参数必须用双引号`"`而不是单引号`'`括起来，否则会报错 **

<i id='demo_dump'></i> `-D` 将请求返回的响应信息保存到指定的文件中  [Back To Top](#demo_top)

```
$ curl -D cookie.txt http://www.alibaba.com
```

**注意：-c 仅包含响应头中的cookie信息 -D 包含响应头中的所有响应信息**  

<i id='demo_referer'></i> `-e` 设置`Referer`引用地址  [Back To Top](#demo_top)

```
$ curl -e http://localhost http://www.XXXX.com/wp-login.php
```

<i id='demo_form'></i> `-F` 通过 `multipart/form-data` 方式发送POST请求，`-F`参数以`name=value`的方式指定参数内容；如果值是一个文件，使用`name=@file`的方式来指定;需要指定上传文件类型时，用`type=`来指定 [Back To Top](#demo_top)

```
$ curl -F "action=upload" -F "filename=@file.tar.gz" http://localhost/action.php
//如果通过代理，上述命令可能会被代理拒绝，需要指定上传文件的MIME类型：
$ curl -x proxy.com:8080 -F "action=upload" -F "filename=@file.tar.gz; type=application/octet-stream" http://localhost/action.php
```

<i id='demo_header'></i> `-H` 指定请求头参数  [Back To Top](#demo_top)

```
$ curl -H "Content-Type:application/json" http://example.com
```

<i id='demo_head'></i> `-I` 仅返回头部信息，使用`HEAD`请求  [Back To Top](#demo_top)

```
$ curl -I http://www.baidu.com
```

**注意：使用`-I`时，接口需要支持`Method=HEAD`请求，否则会返回`405 Method Not Allowed`**

<i id='demo_location'></i> `-L` 执行重定向操作  [Back To Top](#demo_top)

```
$ curl -L www.baidu.com
```

<i id='demo_output'></i> `-o` 将文件保存为命令行中指定的文件名的文件中  [Back To Top](#demo_top)

```
// 将百度首页内容抓到 home.html 中
$ curl -o home.html http://baidu.com
// 符号`>`和`-o`效果相同
$ curl > home.html http://baidu.com
```

<i id='demo_remote'></i> `-O` 使用URL中默认的文件名保存文件到本地  [Back To Top](#demo_top)

```
//将内容保存到 gettext.html 中
$ curl -O http://www.xxx.com/gettext.html
```

<i id='demo_range'></i> `-r` 分段下载  [Back To Top](#demo_top)

```
//获取前100字节的数据
$ curl -r 0-99 http://www.get.this/
//获取最后500字节的数据
$ curl -r -500 http://www.get.this/
```

<i id='demo_silent'></i> `-s` 减少输出信息  [Back To Top](#demo_top)

```
$ curl -s http://www.get.this/
```

<i id='demo_upload'></i> `-T` 指定上传文件路径；可以一个`-T`对应一条Url来表示将一个文件上传到指定的Url;或者同时向一条Url上传多个文件  [Back To Top](#demo_top)

```
// ftp 上传
$ curl -T test.sql ftp://用户名:密码@ip:port/demo/curtain/bbstudy_files/
//同时上传多个文件
$ curl -T "{file1,file2}" http://www.example.com
```

<i id='demo_user'></i> `-u` 设置服务器的用户和密码  [Back To Top](#demo_top)

```
$ curl -u 用户名:密码 http://www.example.com
$ curl -u 用户名:密码 -O http://www.XXXX.com/demo/curtain/bbstudy_files/style.css
```

<i id='demo_verbose'></i> `-v` 用于打印更多信息，可以显示一次http通信的整个过程，包括端口连接和http request头信息  [Back To Top](#demo_top)  

```
$ curl -v www.baidu.com
```

<i id='demo_trace'></i> `--trace` 查看更详细的通信过程  [Back To Top](#demo_top)

```
$ curl --trace output.txt www.baidu.com
//或
$ curl --trace-ascii output.txt www.baidu.com
```

<i id='demo_write_out'></i> `-w` 获取指定输出内容  [Back To Top](#demo_top)

```
$ curl -m 10 -w '%{http_code}\n' http://wyh.life/ -so /dev/null
```

<i id='demo_referer'></i> `-x` 指定代理服务器地址和端口  [Back To Top](#demo_top)  

```
$ curl -u <user>:<passwd> -x <proxy>:<port> http://www.get.this/
```

<i id='demo_request'></i> `-X` -X参数可以支持其他动词  [Back To Top](#demo_top)  

```
$ curl -X POST www.example.com
$ curl -X DELETE www.example.com
```

#### 相关链接

* [curl网站开发指南 - 阮一峰的网络日志](http://www.ruanyifeng.com/blog/2011/09/curl.html)
* [CURL常用命令 - 张贺 - 博客园](http://www.cnblogs.com/gbyukg/p/3326825.html)
* [Curl使用 - 简书](http://www.jianshu.com/p/f686e40ad647)
* [linux curl - 简书](http://www.jianshu.com/p/febb67b699eb)
* [提升开发效率小工具之-curl - 简书](http://www.jianshu.com/p/9a2a9802487a)
* [curl命令常用操作 - web开发 - SegmentFault](https://segmentfault.com/a/1190000005177475)
* [cURL备忘 - 脸滚键盘教™ - SegmentFault](https://segmentfault.com/a/1190000004899829)
* [Curl使用 · Issue #42 · dongjun111111/blog · GitHub](https://github.com/dongjun111111/blog/issues/42)
* [curl - zheng-ji's Wiki](http://wiki.zheng-ji.info/Tool/curl.html)


***

#### **help** 

```
$ curl --help
Usage: curl [options...] <url>
Options: (H) means HTTP/HTTPS only, (F) means FTP only
     --anyauth       Pick "any" authentication method (H)
 -a, --append        Append to target file when uploading (F/SFTP)
     --basic         Use HTTP Basic Authentication (H)
     --cacert FILE   CA certificate to verify peer against (SSL)
     --capath DIR    CA directory to verify peer against (SSL)
 -E, --cert CERT[:PASSWD]  Client certificate file and password (SSL)
     --cert-status   Verify the status of the server certificate (SSL)
     --cert-type TYPE  Certificate file type (DER/PEM/ENG) (SSL)
     --ciphers LIST  SSL ciphers to use (SSL)
     --compressed    Request compressed response (using deflate or gzip)
 -K, --config FILE   Read config from FILE
     --connect-timeout SECONDS  Maximum time allowed for connection
 -C, --continue-at OFFSET  Resumed transfer OFFSET
 -b, --cookie STRING/FILE  Read cookies from STRING/FILE (H)
 -c, --cookie-jar FILE  Write cookies to FILE after operation (H)
     --create-dirs   Create necessary local directory hierarchy
     --crlf          Convert LF to CRLF in upload
     --crlfile FILE  Get a CRL list in PEM format from the given file
 -d, --data DATA     HTTP POST data (H)
     --data-raw DATA  HTTP POST data, '@' allowed (H)
     --data-ascii DATA  HTTP POST ASCII data (H)
     --data-binary DATA  HTTP POST binary data (H)
     --data-urlencode DATA  HTTP POST data url encoded (H)
     --delegation STRING  GSS-API delegation permission
     --digest        Use HTTP Digest Authentication (H)
     --disable-eprt  Inhibit using EPRT or LPRT (F)
     --disable-epsv  Inhibit using EPSV (F)
     --dns-servers   DNS server addrs to use: 1.1.1.1;2.2.2.2
     --dns-interface  Interface to use for DNS requests
     --dns-ipv4-addr  IPv4 address to use for DNS requests, dot notation
     --dns-ipv6-addr  IPv6 address to use for DNS requests, dot notation
 -D, --dump-header FILE  Write the headers to FILE
     --egd-file FILE  EGD socket path for random data (SSL)
     --engine ENGINE  Crypto engine (use "--engine list" for list) (SSL)
 -f, --fail          Fail silently (no output at all) on HTTP errors (H)
     --false-start   Enable TLS False Start.
 -F, --form CONTENT  Specify HTTP multipart POST data (H)
     --form-string STRING  Specify HTTP multipart POST data (H)
     --ftp-account DATA  Account data string (F)
     --ftp-alternative-to-user COMMAND  String to replace "USER [name]" (F)
     --ftp-create-dirs  Create the remote dirs if not present (F)
     --ftp-method [MULTICWD/NOCWD/SINGLECWD]  Control CWD usage (F)
     --ftp-pasv      Use PASV/EPSV instead of PORT (F)
 -P, --ftp-port ADR  Use PORT with given address instead of PASV (F)
     --ftp-skip-pasv-ip  Skip the IP address for PASV (F)
     --ftp-pret      Send PRET before PASV (for drftpd) (F)
     --ftp-ssl-ccc   Send CCC after authenticating (F)
     --ftp-ssl-ccc-mode ACTIVE/PASSIVE  Set CCC mode (F)
     --ftp-ssl-control  Require SSL/TLS for FTP login, clear for transfer (F)
 -G, --get           Send the -d data with a HTTP GET (H)
 -g, --globoff       Disable URL sequences and ranges using {} and []
 -H, --header LINE   Pass custom header LINE to server (H)
 -I, --head          Show document info only
 -h, --help          This help text
     --hostpubmd5 MD5  Hex-encoded MD5 string of the host public key. (SSH)
 -0, --http1.0       Use HTTP 1.0 (H)
     --http1.1       Use HTTP 1.1 (H)
     --http2         Use HTTP 2 (H)
     --ignore-content-length  Ignore the HTTP Content-Length header
 -i, --include       Include protocol headers in the output (H/F)
 -k, --insecure      Allow connections to SSL sites without certs (H)
     --interface INTERFACE  Use network INTERFACE (or address)
 -4, --ipv4          Resolve name to IPv4 address
 -6, --ipv6          Resolve name to IPv6 address
 -j, --junk-session-cookies  Ignore session cookies read from file (H)
     --keepalive-time SECONDS  Wait SECONDS between keepalive probes
     --key KEY       Private key file name (SSL/SSH)
     --key-type TYPE  Private key file type (DER/PEM/ENG) (SSL)
     --krb LEVEL     Enable Kerberos with security LEVEL (F)
     --libcurl FILE  Dump libcurl equivalent code of this command line
     --limit-rate RATE  Limit transfer speed to RATE
 -l, --list-only     List only mode (F/POP3)
     --local-port RANGE  Force use of RANGE for local port numbers
 -L, --location      Follow redirects (H)
     --location-trusted  Like '--location', and send auth to other hosts (H)
     --login-options OPTIONS  Server login options (IMAP, POP3, SMTP)
 -M, --manual        Display the full manual
     --mail-from FROM  Mail from this address (SMTP)
     --mail-rcpt TO  Mail to this/these addresses (SMTP)
     --mail-auth AUTH  Originator address of the original email (SMTP)
     --max-filesize BYTES  Maximum file size to download (H/F)
     --max-redirs NUM  Maximum number of redirects allowed (H)
 -m, --max-time SECONDS  Maximum time allowed for the transfer
     --metalink      Process given URLs as metalink XML file
     --negotiate     Use HTTP Negotiate (SPNEGO) authentication (H)
 -n, --netrc         Must read .netrc for user name and password
     --netrc-optional  Use either .netrc or URL; overrides -n
     --netrc-file FILE  Specify FILE for netrc
 -:, --next          Allows the following URL to use a separate set of options
     --no-alpn       Disable the ALPN TLS extension (H)
 -N, --no-buffer     Disable buffering of the output stream
     --no-keepalive  Disable keepalive use on the connection
     --no-npn        Disable the NPN TLS extension (H)
     --no-sessionid  Disable SSL session-ID reusing (SSL)
     --noproxy       List of hosts which do not use proxy
     --ntlm          Use HTTP NTLM authentication (H)
     --oauth2-bearer TOKEN  OAuth 2 Bearer Token (IMAP, POP3, SMTP)
 -o, --output FILE   Write to FILE instead of stdout
     --pass PASS     Pass phrase for the private key (SSL/SSH)
     --path-as-is    Do not squash .. sequences in URL path
     --pinnedpubkey FILE/HASHES Public key to verify peer against (SSL)
     --post301       Do not switch to GET after following a 301 redirect (H)
     --post302       Do not switch to GET after following a 302 redirect (H)
     --post303       Do not switch to GET after following a 303 redirect (H)
 -#, --progress-bar  Display transfer progress as a progress bar
     --proto PROTOCOLS  Enable/disable PROTOCOLS
     --proto-default PROTOCOL  Use PROTOCOL for any URL missing a scheme
     --proto-redir PROTOCOLS   Enable/disable PROTOCOLS on redirect
 -x, --proxy [PROTOCOL://]HOST[:PORT]  Use proxy on given port
     --proxy-anyauth  Pick "any" proxy authentication method (H)
     --proxy-basic   Use Basic authentication on the proxy (H)
     --proxy-digest  Use Digest authentication on the proxy (H)
     --proxy-negotiate  Use HTTP Negotiate (SPNEGO) authentication on the proxy (H)
     --proxy-ntlm    Use NTLM authentication on the proxy (H)
     --proxy-service-name NAME  SPNEGO proxy service name
     --service-name NAME  SPNEGO service name
 -U, --proxy-user USER[:PASSWORD]  Proxy user and password
     --proxy1.0 HOST[:PORT]  Use HTTP/1.0 proxy on given port
 -p, --proxytunnel   Operate through a HTTP proxy tunnel (using CONNECT)
     --pubkey KEY    Public key file name (SSH)
 -Q, --quote CMD     Send command(s) to server before transfer (F/SFTP)
     --random-file FILE  File for reading random data from (SSL)
 -r, --range RANGE   Retrieve only the bytes within RANGE
     --raw           Do HTTP "raw"; no transfer decoding (H)
 -e, --referer       Referer URL (H)
 -J, --remote-header-name  Use the header-provided filename (H)
 -O, --remote-name   Write output to a file named as the remote file
     --remote-name-all  Use the remote file name for all URLs
 -R, --remote-time   Set the remote file's time on the local output
 -X, --request COMMAND  Specify request command to use
     --resolve HOST:PORT:ADDRESS  Force resolve of HOST:PORT to ADDRESS
     --retry NUM   Retry request NUM times if transient problems occur
     --retry-delay SECONDS  Wait SECONDS between retries
     --retry-max-time SECONDS  Retry only within this period
     --sasl-ir       Enable initial response in SASL authentication
 -S, --show-error    Show error. With -s, make curl show errors when they occur
 -s, --silent        Silent mode (don't output anything)
     --socks4 HOST[:PORT]  SOCKS4 proxy on given host + port
     --socks4a HOST[:PORT]  SOCKS4a proxy on given host + port
     --socks5 HOST[:PORT]  SOCKS5 proxy on given host + port
     --socks5-hostname HOST[:PORT]  SOCKS5 proxy, pass host name to proxy
     --socks5-gssapi-service NAME  SOCKS5 proxy service name for GSS-API
     --socks5-gssapi-nec  Compatibility with NEC SOCKS5 server
 -Y, --speed-limit RATE  Stop transfers below RATE for 'speed-time' secs
 -y, --speed-time SECONDS  Trigger 'speed-limit' abort after SECONDS (default: 30)
     --ssl           Try SSL/TLS (FTP, IMAP, POP3, SMTP)
     --ssl-reqd      Require SSL/TLS (FTP, IMAP, POP3, SMTP)
 -2, --sslv2         Use SSLv2 (SSL)
 -3, --sslv3         Use SSLv3 (SSL)
     --ssl-allow-beast  Allow security flaw to improve interop (SSL)
     --ssl-no-revoke    Disable cert revocation checks (WinSSL)
     --stderr FILE   Where to redirect stderr (use "-" for stdout)
     --tcp-nodelay   Use the TCP_NODELAY option
 -t, --telnet-option OPT=VAL  Set telnet option
     --tftp-blksize VALUE  Set TFTP BLKSIZE option (must be >512)
 -z, --time-cond TIME  Transfer based on a time condition
 -1, --tlsv1         Use >= TLSv1 (SSL)
     --tlsv1.0       Use TLSv1.0 (SSL)
     --tlsv1.1       Use TLSv1.1 (SSL)
     --tlsv1.2       Use TLSv1.2 (SSL)
     --trace FILE    Write a debug trace to FILE
     --trace-ascii FILE  Like --trace, but without hex output
     --trace-time    Add time stamps to trace/verbose output
     --tr-encoding   Request compressed transfer encoding (H)
 -T, --upload-file FILE  Transfer FILE to destination
     --url URL       URL to work with
 -B, --use-ascii     Use ASCII/text transfer
 -u, --user USER[:PASSWORD]  Server user and password
     --tlsuser USER  TLS username
     --tlspassword STRING  TLS password
     --tlsauthtype STRING  TLS authentication type (default: SRP)
     --unix-socket FILE    Connect through this Unix domain socket
 -A, --user-agent STRING  Send User-Agent STRING to server (H)
 -v, --verbose       Make the operation more talkative
 -V, --version       Show version number and quit
 -w, --write-out FORMAT  Use output FORMAT after completion
     --xattr         Store metadata in extended file attributes
 -q                  Disable .curlrc (must be first parameter)
```

***
