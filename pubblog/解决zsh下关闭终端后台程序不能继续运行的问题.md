### 解决zsh下关闭终端后台程序不能继续运行的问题

`bash` 终端退出后要想在后台继续运行程序非常简单，直接在程序运行指令后加上 `&` 即可，比如

```
$ python hello.py &
```

注意： `$` 表示 `bash` 状态下，`->` 表示 `zsh` 状态下。

但是我发现使用 `zsh` 的时候这样退出，程序即刻被终止了，经过一番查询，解决方法很简单，`zsh` 通过 `&!` 或者 `&|` 使程序后台运行：

```
-> python hello.py &!
# or
-> python hello.py &|
```

但是请注意根据 `zsh` 的官方手册说明，使用这种方式后台，程序会即刻 `disown` ，无法使用 `jobs` 以及 `bg` 来调度：

> If a job is started with ‘&|‘ or ‘&!‘, then that job is immediately disowned. After startup, it does not have a place in the job table, and is not subject to the job control features described here.

当然我们也可以先切换到 `bash` ，使用前一种方法：

```
-> bash
$ python hello.py &
# 再切换回zsh
$ zsh
->
```
