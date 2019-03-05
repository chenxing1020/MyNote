# Git常用操作

* 代码克隆：`git clone git@github.com:chenxing1020/xxx.git`
* 显示状态：`git status`
* 添加修改：`git add -A （提交所有更改）`
* 提交本地修改：`git commit -m "add xxx"`
* 添加远程库：`git remote add origin git@servername:path/xxx.git`
* 显示远程代码库：`git remote -v`
* 上传代码库：`git push -u origin master(第一次上传-u指定默认主机，以后可以直接使用git push指令)`
* 远程覆盖本地库：
  * fetch需要手动合并`git fetch --all`
  * pull命令直接自动合并`git pull origin`
* 合并提交：`git rebase -i [start] [end(可缺省)]`