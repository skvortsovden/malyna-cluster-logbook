# GIT

## Remote repository has been renamed

How to rename local repository to match the remote repository name?

Check remote repository name:

```bash
git remote -v
```

If the remote repository has been renamed, you can update the remote URL:

```bash
git remote set-url origin <new-url>
```


## migrate local repository to new remote repository

my local repository is set to **github** and I want to migrate it to **gitlab**

1. create a new repository on https://gitlab.com.
2. update the remote URL of your local repository to point to the new GitLab repository:

```bash
git remote set-url origin <new-gitlab-url>
``` 
3. push your local repository to the new GitLab repository:

```bash
git push -u origin --all
```