## Git Tutorial

[TOC]

*This part is concluded from [(MIT)The missing semeter/git](https://missing.csail.mit.edu/2020/git/) and [Pro Git](https://git-scm.com/book/zh/v2)*

Data model: DAG

### Basic Git commands

`git init`: Initializing a repository in an existing directory
`git add ***`: Specify the files/directories that we want to track
`git rm ***`: remove a file and no longer track it
`git rm --cached`: no longer track a file (but not remove it from the directory)
`git clone ***`: cloning an existing repository (*** can be a URL address)

In the `.gitignore` file, we can specify files that we don't want Git to track. 

`git status`: Check the status of all files

![image-20220130140730558](C:\Users\wuyushen\AppData\Roaming\Typora\typora-user-images\image-20220130140730558.png)

`git status -s` or `git status --short`: show short status

![image-20220130140716431](C:\Users\wuyushen\AppData\Roaming\Typora\typora-user-images\image-20220130140716431.png)

`git add`: to stage a modified file. `git add .` for all tracked modified files.
Before staging:
![image-20220130140758854](C:\Users\wuyushen\AppData\Roaming\Typora\typora-user-images\image-20220130140758854.png)
After staging:
![image-20220130140429811](C:\Users\wuyushen\AppData\Roaming\Typora\typora-user-images\image-20220130140429811.png)

`git diff`: show all modifications since last staging (All unstaged changes)

![image-20220130141304897](C:\Users\wuyushen\AppData\Roaming\Typora\typora-user-images\image-20220130141304897.png)

`git diff --staged`: Show all staged changes compared to last commit

![image-20220130141927351](C:\Users\wuyushen\AppData\Roaming\Typora\typora-user-images\image-20220130141927351.png)



`git commit ***`: commit all **staged** changes. Git will launch an editor, print the `git status` output and ask for a committing message.
`git commit -a ***`: commit all **tracked** modified changes, no matter whether the changes are staged.
`git commit -m "<commit-message>"`

`git mv ***1.md ***2.md`: rename a file

![image-20220130144542464](C:\Users\wuyushen\AppData\Roaming\Typora\typora-user-images\image-20220130144542464.png)



### Git log

`git log`: show all commit histories
`-p` option shows the differences in each commit
`-3` option shows the last 3 commits
`--stat` option shows the statistics in each commit
`--pretty=format` option supports specifying the output format

![image-20220130144926906](C:\Users\wuyushen\AppData\Roaming\Typora\typora-user-images\image-20220130144926906.png)

![image-20220130145220825](C:\Users\wuyushen\AppData\Roaming\Typora\typora-user-images\image-20220130145220825.png)



### Git undoing

`git checkout -- <file>...`: to discard changes in working directory
`git restore --staged <file>...`: to unstage the staged changes (only changes the staged/unstaged status, won't change the file)
`git commit --amend`:to cover the last commit with staged files

### Git Tagging

`git tag -l`: list all tags
`git tag -l "<name>"`: list all tags matching the name
`git tag <name>`: create a lightweight tag for the current commit
`git tag -a <name> -m <message>`: create an annotated tag with message
`git show <name>`: show the message about the tagged commit
`git tag -d <name>`: delete a tag

### Git Branching

Git uses pointers to track typical commits, and the `HEAD` pointer always points to the current commit that we're working on.

`git branch`: shows all branches
`git branch <branch-name>` creates a new pointer (branch)
`git checkout <branch-name>` moves the `HEAD` pointer to the new branch, and all commits in the next will be done on the new branch
`git checkout -b <branch-name>`
`git branch -d <branch-name>` deletes a pointer (branch)

`git merge <branch-name>`: merge the branch to the `HEAD` branch

An Example: 1. Create a new branch "missing", and commit in it

![image-20220130213209669](C:\Users\wuyushen\AppData\Roaming\Typora\typora-user-images\image-20220130213209669.png)

2. Checkout the master branch, and do modifications here

![image-20220130214053777](C:\Users\wuyushen\AppData\Roaming\Typora\typora-user-images\image-20220130214053777.png)
![image-20220130214216828](C:\Users\wuyushen\AppData\Roaming\Typora\typora-user-images\image-20220130214216828.png)

3. Merge the missing branch to master branch

![image-20220130214317367](C:\Users\wuyushen\AppData\Roaming\Typora\typora-user-images\image-20220130214317367.png)

