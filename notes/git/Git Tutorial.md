# Git Tutorial

[TOC]

------

*This part is concluded from [(MIT)The missing semeter/git](https://missing.csail.mit.edu/2020/git/) and [Pro Git](https://git-scm.com/book/zh/v2)*

Data model: DAG

## Basic Git commands

`git init`: Initializing a repository in an existing directory

`git add ***`: Specify the files/directories that we want to track

`git rm ***`: remove a file and no longer track it

`git rm --cached`: no longer track a file (but not remove it from the directory)

`git clone ***`: cloning an existing repository (*** can be a URL address or a local address)

In the `.gitignore` file, we can specify files that we don't want Git to track. 

`git status`: Check the status of all files
`git status -s` or `git status --short`: show short status

`git add`: to stage a modified file. `git add .` for all tracked modified files.

`git diff`: show all modifications since last staging (All unstaged changes)
`git diff --staged`: Show all staged changes compared to last commit

`git commit ***`: commit all **staged** changes. Git will launch an editor, print the `git status` output and ask for a committing message.
`git commit -a ***`: commit all **tracked** modified changes, no matter whether the changes are staged.
`git commit -m "<commit-message>"`

`git mv ***1.md ***2.md`: rename a file



### Git log

`git log`: show all commit histories
`-p` option shows the differences in each commit
`-3` option shows the last 3 commits
`--stat` option shows the statistics in each commit
`--pretty=format` option supports specifying the output format



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
`git checkout -b <branch-name>`: create a new branch and checkout to this branch
`git branch -d <branch-name>` deletes a pointer (branch)

`git merge <branch-name>`: merge the branch to the `HEAD` branch



## Git Remote

`git remote` for all remote servers
`git remote add <shortname> <url>`: add a new remote server and specifies a shortname

`git fetch <remote>`: fetch new data from the remote (since last connection). Only downloads data to local repo and won't merge to the current branch.

`git pull <remote>`: fetch new data from the remote (since last connection) and merge to current branch.

`git push <remote> <branch>`: push the specified branch to the remote.

`git remote show <remote>` for information on the remote.

`git remote rename` 
`git remote remove` or `git remote rm`



### Remote branch

Remote branches are named in the format of `<remote>/<branch>` 

`git push <remote> <local-branch>:<remote-branch>`: pushes the local branch to remote branch

We can use local branches (called "tracking branch") to track remote branches, and the tracked remote branch is called "upstream branch".

- `git checkout -b <branch> <remote>/<remote-branch>` -> `git checkout --track <remote>/<remote-branch>`: create a local tracking branch.
- `git branch -u <remote>/<remote-branch> <local-branch>`: let the local branch to track the remote branch.
- `git branch -vv`: show all tracking/upstream branches.

`git push <remote> --delete <remote-branch>`: delete a remote branch.
