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
