### Git使用技巧

#### 一次添加多个文件

```
$ git add .
```

**为什么用 `.` 不用 `*` ?**

In order to add the files that are not in the gitignore file,

> use `git add .` in the place of  `git add *`

This will stop confusing the *unix system since * means all ( including the ignored ones ) while . means the ones relative to the active action

***

#### 强制添加文件

```
git add -f aaa.exe
```

一般在添加一个被忽略的文件时，会提示如下错误：

```
$ git add phantomjs.exe
The following paths are ignored by one of your .gitignore files:
phantomjs.exe
Use -f if you really want to add them.
```

遇到这种情况时候需要使用 `git add -f` 命令强制添加这个文件。

详见：[git强制添加(add)文件](http://m.blog.csdn.net/article/details?id=50906447)

***

