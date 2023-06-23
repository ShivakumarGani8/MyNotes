1) git
    1.git config
    -To config username ->git config --global user.name "username"
    -To config email ->git config --global user.email email_address
    -To config branch -> git config --global init.default main

    Intially when we change directory of our git bash it'll divide files into two types
    -Untracked -> The file is not been tracked by git
    -tracked -> The file is been tracked by git(File added using add command)
    Once we add file into git it unstaged state until we commit on it
    
    2.git init
    -To initialize empry repository ->Git init
    3.git status
    -To see status of current directory ->git status
    -To see short status of current directory -> git status -s
    4.git add
    -To add file from working directory to git staging area->git add file_name
    -To add all files from working directory to git staging area-> git --a
    5.git rm
    -To remove staged file from git-> git rm --cached file_name
    -To remove all staged files from git-> git reset head --
    6.git commit
    -To commit a file from staging area-> git commit file_name(A new shell will be opened where it'll ask for commenting to add comment press i "add comments" press esc & :wq to close opened shell)
    -To commit files directly from staging area->git commit -m "add comments"
    -To commit files directly without stage -> git commit -a -m "add comments"
    7.git checkout
    -To rollback to last commit -> git checkout file_name
    -To rollback to last commit for all files -> git checkout -f
    8.git log
    -To check log of all the previous commits-> git log
    -To limit/filter logs->git log -p -limit(integer)
    9.git diff
    -To compare staged area and working area->git diff
    -To compare staged area and last commit-> git diff --staged
    10.git ignore
    -To ignore a file to be tracked from git -> create a file named .gitingnore in current working directoy

2) touch
    1. touch 
    -To create an empty file -> touch filename.extension

3)Branch
    1. To create a branch -> git branch branch_name(We can work on sub branches and where we can even commit the file once the file is commited it keeps a main copy in main branch until we merge)
    2. To toggle between branches -> git checkout branch_name
    3. To merge branches(By standing main branch) -> git merge branch_name
    4. To directly create and switch to branch -> git branch -b branch_name creates and swithched to the branch created 
