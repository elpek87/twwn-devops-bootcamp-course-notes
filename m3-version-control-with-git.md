# MODULE 3 - Version Control wit GIT

## Basic commands walkthrough:

`$ git init`

Used to simply create local repository which can either be used locally or be pushed to the remote.

`$ git remote add`

Used to connect local repository with the remote one.

`$ git push --set-upstream origin <branch_name>`

Used to push local branch to remote repository **origin** and set it as the default upstream branch.

## GIT branching concepts

A *branch* is simply a moveable pointer to a commit. Branches are worked on by developers and then merged back into the master branch.

Branching commands walkthrough:

`$ git branch`

Used to display what branch is currently in use

*-d* - deletes branch locally (after deleted on remote)

`$ git chechout`

Used to switch to some other branch. Switch *-b* can be used to create branch in local repo.

## Development models

1. Trunk-Based Development

Small changes get frequently integrated directly into master branch (the trunk).

2. Feature-based Development

Long-lived branches are created by developers and then they merge them back once feature/bugfix is considered done.

## Merge Requests / Pull Requests

PRs are formal requests to merge one branch into another - usually after reviews and checks.

## Merging

Merging in git is basically the process of combining changed from two lines of development into a single history. Typically before you commit your changes and the file was touched by another then you have to pull then push and merge commit is visible in git history. In order to do it in a more clean way you can use git rebase.

`$ git pull -r`

Used to pull the code from the remote and then rebases your commits on top of the remote ones.

## Merge conflicts resolution

A merge conflict happens when Git cannot automatically combine changes because two branches modified the same part of the code in incompatible ways. Merge conflict needs to be fixed manually.

`$ git rebase --continue`

Used to tell git that conflict is resolved and remaining commits can be applied.

## Gitignore

The *.gitignore* file is used to tell git which files and directories should intentionally NOT be tracked or committed.

`$ git rm -r --cached <pattern>`

Used when files or directories that are listed in .gitignore were committed already and you need to remove them.

## macOS specific tips and tricks

Since macOS Finder auto creates *.DS_Store* directories - which you not necessarily need in your git repositories - you can use global *.gitignore* file to stop it from being accidentally added.

1. Create global .gitignore file:
`$ touch ~/.gitignore_global`

2. Add .DS_Sore to global .gitignore file:
`$ echo ".DS_Store" >> ~/.gitignore_global`

3. Tell git to use global .gitignore file:
`$ git config --global core.excludesFile ~/.gitignore_global`

It will ignore only *new* files. If file was already added then you can remove it by:

`$ git rm --cached .DS_Store`

If you want to check why file is ignored just use:

`$ git check-ignore -v .DS_Store`

## Git Stash

`$ git stash`

Used when you are working on something and then need to switch context to work on something else. It saves your local changes and restores working directory to clean state.

`$ git stash pop`

Gives you back your changes upon returning to your previous work that was interrupted.

## Git history

Git history is complete chain of commit that records every change ever made to a repository.

`$ git history`

Used to view git history.

`$ git checkout <commit-hash>`

Used to go back in time to certain commit. To go back to the latest commit just checkout branch name.

## Undoing commits

`$ git reset`

Used to move current branch to another commit and optionally update the staging area or working directory.

*--soft HEAD~1* - goes back one commit, keeps changes staged, doesn't touch files

*--mixed HEAD~1* - goes back one commit, unstage changes, keeps files changed

*--hard HEAD~1* - goes back one commit, discards staged changes and files changes

When commit is already pushed to the remote to revert it you need to:

`$ git reset <args>`
`$ git push --force`

`$ git commit amend`

Used to replace the most recent commit with a new one.

`$ git revert <commit-hash>`

Used to revert the entire commit by creating a new commit to revert the old commit changes.

## Git merge

`$ git merge <source>`

Used to integrate changes from one branch to another.

## Best practices:

1. Use separate branches for features and/or bugfixes and prefix the branch names with these.
2. In DevOps it is common to work on master branch only and have the whole process set up as CI/CD.
3. After merging feature/bugfix branches into master delete the branches.
4. Avoid force pushing (after reset) on master branches or the ones shared with others.
