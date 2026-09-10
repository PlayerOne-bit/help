# GIT Tutorial

Before we begin making an app always start with making version control using [Git](https://git-scm.com/install/)

In GitHub create a [new repository](https://github.com/new):
1. Add your repository name of the app
2. Make the visibility either public or private
3. Add README.md to make a simple description about your app
4. Don't add license when your app is for commercial use (All rights are reserved)

__Note:__ To run npm, git, or any commands in any IDE with terminal...
Go to environment variable, paste the path of the installed command to the user variables.
1. In windows search, `System Properties`
2. `Environment Variables`
3. `Path (User variables)`
4. `Edit`
5. `New`
6. Paste the `Parent folder path of the installed .exe`
7. `OK` everything

If you are new to Git Version Control, run this command:
```git
git config --global user.name "Your username"
git config --global user.email "Your@email.example"
```
Run this command in VS Code Terminal to clone the repository set up:
```git
git clone https://github.com/<Your Username>/<Your Repository Name>
```
As a beginner know this basics of git:
1. To Stage file changes
```git
git add <file-name> <file-name_2> <...>
```
2. To Save files (`-m` means message it is required to commit)
```git
git commit -m "Your message"
```
3. To Push to GitHub Repository
```
git push <>
```
Always do this when you are making major and minor changes so you can revert back when there are errors. For collaboration, you might want to learn this escpecially for example when you are working on a feature or fixing codes:
- `git branch <branch-name>` = create a new branch (add `-d` to delete)
- `git switch <branch-name>` = to switch branch (add `-c` to create and switch)
- `git branch` = to list branches
- `git pull` = __MUST DO__ before you merge or push
- `git merge <branch-name>` = merges that <branch-name> to current branch which is now ready to push (add `--abort` to cancel)
### If you ever have mistakes:
- `git rm <file-name>` = to delete file/s from git and local (add `--cached` to remove on git only)
- `git restore <file-name>` = discard unstaged changes in a file (add `--staged` to undo `git add`)
- `git log` = display past commits
- `git revert <commit-id>` = to undo from one pushed commit
- `git reset HEAD~1 --soft` = to undo commits in local (`HEAD-1` depicts go back 1 commit and replace `--soft` with `--hard` if you want to permanently delete changes)

Now create `.gitignore` file to prevent pushing unnecessary or sensitive files. Just list down "file names" such as `.env`, etc. to avoid leaks.

[Return](javascript:history.back())