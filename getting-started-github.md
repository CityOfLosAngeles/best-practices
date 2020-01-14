# Getting Started with GitHub

It's your first day on the job, and getting your environment all set up is a big hassle. Hopefully, you've been added as a user to our shared JupyterHub platform. If you'd like to be, shoot us an [email](mailto:ITAData@lacity.org)! Otherwise, if you already do work in Python or R, but want to check that work into version control, then this guide will get you started.

On a Windows machine, you'll need to have a terminal ready. We recommend either [Git for Windows](https://git-scm.com/download/win) or WSL/Ubuntu (search "Ubuntu" in the Microsoft Store). Mac and Linux machines already come with one installed. The terminal is where you'll be typing these Git commands.

* [Create a GitHub repository](#create-a-github-repository-and-sync-local-and-remote-repos)
* [Typical project workflow](#typical-project-workflow)

## Create a GitHub repository and sync local and remote repos

We will create a local folder and add some files. Then, we'll go to our GitHub account, create a repository (repo) there. Finally, we'll link our local folder with the GitHub repository. The remote repository lives on the GitHub website, whereas the local repository is what's on your local computer.

1. Change into the directory where your project files live: `cd my-username/folder_name`
2. Files to include in version control are:
    * Python/R scripts (.py, .R)
    * Markdown files (.md, .Rmd)
    * Notebooks: (.ipynb)
    * Dockerfile
    * Data catalogs
    * Plain text files
    * Other sources for documentation
    * Maybe some small datasets (<10MB, even though Git has a 100MB limit)
    
    You can also let Git know about files you want to keep in the folder, but don't want to check into version control. [Read about .gitignore here.](https://www.pluralsight.com/guides/how-to-use-gitignore-file)

3. Turn this folder into a Git repo: `git init`
4. First time users need to configure their GitHub credentials in their bash:
    
    ```
    git config --global user.name my_username
    git config --global user.email my_email@lacity.org
    ```

5. Add the files you've changed and commit it:
    
    ```
    # To add specific files
    git add notebook1.ipynb script2.py README.md

    # To add specific file types (use asterisk)
    git add *.ipynb  # adds all notebooks
    git add *.py  # adds all Python scripts

    # Commit those changes, and add a human-readable commit message
    # to describe the work. This message will be saved in the project history.
    git commit -m "Added new data"

    # To see which files have uncommitted changes at any point
    git status
    git log
    ```

6. On your GitHub browser, create a new repository with the same `folder_name`. 
7. Grab the link, which looks like `https://github.com/my_username/folder_name.git`. If you're working under the City of Los Angeles organization, the link looks like `https://github.com/CityofLosAngeles/folder_name.git`.
8. Link the local and remote repository, and call the remote repository "origin":

    ```
    git remote add origin https://github.com/my_username/folder_name.git

    # List what the remote repositories are. The -v stands for verbose.
    git remote -v
    ```

9. Push the changes in your local repo to the remote repo: `git push origin master`


## Typical project workflow
There are many collaborators on a project/GitHub repo. The **master** branch is the main branch, the canonical source and official code for the project. Any tasks that individual collaborators are working on should be done off of separate branches. [Learn all about branches in GitHub here](https://thenewstack.io/dont-mess-with-the-master-working-with-branches-in-git-and-github/) 

1. Before you start your new task, make sure you sync up with the master: `git pull origin master`
2. Create a new branch to do your task: `git checkout -b my-new-branch`
3. Make changes and commit changes to the remote repo
    
    ```
    git add *
    git commit -m "Here is my commit message"
    git push origin my-new-branch
    ```

4. The `my-new-branch` can be merged with the master branch when the task is complete on the GitHub browser.
5. Delete your old branch: `git branch -d my-new-branch`
6. Repeat this process again and use a new branch for a new task