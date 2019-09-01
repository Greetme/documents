# `GitHub` 常见问题

---
## (190725) warning：LF will be replaced by CRLF in ××××.××(文件名) <br/> The file will have its original line ending in your working directory.

### 问题
Windows 下 Git 使用报错：
```
warning：LF will be replaced by CRLF in ××××.××(文件名)
The file will have its original line ending in your working directory.
```
### 原因
Windows 中的换行符为 CRLF，而 Linux 下的换行符为 LF （使用 Git 命令行 Git Bash，实际上就是相当于 Linux 环境），所以在执行 `$ git add xxx.xx` 操作时，会出现这个错误提示。

### 解决方法与步骤
（**注意：会删仓库！会删仓库！会删仓库！**）

(1) 删除 .git：`$ rm -rf .git`  
(2) 禁用自动转换，即设置：`$ git config --global core.autocrlf false`  
再重新初始化，并执行添加 add 操作：  
(3) `$ git init`  
(4) `$ git add xxx.xx`
---
## (190901) ! [rejected] master -> master (non-fast forward)
[参考链接1](https://blog.csdn.net/xieneng2004/article/details/81044371)，[参考链接2](https://www.jianshu.com/p/f26c71d05e44)

### 问题
如名为 test 的项目操作过程：

(1) 在 test 文件夹里打开 Git Bash，输入 `git init` 初始化本地仓库，GitHub 创建远程仓库 test

(2) 以下命令关联本地和远程仓库，***为我的用户名
```	
$ git remote add origin git@github.com:***/manage.git
```

(3) 本地已经有项目代码了在 add 和 commit 之后，想要 push 到远程仓库
```
$ git push origin master
```

此时报错：
```
! [rejected] master -> master (non-fast forward)
    ……
    ……
```

### 原因
问题（non-fast-forward）的出现原因在于：Git仓库中已经有一部分代码，所以它不允许你直接把你的代码覆盖上去。

### 解决方法与步骤
有2种解决方法：  

**方法1**  
(1) `$ git pull origin master --allow-unrelated-histories` // 把远程仓库和本地同步，消除差异  
(2) 重新 add 和 commit 相应文件  
(3) `$ git push origin master`，此时就能够上传成功了

> 注：下述方式未测试，不知与上述方式的效果是否相同。  
> 先把 Git 的东西 fetch 到你本地然后 merge 后再 push：
> ```
> $ git fetch
> $ git merge
> ```

**方法2**（不推荐）  
强推，即利用强覆盖方式用你本地的代码替代 Git 仓库内的内容：  
`$ git push -f origin master`
---
