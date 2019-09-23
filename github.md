# Helpful git+GitHub commands

Through your system's command line, you can control your GitHub repositories and branches with great precision. 
This section is an evolving list of commands for common operations. Note that for many operations, GitHub has a variety of ways to accomplish the same or similar objective. These are but a few options.

For a glossary of these terms and more, visit the [github site](https://help.github.com/en/articles/github-glossary). Below are basic steps you can take to create a pull workflow for your github repository.  This flow assumes you already have a repository.

Helpful commands:
* [Committing](#committing)
* [Git add](#git-add)
* [Getting rid of unwanted changes](#getting-rid-of-unwanted-changes)
* [Incorporating changes in the master branch to your current](#incorporating-changes-in-the-master-branch-to-your-current)
* [Borrowing a commit from another branch for use in current branch](#borrowing-a-commit-from-another-branch-for-use-in-current-branch)
* [See the repository's in commit history](#see-the-repositorys-commit-history)
* [Pull workflow overview](#pull-workflow-overview)

### Committing 

A commit in GitHub is similar to saving your work. It allows the system to capture the changes you have made and offers checkpoints through IDs that both show the progress of your work and can be referenced for particular tasks.

Once you are ready to commit, you will enter the following command in your terminal

`git commit -m "commit message"`

It is best practice to create commits for separate tasks or features.

### Git add

In order to commit changes you are working on, you need to track them through an add command.

`git add .` will add everything in your current directory
`git add -A` will add everything in the repository

Note that if you never switch directories from the default, these two commands will essentially accomplish the same thing.

### Getting rid of unwanted changes

If you have made changes and no longer wish to commit them, use the git stash command:

Use `git status` to see the file names of what changed. 

`git stash [file name/directory]`

If you need to reverse the stash, use the following command:

`git stash pop`

### Incorporating changes in the master branch to your current

`git fetch origin`

`git rebase origin/master`


### Borrowing a commit from another branch for use in current branch

`git cherry-pick [commit]`

### See the repository's commit history

`git log`


# Pull workflow overview

Below are basic steps you can take to create a pull workflow for your github repository.  This flow assumes you already have a repository.

### Step 1 - **UPDATE** the master branch
In the gitshell and root directory of your repo, type `git checkout master` and then `git pull origin master` to get the latest master code.

### Step 2 - Create a new branch
Type `git checkout -b name-of-branch`

### Step 3 - Make changes and **COMMIT**
Work to address the branch's issue or new feature. Add your changes with `git add .` and make commit(s) with `git commit -m`. 

When the issue/feature is complete and ready for review, you can close the issue with a commit by typing  `git commit -m "fixes #N"`  with #N being the number of the issue. 

### Step 4 - **PUSH BRANCH**
Type `git push origin [name-of-branch]`

### Step 5 - Create a pull request in github

### Step 6 - Merge pull request with github/origin master 

### Step 7 - Checkout local master, pull in from upstream/origin
`git checkout master`

`git pull origin master`

### Step 8 - Delete branch
To delete the branch locally, run `git branch -d {name-of-branch}`

