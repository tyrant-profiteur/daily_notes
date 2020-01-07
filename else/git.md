### Git上传完整流程：

1. 在项目文件夹下，启动git branch,输入git init

2. git add . 添加所有文件

3. .git commit -m '' 上传到本地仓库

4. git remote add origin git@github.com .......git  关联GitHub仓库

   ![img](E:\daily_notes\Java\pictures\Git-0001)

   ​	如果出现错误：fatal: remote origin already exists，则执行以下语句：git remote rm origin，再执行4

5. git push origin master(git push -u origin master,第一次加上 -u,创建)

   ​	git push <远程主机名> <本地分支名>:<远程分支名>：推送本地master分支到远程仓库的master分支

6. 如果5出现错误:failed to push som refs to……,则执行以下语句：git pull origin master，再执行5

7. 如果执行6出现错误:fatal: refusing to merge unrelated histories(拒绝合并不想关历史)，则执行git pull origin master --allow-unrelated-histories，将两个独立的仓库合并（会合并历史），再执行6

### git pull 

pull ≈ fetch + merge，fetch并不会自动合并或修改当前的工作，必须手动将其合并到工作当中。

如果你使用 `clone` 命令克隆了一个仓库，命令会自动将其添加为远程仓库并默认以 “origin” 为简写。 所以，`git fetch origin` 会抓取克隆（或上一次抓取）后新推送的所有工作。