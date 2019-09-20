``` shell
创建本地git仓库
mkdir /root/git_local
# 创建本地目录
cd /roog/git_local
git init
# 进入本地目录，并使用git init，进行初始化仓库
git config --global user.email "wenlong@wenlong.com"
git config --global user.name "wenlong"
# 配置用户基本信息，这个是必选的
连接远程git仓库，在这里用github做演示
1. 先决条件
在github上创建个账户，并创建仓库，然后添加ssh-key，即将本机的公钥复制到github上
2. 在本地进行工作，然后将push到github上
echo 11 > 1.txt
git add . 
git commit -m "1"
# 每次将本地同步到远程前，都必须使用git add 和git commit
git remote add origin git@github.com:userkk/first.git
# 首次连接远程git仓库的时候，请使用如上命令，以后，就可以直接使用git push了
git常用命令
git brange
# 显示分支
git brange one
# 创建分支one
git checkout one
# 切换到分支one
git push
# 推送变更到仓库，如果是远程仓库，第一次提交更新必须使用git remote add origin！！！
git log
# 显示commit的日志
git log --pretty=oneline
# 简洁显示commit的日志
git tag
# 显示tag
git tag -a v1 -m "1banben"
# 创建tag v1，并且给v1添加描述信息1版本
git push origin v1
# 只推送v1版本的变更
git push --tag origin
# 推送所有tag
git show
# 显示所有的tag
git show v1
# 显示v1的信息
git tag v1 -d 
# 删除v1
git push origin :refs/tags/v0
# 将本地的tag v0 推送到远程仓库
git ls-remote brange
# 显示远程分支
注意，git是在本地clone一个远程服务器上的仓库，然后再本地进行各种变更，然后再将所有的变更推送到远程服务器，即push
```