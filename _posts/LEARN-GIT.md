title: git配置
tags: git
categories: git
date: 2019-03-07 12:03:24
---
## LEARN GIT  *through  [blog](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/0013743256916071d599b3aed534aaab22a0db6c4e07fd0000)*

----

### install the git bash

linux: pacman -S git

----

<!--more-->

### basic operation

 1.	#### single simple file git

> 1. mkdir learn
>
> 2. cd learn
> 3. git init ,git config --global user.name="",....user.email="" ;  **init repo and set name,pwd:** 
> 4. vim learn : **create a file and use git to commit** 
> 5. git commit -m "writed some thing" : **commit information**

2.	####  multiple file

> 1. git add file1
> 2. git add file2
> 3. git commit -m "multiply files"

3. ####	changed file

> 1. git status :   **if the file is changed,use this command to see status**
> 2. git diff readme.txt  : **see the details of the chang**
> 3. git add readme.txt : **upload**
> 4. git commit -m "changed"

4.	####  check repo history

> 1. git log

5.	####  roll back

> 1. git reset --hard HEAD^ 		**^:means your previous version==>  ^^two previous version ==> HEAD^^^ == HEAD~3**

6.	####  roll to future version

> 1. git reset --hard 1094a  (if you remember the id of the target version)
>
>    regret and don't know the version id: **git reflog**  
>
>    2. git reflog : command history

7.	####  redo after commiting or adding && not pushing to the remote repository

> 1. git checkout -- readme.txt  **(!!!not --readme.txt  It means switching to another branch)**
> 2. or use: git reset HEAD readme.txt

8. #### delete file

> 1. rm test.txt
> 2. git rm test.txt
> 3. git commit -m ""

-----

### remote repository

1. #### how to connect to a remote reposity

   1.  ssh-keygen -t ras -C "tassassin@sina.com" :  **default directory is /user/home/.ssh**

   **​	then copy the id_ras.pub to you github !**

   2. git remote add origin https://github.com/MOIPA/tmp.git
   3. **git push -u origin master**     //push content to remote server do it when every local changed 

2. #### how to git clone 

   1. git clone http://github.com/MPOIA/javaEE
   
3. #### how to clone specific branch
	
   1. git clone -b "branch" "url"
   
4. #### how to pull specific branch
	
   1. git checkout -b 本地分支名 origin/远程分支名

### branches
   1. git checkout -b branchName(-b : create a branch and switch to it)
      ​	-b :equals two command:
	​			a: git branch bname
	​			b: git checkout bname
   2. git branch (see branches)
   3. connflicts: solve it in person
      ​	a:git status : show conflicted files
	b:vim conflicted files: show details
	c: git log --graph --pretty=oneline --abbrev-commit  :show branches merge status
   4. git merge --no-ff : use this argument can show merge history in git graph command
   5.  force delete unmerged branches : git branch -D bName

### git stash (used to store the context)
   1. when developing in dev branch, a bug in master branch need to be solved.
	 2. git stash in dev branch
	 3. git checkout -b issue-101 to create a bugfix branch
	 4. after fixed, git checkout master then: git merge -m "bug fix issue 101" issue-101
	 5. git checkout dev , git stash pop to continue
	**conclusion: it is used for clean git , because you can't push when have some unfinished work **

### git push & git pull
   1. git push origin(default name of remote repository) &lt;branch&gt;(can be master or other banch name)
	 2. if push failed,use **git pull** to get the newest update and solve the conflict
	 3. git pull  : if failed and shows no tracking information use **git branch --set-upstream-to=origin/&lt;branch&gt; localBranchName
					​			it failed because u don't connect ur local dev to remote dev
	team work:

      when work with a team, members only can get master branch and it's default name is origin
      now if we want to get a remote branch like dev: **git checkout -b &lt;lbname&gt; origin/&lt;bname>**
### git rebase
   1. make the git history become a straight line

---

### TAG  convient for managing (one tag means one release)
1. #### create tag
	1. git tag v1.0
	2. git tag : show tags
2. #### create tag for old version
	1. git tag v0.1 <71180d>
3. #### show tag details
	1. git show &lt;v0.1&gt;
4. #### create a tag with a readme
	1. git tag -a v0.8 -m "merged something" <0123d>
5. #### delete a tag
	1. git tag -d v0.1
6. #### push tag to remote
	1. git push origin v1.0  :push one tag
	2. git push origin --tags :push all tags
7. #### delete tag of remote
	1. git tag -d v1.0  (delete on local first)
	2. git push origin :refs/tags/v0.9   (inform remote)

### if want to synchronize my respository to github and gitee
1. git remote -v 
2. git remote rm origin
3. git remote add github git@github.com:MOIPA/REpo.git
4. git remote add gitee git@gitee.com:MOIPA/REpo.git 

** if want to push to these two server **
1. git push github master
2. git push gitee master

----

### configure
  1. git config --global color.ui true (high light color)
  
2. config some files won't commited to git
      a: create a file .gitignore (store somefile names which will not be committed)
      b: the file module can be download at *[officialSite](https://github.com/github/gitignore)*
  
3. configure alias:
	 ​    git config --global alias.&lt;st&gt; status
  
4. **one good example:**
      git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
  
5. **notice**
	all the configure are stored in .git/config
	
----

### Problems

1. #### switch to ssh instead of https

   1. git remote -v : see connection 
   2. git remote rm oringin :clear connection
   3. git remote add oringin git@github.com:MOIPA/REpo.git

2. #### error when test SSH connection

   1. ssh -t github.com  : denied
   2. solution : ssh-keyscan -t rsa github.com >> ~/.ssh/known_hosts
   3. then git push -u origin master