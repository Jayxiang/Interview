# iOS 面试-第三方库相关

- [pod update 和 pod install 的区别](#pod-update-和-pod-install-的区别)
- [动态库和静态库的区别](#动态库和静态库的区别)
- [Git 常用的命令](#Git-常用的命令)
- [URL Schema 和 Universal Link](#url-schema-和-universal-link)

#### pod update 和 pod install 的区别
```
pod install 用于工程第一次安装 pod 库和修改 podfile(添加，更新，移除 pod 库)的时候。
pod update 用于更新特定的 pod(或所有的 pod)版本时。
```

更详细可以参考：[pod install vs. pod update](https://blog.csdn.net/huangyimo/article/details/85130398)

#### 动态库和静态库的区别
动态库形式：.dylib 和 .framework
静态库形式：.a 和 .framework

.a 和 .framework 的区别
.a 是单纯的二进制文件，.framework 是二进制文件+资源文件。
其中 .a 不能直接使用，需要 .h 文件配合，而 .framework 则可以直接使用。
.framework = .a + .h + sorrceFile(资源文件)

静态库：链接时，静态库会被完整地复制到可执行文件中，被多次使用就有多份冗余拷贝  
系统动态库：链接时不复制，程序运行时由系统动态加载到内存，供程序调用，系统只加载一次，多个程序共用，节省内存  
自定义动态库（Embedded Framework）：在我们自己应用的 .app 目录里面，只能自己的 App Extension 和 APP 使用。

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

#### URL Schema 和 Universal Link
```
URL Schema: 跳转的时候会出现提示框，“是够打开XX”，用户体验不好。
微信屏蔽 URL Schema，必须是出现在白名单里面才可以跳转（意味着用户无法在微信里面一键直达 APP)
通用链接的优点:
安全性：当用户安装应用程序,iOS 会检查您已经上传到 web 服务器文件,以确保您的网站允许你的应用程序能打开代表它的 URL 文件,只要你创建并且上传该文件,那么你的应用和服务器之间的关联是安全的。
灵活性：当你的应用程序没有被安装的时候,通用链接照样能够工作。当用户没有安装你的应用程序,点击该链接,将会以用户所期望的以 Safari 的形式访问
私有性：其他的应用程序可以和你的应用程序进行通信，不管你的应用是否被安装。
简单性：通用链接既能支持你的网站,又能支持你的应用。
独特性：与自定义的 URL 链接相比,通用链接不能被其他的应用程序所访问,因为它们使用的是标准的 HTTP 或 HTTPS 链接到你的网站。
```