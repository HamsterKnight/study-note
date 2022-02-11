# 本地仓库推送到github的远程空仓库中

1. 初始化git配置文件

   ```
   git init
   ```

2. 本地跟远程仓库地址绑定

   ```
   git remote add origin ssh的仓库地址
   ```

3. 本地拉取更新仓库上的内容

   ```
   git pull
   ```

4. 本地分支main跟远程分支main绑定

   ```
   git branch --set-upstream-to=origin/main main
   
   ```

   