# Git

### Clone and change git origin
`git remote -v` : Shows a list of remote repositories

`git remote rm origin` : Removes origin as a remote repository

`git remote add origin <url>` : Adds a remote repository named origin whose url is `<url>`

`git remote rename <old> <new>` : Renames remote repository `<old>` to `<new>`


### Git cherry-pick

Git cherry-picking command let you select a comment by its hash and apply it at the top or your current workign branch.

Let say that we want to pick a commit from a `patch` branch and apply it to `dev` branch

1. We need to get the hash <MY_HASH> of the commit we want ot pick for example using `git log`

2. We want to switch to the branch where we want to apply this commit :
```
git checkout master
```

3. We can now use `git cherry-pick` command to apply the selected commit to master :
```
git cherry-pick <MY_HASH>
```
The commit <MY_HASH> is applied on top of master 💪


### Git revert
