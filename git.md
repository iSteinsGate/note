# git初始化仓库

1. 库初始化

   ```bash
   git init
   ```

2. 添加远程仓库地址

   ```
   git remote add origin git@github.com:iSteinsGate/note.git
   ```

3. 拉取master分支

   ```
   git pull --rebase origin master
   ```

   注意：git pull = git fetch + git merge
   			git pull --rebase = git fetch + git rebase

4. push到远程主分支

   ```
   git push -u origin master
   ```

5. 关联远程主分支(关联后，可直接push)

   ```
   git push --set-upstream origin master
   ```

   