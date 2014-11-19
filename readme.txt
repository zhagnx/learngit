# git init --bare parent.git    # 创建一个bare repo用做clone用，相当于server端repo
Initialized empty Git repository in /home/gewang/test/test/parent.git/

# git clone parent.git child1 # clone出第一个工作repo
Cloning into 'child1'...
done.
warning: You appear to have cloned an empty repository.
# cd child1/
# echo a > a && git add . && git commit -m "init drop" # 第一次commit
[master (root-commit) 985c04f] init drop
 1 files changed, 1 insertions(+), 0 deletions(-)
     create mode 100644 a

# git push origin master:master # 在server端repo中创建master分支
     Counting objects: 3, done.
     Writing objects: 100% (3/3), 206 bytes, done.
     Total 3 (delta 0), reused 0 (delta 0)
    Unpacking objects: 100% (3/3), done.
    To /home/gewang/test/test/parent.git
     * [new branch]      master -> master

# cd ..
# git clone parent.git child2 # 创建第二个工作repo
     Cloning into 'child2'...
     done.
# cd child2/
# git checkout -b test # 创建test分支
     Switched to a new branch 'test'
# echo abc >> abc && git add . && git commit -m "create test branch"
     [test c3d70fd] create test branch
      1 files changed, 1 insertions(+), 0 deletions(-)
     create mode 100644 abc

# git push origin test:test # 将test分支推送至服务器，注意没有加-u参数
     Counting objects: 4, done.
     Delta compression using up to 24 threads.
     Compressing objects: 100% (2/2), done.
     Writing objects: 100% (3/3), 268 bytes, done.
     Total 3 (delta 0), reused 0 (delta 0)
    Unpacking objects: 100% (3/3), done.
    To /home/gewang/test/test/parent.git
     * [new branch]      test -> test

# git config -l # 查看配置，由于没有使用-u参数，所以没有test分支对应的remote, merge配置项
     core.repositoryformatversion=0
     core.filemode=true
     core.bare=false
     core.logallrefupdates=true
     core.ignorecase=true
     remote.origin.fetch=+refs/heads/*:refs/remotes/origin/*
     remote.origin.url=/home/gewang/test/test/parent.git
     branch.master.remote=origin
     branch.master.merge=refs/heads/master


# cd ../child1 # 回到第一个工作repo
# git branch
* master
# git pull origin test # 在master分支上执行 git pull origin test, 可以看到server端test分支的内容被merge进了本地master分支
remote: Counting objects: 4, done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 3 (delta 0), reused 0 (delta 0)
Unpacking objects: 100% (3/3), done.
From /home/gewang/test/test/parent
 * branch            test       -> FETCH_HEAD
 Updating 985c04f..c3d70fd
 Fast-forward
  abc |    1 +
   1 files changed, 1 insertions(+), 0 deletions(-)
   create mode 100644 abc

# git fetch # 执行git fetch为server端test分支建立本地镜像分支
From /home/gewang/test/test/parent
 * [new branch]      test       -> origin/test
# git branch -a
* master
  remotes/origin/master
  remotes/origin/test

# git checkout test # 虽然本地没有手工创建test分支，但是可以直接checkout，同时git自动为你配置了它与server端test分支的关联。够贴心。
Branch test set up to track remote branch test from origin.
Switched to a new branch 'test'
# git config -l
core.repositoryformatversion=0
core.filemode=true
core.bare=false
core.logallrefupdates=true
core.ignorecase=true
remote.origin.fetch=+refs/heads/*:refs/remotes/origin/*
remote.origin.url=/home/gewang/test/test/parent.git
branch.master.remote=origin
branch.master.merge=refs/heads/master
branch.test.remote=origin
branch.test.merge=refs/heads/test

# echo ddd >> ddd && git add . & git commit -m "update branch test" # 在test分支上checkin代码并推送至服务器
[1] 1260
[test 3a078d4] update branch test
 1 files changed, 1 insertions(+), 0 deletions(-)
  create mode 100644 ddd
  [1]+  Done                    echo ddd >> ddd && git add .
# git push # 不带参数执行git push可以看到本地更新的2个分支都推送了新的change到服务器端。详细解释参考 http://loveky2012.blogspot.com/2012/08/default-behaviour-of-git-pull-and-git-push.html
Counting objects: 4, done.
Delta compression using up to 24 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 295 bytes, done.
Total 3 (delta 0), reused 0 (delta 0)
Unpacking objects: 100% (3/3), done.
To /home/gewang/test/test/parent.git
   985c04f..c3d70fd  master -> master
      c3d70fd..3a078d4  test -> test
# cd ../child2 # 切换到第二个工作repo，执行git pull在将server端change拿到本地后会报错，因为配置里没有关于test分支的mrege信息。（这是由于之前push test分支时没有加-u参数导致的）
# git pull
From /home/gewang/test/test/parent
   985c04f..c3d70fd  master     -> origin/master
      c3d70fd..3a078d4  test       -> origin/test
      You asked me to pull without telling me which branch you
      want to merge with, and 'branch.test.merge' in
      your configuration file does not tell me, either. Please
      specify which branch you want to use on the command line and
      try again (e.g. 'git pull <repository> <refspec>').
      See git-pull(1) for details.

      If you often merge with the same branch, you may want to
      use something like the following in your configuration file:
      [branch "test"]
      remote = <nickname>
      merge = <remote-ref>

      [remote "<nickname>"]
      url = <url>
      fetch = <refspec>

      See git-config(1) for details.

# git branch --set-upstream test origin/test # 手工配置关联
Branch test set up to track remote branch test from origin.
# git pull # 再次pull即可
Updating c3d70fd..3a078d4
Fast-forward
 ddd |    1 +
  1 files changed, 1 insertions(+), 0 deletions(-)
   create mode 100644 ddd
