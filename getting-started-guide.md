# Getting Started with GitHub

It's your first day on the job, and getting your environment all set up is a big hassle. Hopefully, you've been added as a user to our shared JupyterHub platform. If you'd like to be, shoot us an [email](mailto:ITAData@lacity.org)! Otherwise, if you already do work in Python or R, but want to check that work into version control, then this guide will get you started.

On a Windows machine, you'll need to have a terminal ready. We recommended either [Git for Windows](https://git-scm.com/download/win) or WSL/Ubuntu (search "Ubuntu" in the Microsoft Store). Mac and Linux machines already come with one installed. The terminal is where you'll be typing these Git commands.

* [Create a GitHub repository](#create-a-github-repository-and-sync-local-and-remote-repos)
* [Typical project workflow](#typical-project-workflow)

## Create a GitHub repository and sync local and remote repos

We will create a local folder and add some files. Then, we'll go to our GitHub account, create a repository (repo) there. Finally, we'll link our local folder with the GitHub repository. The remote repository lives on the GitHub website, whereas the local repository is what's on your local computer.

* In your bash, make a new project folder: `mkdir folder_name`
* Change into that directory: `cd folder_name`
* Move your existing notebooks into new folder: `mv *.ipynb folder_name`
* Turn this folder into a Git repo: `git init`
* First time users need to configure their GitHub credentials in their bash:
    
```
git config --global user.name my_username
git config --global user.email my_email@lacity.org
```

* Add the notebooks and commit it:
    
```
git add *.ipynb
git commit -m "Initial commit"

# To view the changes just made
git status
git log
```

* On your GitHub browser, create a new repository with the same folder_name. 
* Grab the link, which looks like https://github.com/CityofLosAngeles/folder_name.git
* Link the local and remote repository, and call the remote repository "origin":

```
git remote add origin https://github.com/my_username/folder_name.git

# List what the remote repositories are. v stands for verbose.
git remote -v
```

* Push the changes in your local repo to the remote repo: `git push origin master` or `git push origin other-branch-name`


## Typical project workflow
There are many collaborators on a project/GitHub repo. The **master** branch is the main branch, the canonical source and official code for the project. Any tasks that individual collaborators are working on should be done off of separate branches. [Read more about branches in GitHub here](https://thenewstack.io/dont-mess-with-the-master-working-with-branches-in-git-and-github/) 

* Before you start your new task, make sure you sync up with the master: `git pull origin master`
* Create a new branch to do your task: `git checkout -b my-new-branch`
* Make changes and commit changes to the remote repo
    
```
git add *
git commit -m "Here is my commit message"
git push origin my-new-branch
```

* The `my-new-branch` can be merged with the master branch when the task is complete.
* Delete your old branch: `git branch -d my-new-branch`
* Repeat this process again and use a new branch for a new task