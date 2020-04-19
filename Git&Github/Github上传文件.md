# Github操作流程

创建远程库（create new repository）

创建远程库地址别名
	`git remote -v`查看当前所有远程地址别名
	`git remote add [别名] [远程地址] `，通常使用`git remote add origin github_addr`

推送`git push [别名] [分支名]`，比如`git push origin master`

克隆 `git clone [远程地址]`会把完整的远程库下载到本地，创建远程地址别名，初始化本地库

拉取 `pull = fetch + merge`
	`git fetch [远程库地址别名] [远程分支名]`
	`git merge [远程库地址别名/远程分支名]`
	`git pull [远程库地址别名] [远程分支名]`

如果上传的文件不是基于Github远程库的最新版本，那就不能推送，必须先拉取

# Github上传文件

## Git设置

- 下载完Git之后，在Terminal中输入以下命令来初始化用户名和用户邮箱（该邮箱只是作为一个识别功能）

  ```
  $ git config --global user.name "bowie"
  $ git config --global user.email "hbw234@outlook.com"
  ```

- 设置SSH Key

  1. ` cd ~/.ssh` 和`ls`命令查看是否有ras文件，有则已经生成密钥

  2. 没有密钥，通过以下代码生成，默认路径，默认没有密码

     ```bash
     $ ssh-keygen -t rsa -C "hbw234@outlook.com"
     ```

- 上传本地文件到Github

  1. `cd fileaddress`来定位到带上传文件夹，使用`git init`，初始化成功之后会有一个 .git文件夹，通过`ls -laf` 可以查看

  2. 执行命令 `git add .`将所有文件添加到仓库

  3. `git commit -m "message"`提交注释，如果不使用`-m`则会出现Vim界面来添加注释，一定要有
     <u>然后带上传的文档就在暂存区了</u>

  4. 然后复制Github仓库地址，执行命令

     ```bash
     git remote add origin https://github.com/....
     ```

- 上传本地代码
  1. `git push -u origin master`来上传文件
  2. 如果出现报错 `non-fast-forward`,这是因为git仓库中已有一部分代码，不允许你直接覆盖，可以有以下两个方法
     强推`git push -f`，除非很有信心，普遍不建议
     可以先`git pull`再`git push`,这里执行的是fetch和merge两条命令

---

