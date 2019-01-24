## LEARN GIT  *through blog [blog][https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/0013743256916071d599b3aed534aaab22a0db6c4e07fd0000]*

----

### install the git bash

linux: pacman -S git

----

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

### branches
   1. git checkout -b branchName(-b : create a branch and switch to it)
   	-b :equals two command:
				a: git branch bname
				b: git checkout bname
   2. git branch (see branches)

### Problems

1. #### switch to ssh instead of https

   1. git remote -v : see connection 
   2. git remote rm oringin :clear connection
   3. git remote add oringin git@github.com:MOIPA/REpo.git

2. #### error when test SSH connection

   1. ssh -t github.com  : denied
   2. solution : ssh-keyscan -t rsa github.com >> ~/.ssh/known_hosts
   3. then git push -u origin master

