# iOS 面试-第三方库相关

- [pod update 和 pod install 的区别](#pod-update-和-pod-install-的区别)
- [Git 常用的命令](#Git-常用的命令)

#### pod update 和 pod install 的区别
```
pod install 用于工程第一次安装 pod 库和修改 podfile(添加，更新，移除 pod 库)的时候。
pod update 用于更新特定的 pod(或所有的 pod)版本时。
```

更详细可以参考：[pod install vs. pod update](https://blog.csdn.net/huangyimo/article/details/85130398)

#### Git 常用的命令
* 1.`git init` 初始化一个 Git 仓库。
* 2.`git add <file>` 添加文件
* 3.`git status` 查看当前 Git 仓库的文件状态。
* 4.`git diff` 查看一个文件前后有什么不同。
* 5.`git commit -m '描述信息'` 提交文件及相关信息。
* 6.`git checkout -- <file>...` :当你改乱了工作区某个文件的内容，想直接丢弃工作区的修改。
* 7.`git log` 命令显示从最近到最远的提交日志（可以回到过去）。
* 8.`git reset --hard HEAD^` 回退到上一个版本。
* 9.`git reset --hard 1094a` 如果是错误回退，只要当前窗口还没有关闭，可以找到最新的 `commit id`，根据 `commit id` 还原。
* 10.`git reflog` 可以用来记录你的每一次命令，即便是误操作，可以通过这个命令进行还原(也就是可以回到未来)。
* 11.`git checkout` 是切换分支的意思。
* 12.`git reset HEAD <file>` 可以把暂存区的修改撤销掉（unstage）
* 13.`git remote add origin 远程仓库地址` 将本地仓库与远程仓库关联
* 14.`git push -u origin master` 会把本地的 `master` 分支内容推送的远程新的 `master` 分支，还会把本地的 `master` 分支和远程的 `master` 分支关联起来.
* 15.`git merge xxx` 合并指定分支到当前分支
* 16.`git branch` 查看当前的所有分支。
* 17.`git branch xxx` 创建指定的分支。