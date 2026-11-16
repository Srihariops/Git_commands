# Git Complete Practical Guide for DevOps

Every command has a `#` comment on the right side that says what it does. The comments use simple words and tell you **where your work moves**.

---

## 0. First learn the 4 areas

```
WORKING AREA  --git add-->  STAGING AREA  --git commit-->  LOCAL REPO  --git push-->  REMOTE REPO
(your files,                (waiting room,                 (commits on                 (GitHub / GitLab,
 you edit here)              next commit list)              your computer)              shared with team)
```

| Command | Moves work | Simple meaning |
|---|---|---|
| `git add` | Working area to Staging area | Pick changes for the next commit |
| `git commit` | Staging area to Local repo | Save the picked changes as a commit |
| `git push` | Local repo to Remote repo | Upload my commits |
| `git fetch` | Remote repo to Local repo | Download only. My files do not change |
| `git pull` | Remote repo to Local repo to Working area | Download and update my files |
| `git clone` | Remote repo to my computer | Download the full project |
| `git restore file` | Local repo to Working area | Throw away my edits |
| `git restore --staged file` | Staging area to Working area | Take file out of waiting room |
| `git reset --soft` | Local repo to Staging area | Undo commit, keep changes staged |
| `git reset --mixed` | Local repo to Working area | Undo commit, keep changes as edits |
| `git reset --hard` | Deletes in all 3 local areas | Undo commit and delete changes |
| `git stash` | Working/Staging area to a hidden drawer | Hide unfinished work |
| `git stash pop` | Drawer to Working area | Bring hidden work back |
| `git switch branch` | Local repo to Working area | My files change to match that branch |

**Why a waiting room?** You changed 10 files, but only 3 belong to this task. Add those 3, commit them. The other 7 stay in the Working area.

**Words you will see**

| Word | Meaning |
|---|---|
| Repo | Project folder plus all its history |
| Commit | One save point with an ID (like `3f9a2c1`), message, author |
| Branch | A separate line of work |
| HEAD | "You are here" (current commit or branch) |
| origin | Default nickname of the remote repo |
| Upstream | The remote branch your local branch is linked to |
| Tracked file | A file Git already knows |
| Untracked file | A new file Git does not know yet |

**The best daily habit:**

```
git status  ->  git diff  ->  git add  ->  git diff --staged  ->  git commit  ->  git push
```

---

## 1. Setup: init, clone, config

### git init: start Git in a new folder

```bash
mkdir payment-api                        # make a new folder
cd payment-api                           # go inside
git init                                 # start tracking this folder. Makes a hidden .git folder (the Local repo)
```

### git clone: download an existing project

```bash
git clone https://github.com/company/payment-api.git                # Remote repo -> my computer, full history
git clone https://github.com/company/payment-api.git my-folder      # download into a folder with my own name
git clone -b development https://github.com/company/payment-api.git # download and start on development branch
git clone --depth 1 https://github.com/company/payment-api.git      # only latest commit, no old history. Fast. Good for CI.
```

Joining a project? Use `clone`. Do not run `git init` inside a cloned repo.

### git config: your name and email (do once)

```bash
git config --global user.name "Srihari"              # my name on every commit, for all repos of my user
git config --global user.email "srihari@company.com" # my email on every commit
git config user.email "srihari@company.com"          # only for THIS repo (no --global). Example: work email only in work repo
git config --system core.editor vim                  # for all users of this machine (needs sudo)
git config --global init.defaultBranch main          # new repos start with branch name main
git config --list                                    # show all settings
git config --get user.email                          # show only the email
git config user.name                                 # show only the name
git config --list --show-origin                      # show each setting and which file it came from. Use when Git picks a wrong email
```

| Level | Flag | Applies to | File |
|---|---|---|---|
| Local | none | Only this repo | `.git/config` |
| Global | `--global` | All repos of my user | `~/.gitconfig` |
| System | `--system` | All users of the machine | `/etc/gitconfig` |

Local beats global, global beats system.

### Connect a local project to GitHub

```bash
git remote add origin https://github.com/company/payment-api.git   # give the GitHub URL the nickname origin
git branch -M main                                                 # rename my current branch to main
git push -u origin main                                            # first upload. Link my main to origin/main
```

---

## 2. Check where you are

### git status

```bash
git status                  # my branch + which files changed, staged, or new
git status -s               # short version
```

Short symbols:

```
 M Dockerfile               # M in the RIGHT column = modified, NOT staged (Working area)
M  Dockerfile               # M in the LEFT column  = modified and staged (Staging area)
A  deployment.yaml          # new file, staged
?? test.txt                 # new file, Git does not track it yet
 D old.yaml                 # deleted, deletion not staged
D  old.yaml                 # deleted, deletion staged
```

### Small info commands

```bash
git branch --show-current       # print only the current branch name
git rev-parse --show-toplevel   # print the root folder of this repo
git rev-parse HEAD              # print the full ID of the current commit
git rev-parse --short HEAD      # print the short ID of the current commit
```

Run `git status` before switching branch, committing, pulling, or cleaning files.

---

## 3. Add, restore, remove, move files

### git add: Working area to Staging area

```bash
git add Dockerfile                  # Working area -> Staging area, one file
git add Dockerfile deployment.yaml  # Working area -> Staging area, two files
git add .                           # Working area -> Staging area, ALL changes in this folder and below (new, modified, deleted)
git add -A                          # Working area -> Staging area, ALL changes in the whole repo
git add -u                          # Working area -> Staging area, only files Git already tracks. New files are skipped
git add '*.yaml'                    # Working area -> Staging area, all .yaml files (keep the quotes)
git add -p                          # Working area -> Staging area, piece by piece. Git asks y (yes) or n (no) for each piece
```

| Command | New files | Modified files | Deleted files |
|---|---|---|---|
| `git add .` | yes | yes | yes (this folder and below) |
| `git add -A` | yes | yes | yes (whole repo) |
| `git add -u` | no | yes | yes |
| `git add file` | that file | that file | that file |

**Real example:** you changed `Dockerfile`, `deployment.yaml`, `notes.txt`. Task is only the Docker image.

```bash
git status                  # see all changed files
git add Dockerfile          # stage only the Dockerfile
git diff --staged           # check: only Dockerfile is in the Staging area
```

### git restore: undo file edits

```bash
git restore Dockerfile                   # Local repo -> Working area: throw away my edits. File goes back to last commit. NO UNDO
git restore .                            # same for all files in this folder and below
git restore --staged Dockerfile          # Staging area -> Working area: take it out of the waiting room. My edits stay
git restore --source=HEAD~1 Dockerfile   # bring back Dockerfile as it was one commit ago
git restore --source=a1b2c3d Dockerfile  # bring back Dockerfile from a specific commit
```

- `git restore file` = throw away my edits.
- `git restore --staged file` = unstage, but keep my edits.

### git rm: delete files

```bash
git rm old-deployment.yaml     # delete the file from my folder AND stage the deletion
git rm -r old-manifests/       # delete a whole folder and stage the deletion
git rm --cached .env           # stop tracking .env in Git, but KEEP the file on my disk
```

If a real password was committed, `git rm --cached` is not enough. The secret is still in old commits. **Change the password.**

### git mv: rename or move files

```bash
git mv old-name.yaml new-name.yaml          # rename and stage it
git mv deployment.yaml k8s/deployment.yaml  # move into k8s folder and stage it
```

---

## 4. git diff: check exactly what changed

Lines starting with `-` were removed. Lines starting with `+` were added.

```bash
git diff                          # Working area vs Staging area: my edits NOT added yet
git diff --staged                 # Staging area vs last commit: what my NEXT commit will contain
git diff --cached                 # same as --staged
git diff HEAD                     # Working area vs last commit: ALL uncommitted changes (staged + not staged)

git diff HEAD~1 HEAD              # commit before the last vs the last commit: what the LAST commit changed
git diff a1b2c3d e4f5a6b          # commit vs commit: compare any two commits
git diff a1b2c3d                  # that old commit vs my Working area: what changed since then
git diff a1b2c3d -- Dockerfile    # same, but only one file

git diff main development         # branch vs branch: what development has that main does not
git diff origin/main              # my branch vs remote main (run git fetch first)

git diff -- Dockerfile            # unstaged changes of one file only
git diff --stat                   # short summary: files and number of lines
git diff --name-only              # only the NAMES of files with unstaged changes
git diff --staged --name-only     # only the NAMES of staged files
git diff --check                  # find whitespace problems (trailing spaces) before committing
```

**Real example:** changed 5 files, want to commit only 2.

```bash
git status                                    # see all changed files
git diff                                      # read the edits
git add Dockerfile deployment.yaml            # stage only these 2
git diff --staged                             # read exactly what will be saved
git commit -m "Update container deployment"   # save only the 2 files
```

---

## 5. git commit: Staging area to Local repo

```bash
git commit -m "Update payment API Dockerfile"    # save staged changes as a commit
git commit                                       # opens editor for a longer message
git commit -am "Update application config"      # -a = add all TRACKED changed files first, then commit. New files are NOT included
git commit --allow-empty -m "Trigger CI"         # commit with no file changes. Used to re-run a pipeline
```

`git commit` does **not** upload. It only saves on your computer.

**Good message:** says what and why, short. "Update Dockerfile base image to node:20". Bad: "changes", "fix", "asdf".

### git commit --amend: fix the LAST commit (only if NOT pushed)

```bash
# Case 1: wrong message
git commit -m "update"                                   # bad message
git commit --amend -m "Update payment API deployment"    # replace the message of the last commit. New commit ID

# Case 2: forgot a file
git add deployment.yaml                  # Working area -> Staging area
git commit --amend --no-edit             # put it INTO the last commit. --no-edit = keep the same message, do not open editor
git show --name-only HEAD                # check: both files are in the last commit
```

- `--amend` = do not make a new commit, change the last one.
- `--no-edit` = do not ask me for a new message, keep the old one.
- The commit gets a new ID. Do not amend commits already pushed and shared.

---

## 6. Read history: log, show, shortlog

```bash
git log                                  # full history, newest first
git log --oneline                        # one line per commit
git log --oneline -5                     # last 5 commits
git log --oneline --graph --all          # all branches as a text tree
git log --all --decorate --oneline --graph   # same, also shows branch and tag names
git log --author="Srihari"               # commits by this author
git log --since="2 weeks ago"            # recent commits only
git log -- Dockerfile                    # commits that changed this file
git log --follow -- Dockerfile           # history of this file, even across renames
git log -p -- Dockerfile                 # history + the actual line changes of this file
git log --stat                           # history + which files changed in each commit
git log -S"proxy_timeout" --oneline      # commits that added or removed this exact text
git log main..feature/login --oneline    # commits on feature/login that main does not have
git log --date=short --pretty=format:"%h %ad %an %s"   # custom: id, date, author, message
```

```bash
git show                          # latest commit and what it changed
git show a1b2c3d                  # one commit (ID from git log)
git show --stat a1b2c3d           # summary of files changed in that commit
git show --name-only a1b2c3d      # only the file names
git show a1b2c3d:Dockerfile       # the Dockerfile exactly as it was in that commit
git show HEAD~2                   # the commit 2 steps before the latest
git show v1.0.0                   # the commit and details of this tag
```

```bash
git shortlog -sn                  # number of commits per person, sorted
```

---

## 7. Branches: branch, switch, checkout

**Rule:** never work on `main`. Make a branch for every task. When you switch branch, the files in your Working area change to match it.

### git branch

```bash
git branch                           # list local branches. * = where I am
git branch -r                        # list remote branches
git branch -a                        # list local + remote branches
git branch -v                        # list branches with their last commit
git branch feature/login             # create a branch, but stay where I am
git branch feature/login a1b2c3d     # create a branch starting from an old commit
git branch -m new-name               # rename the branch I am on
git branch -m feature/login feature/user-login   # rename a specific branch
git branch -M main                   # force rename current branch to main
git branch -d feature/login          # delete. SAFE: refuses if there is unmerged work
git branch -D feature/login          # FORCE delete. No safety check. Can lose work
git branch --merged                  # branches already merged (safe to delete)
git branch --no-merged               # branches with work not merged yet
```

### git switch (modern way)

```bash
git switch development                      # move to a branch
git switch -c feature/payment-api           # create the branch AND move onto it
git switch -c hotfix/login-error main       # create the branch starting from main
git switch -                                # go back to the previous branch
git switch --detach a1b2c3d                 # look at an old commit (detached HEAD)
```

### git checkout (old way, you will still see it)

```bash
git checkout development       # same as git switch development
git checkout -b feature/login  # same as git switch -c feature/login
git checkout -- Dockerfile     # same as git restore Dockerfile
git checkout a1b2c3d           # go to an old commit (detached HEAD)
```

Use `switch` for branches and `restore` for files.

### Detached HEAD

```bash
git switch --detach a1b2c3d     # my files look exactly like that old commit. I can look and test
git switch -                    # done looking, go back to my branch
git switch -c rescue-branch     # I made commits here and want to keep them? Make a branch
```

---

## 8. Remote: link to GitHub / GitLab

```bash
git remote                                        # list remote names
git remote -v                                     # show remote names and URLs
git remote add origin https://github.com/company/payment-api.git   # connect local repo to this URL, nickname origin
git remote set-url origin git@github.com:company/payment-api.git   # change the URL (for example HTTPS to SSH)
git remote remove origin                          # remove the link. Does NOT delete anything on GitHub
git remote show origin                            # details: branches, which local branch tracks which
git ls-remote origin                              # list branches and tags on GitHub without downloading
git ls-remote --heads origin                      # list only branches
```

`origin` is just a nickname.

---

## 9. Get updates: fetch and pull

### git fetch: Remote repo to Local repo (my files do NOT change)

```bash
git fetch origin                          # download new commits from GitHub. Working area untouched
git fetch --all                           # same for all remotes
git fetch --prune                         # also remove names of remote branches deleted on GitHub
git fetch origin development              # download only the development branch

git log --oneline HEAD..origin/development    # after fetch: commits on the server that I do not have
git diff HEAD origin/development              # after fetch: exact line changes from the server
```

### git pull: Remote repo to Local repo to Working area

`git pull` = `git fetch` + merge (or rebase).

```bash
git pull                              # fetch + integrate from the linked upstream branch
git pull origin development           # fetch origin/development and integrate it into my CURRENT branch
git pull --no-rebase origin main      # fetch + MERGE (the default, written clearly)
git pull --rebase                     # fetch + put MY commits on top of the new ones (straight history)
git pull --ff-only                    # update only if Git can simply move forward. If not, stop. Safest
# fatal: Not possible to fast-forward, aborting.   <- histories split. Decide: merge or rebase
```

| I want to... | Use |
|---|---|
| Only see what is new on GitHub | `git fetch` |
| Update my branch | `git pull` |
| Safe update of a shared branch | `git pull --ff-only` |

---

## 10. git push: Local repo to Remote repo

```bash
git push                                     # upload current branch to its upstream
git push origin development                  # upload my development branch to origin/development
git push -u origin feature/payment-api       # first push of a new branch. -u links it. After this plain git push works
git push --all origin                        # push ALL local branches
git push origin v1.0.0                       # push one tag
git push origin --tags                       # push all tags
git push origin --delete feature/payment-api # delete a branch on GitHub
git push --force-with-lease                  # overwrite remote history ONLY if nobody pushed something I have not seen. Safer
git push --force                             # overwrite remote history no matter what. Dangerous
```

After the push, open a **Pull Request** (GitHub) or **Merge Request** (GitLab). Team reviews, CI runs.

Use force only on your own feature branch (after `rebase` or `amend`). Never on `main`.

---

## 11. git merge: join another branch INTO my current branch

```bash
git switch development                    # stand on the branch that should RECEIVE the changes
git pull --ff-only origin development     # update it first
git merge feature/login                   # bring feature/login into development
git merge --no-ff feature/login           # always make a merge commit, so history shows "feature merged here"
git merge --ff-only feature/login         # merge only if Git can simply move forward (no merge commit)
git merge --abort                         # cancel the merge, go back to before
git merge --continue                      # finish the merge after fixing conflicts
```

### Merge conflict: same line changed by two people

```
<<<<<<< HEAD
image: payment-api:v1            <- MY version (current branch)
=======
image: payment-api:v2            <- THEIR version (branch being merged)
>>>>>>> feature/update-image
```

```bash
git status                       # which files are in conflict?
vim deployment.yaml              # open it, keep the correct final content, DELETE the 3 marker lines (<<<<<<<, =======, >>>>>>>)
git diff                         # review the fixed file
git add deployment.yaml          # Working area -> Staging area. Tells Git: "this file is fixed"
git merge --continue             # finish the merge (or: git commit)
git merge --abort                # panic? cancel everything
```

If it happened in a rebase: `git rebase --continue` or `git rebase --abort`.
Do not blindly pick "ours" or "theirs" in production config. Read both and keep the correct one.

---

## 12. git rebase: put my commits on top of the newest code

```bash
git fetch origin                        # download newest info
git switch feature/login                # go to my branch
git rebase origin/development           # lift my commits, place them after the newest development commits
```

If conflict:

```bash
git status                              # which file?
git add Dockerfile                      # Working area -> Staging area, after fixing
git rebase --continue                   # go on to the next commit
git rebase --skip                       # skip the commit causing the conflict (only if you do not want it)
git rebase --abort                      # cancel, go back to before
git push --force-with-lease             # after rebase the history changed, push needs this (only on MY branch)
```

### Interactive rebase: clean my last commits

```bash
git rebase -i HEAD~4                    # -i = interactive. HEAD~4 = last 4 commits. An editor opens
```

```
pick   1b2c3d4 wip                      # pick   = keep the commit
squash a7f8e9d fix again                # squash = join into the commit above, edit combined message
fixup  d4c5b6a typo                     # fixup  = join into the commit above, throw away this message
reword e1a2b3c done                     # reword = keep changes, change the message
drop   9f8e7d6 test                     # drop   = delete the commit
```

| Situation | Use |
|---|---|
| Update my own private branch | `rebase` |
| Branch shared with others | `merge` |
| Not sure | `merge` (the safe one) |

Rebase changes commit IDs. Never rebase shared commits.

---

## 13. git cherry-pick: copy ONE commit to another branch

**Real use:** bug fix is on `development`, production needs only that fix.

```bash
git log --oneline development            # find the commit ID of the fix
git switch main                          # go to the branch that needs the fix
git cherry-pick a1b2c3d                  # copy that one commit here (new commit, new ID)
git cherry-pick a1b2c3d e4f5a6b          # copy two commits, in this order
git cherry-pick --continue               # after a conflict: fix, git add, then this
git cherry-pick --abort                  # cancel
```

---

## 14. Undo: pick the right command

| My problem | Where | Use |
|---|---|---|
| Edited a file, want old version | Working area | `git restore file` |
| Ran `git add` on wrong file | Staging area | `git restore --staged file` |
| Last commit has wrong message or missing file (not pushed) | Local repo | `git commit --amend` |
| Undo a local commit (not pushed) | Local repo | `git reset` |
| Undo a commit already pushed | Remote repo | `git revert` |
| Lost commits | Local repo | `git reflog` |
| Junk new files | Working area | `git clean` |

### git reset: move my branch back (NOT pushed commits only)

```bash
git log --oneline -3                # 7be10d4 Add test file  <- I want to undo this
git reset --soft HEAD~1             # Local repo -> Staging area: commit removed, changes STAGED
git reset --mixed HEAD~1            # Local repo -> Working area: commit removed, changes kept as edits (default)
git reset --hard HEAD~1             # commit removed AND changes DELETED. Dangerous
git reset --hard a1b2c3d            # go back to this commit, delete everything after it
git reset HEAD Dockerfile           # unstage one file (older way of restore --staged)
```

`HEAD~1` = one commit back. `HEAD~3` = three commits back.

| Mode | Commit removed? | My changes |
|---|---|---|
| `--soft` | yes | kept, staged |
| `--mixed` | yes | kept, not staged |
| `--hard` | yes | deleted from tracked files |

### git revert: undo a PUSHED commit safely

```bash
git revert a1b2c3d                  # NEW commit that does the opposite. Old commit stays in history
git revert HEAD                     # revert the latest commit
git revert --no-commit a1b2c3d      # prepare the undo, let me review, commit later
git push                            # upload the revert commit
```

- `reset` **erases** history. Only for commits nobody else has.
- `revert` **adds** to history. Safe for shared branches.

---

## 15. git stash: hide unfinished work

**What stash means:** keep your unfinished changes safely aside, like a drawer. You work on something else. Later you take them back.

**Situation:** you edit `login.js` and `login.css` on `feature/login`. Not finished. Manager says: production is down, fix now. You cannot commit half work and you cannot switch branch with it.

```bash
git status                                    # see the unfinished changes
git stash push -m "unfinished login feature"  # Working area -> drawer, with a label. Folder is clean now
git switch main                               # now I can switch freely
git switch -c hotfix/login-crash              # branch for the fix
# ... fix, git add, git commit, git push, deploy ...
git switch feature/login                      # back to my branch
git stash list                                # stash@{0}: On feature/login: unfinished login feature
git stash pop                                 # drawer -> Working area. And remove it from the drawer
```

All stash commands:

```bash
git stash                          # hide tracked changes, default message
git stash push -m "message"        # hide with a label
git stash -u                       # also hide NEW (untracked) files. Without -u they stay in the folder
git stash -a                       # also hide ignored files. Careful: build output too
git stash list                     # show all stashes
git stash show                     # summary of the latest stash
git stash show -p                  # exact line changes in the latest stash
git stash apply                    # bring back latest stash, KEEP it in the drawer
git stash apply stash@{1}          # bring back the second stash
git stash pop                      # bring back latest stash and REMOVE it from the drawer
git stash pop stash@{1}            # bring back a specific one and remove it
git stash drop stash@{0}           # delete one stash
git stash clear                    # delete ALL stashes
git stash branch feature/recover-login stash@{0}   # new branch from the stash's commit, apply it, drop the stash
```

| Command | Brings work back | Keeps copy in drawer |
|---|---|---|
| `git stash pop` | yes | no |
| `git stash apply` | yes | yes |

Stash is temporary. Commit important work when ready.

---

## 16. Detective: who changed what

```bash
git blame Dockerfile                  # who last changed each line, and in which commit
git blame -L 10,30 Dockerfile         # only lines 10 to 30
git show a1b2c3d                      # open that commit and read WHY it changed

git grep "containerPort"              # search text in all tracked files (Ctrl+F for the whole project)
git grep -n "image:"                  # also show line numbers
git grep -i "password"                # ignore upper/lower case. Do not share real secrets you find
```

### git bisect: find the commit that broke the app

```bash
git bisect start                      # begin
git bisect bad                        # current version is broken
git bisect good v1.0.0                # this old version worked
# Git jumps to a middle commit. Test your app, then tell Git:
git bisect good                       # this one works
git bisect bad                        # this one is broken
# ... repeat. After about 7 tests (for 80 commits): "a1b2c3d is the first bad commit"
git bisect reset                      # finished, go back to where I started
```

---

## 17. git clean: delete untracked files

```bash
git clean -n         # DRY RUN: only show files that WOULD be deleted. Always run first
git clean -fdn       # dry run for files and folders
git clean -f         # delete untracked files
git clean -fd        # delete untracked files AND folders
git clean -fdx       # also delete IGNORED files (.env, build output). Very dangerous
```

No trash bin. Deleted files are gone.

---

## 18. .gitignore: files Git must skip

Make a file named `.gitignore` in the repo root:

```
# Local environment values, never commit secrets
.env
.env.*

# Dependencies
node_modules/

# Build output
dist/
build/

# Logs
*.log

# Editor settings
.vscode/
```

```bash
git status                       # check the files are now ignored
git check-ignore -v .env         # show which rule ignores .env and where it is written
git add .gitignore               # Working area -> Staging area, so the team gets the rules
```

`.gitignore` does NOT stop tracking a file Git already tracks:

```bash
git rm --cached .env             # Git forgets .env, file stays on my disk
git add .gitignore               # stage the ignore rule
git commit -m "Stop tracking local environment file"   # save
```

---

## 19. Tags: name a release

```bash
git tag                                          # list tags
git tag v1.0.0                                   # simple tag on the current commit
git tag -a v1.0.0 -m "Production release 1.0.0"  # annotated tag with message, author, date
git show v1.0.0                                  # see the tag and its commit
git push origin v1.0.0                           # Local repo -> Remote repo, one tag. Tags are NOT pushed automatically
git push origin --tags                           # push all tags
git tag -d v1.0.0                                # delete tag on my computer
git push origin --delete v1.0.0                  # delete tag on GitHub
git describe --tags                              # nearest tag, like v1.0.0-5-ga1b2c3d (5 commits after v1.0.0)
```

The pipeline can deploy when it sees a new tag. If `v1.3.0` has a bug, redeploy `v1.2.0` to roll back.

---

## 20. Recover work and check the repo

```bash
git reflog                                # Git's diary: every move HEAD made on my computer
git reflog --date=local                   # same, with local date and time
git show a1b2c3d                          # inspect the lost commit
git branch recovery/save-work a1b2c3d     # make a branch pointing to it. Safe way to recover
git reset --hard a1b2c3d                  # or jump back to it (check git status first)
```

Reflog saves **commits** only. Edits you never committed cannot be recovered.

```bash
git fsck --full                    # check the repo is not corrupted (troubleshooting only)
git ls-files                       # list all files Git tracks
git ls-files '*.yaml'              # list tracked .yaml files
git show-branch                    # show how branches relate
git help commit                    # open the manual of any command
```

---

## 21. Real DevOps flows

### Flow 1: Feature work, start to finish

```bash
cd payment-api                                       # 1. go to the project
git status                                           # 2. any unfinished work?
git branch --show-current                            # 3. which branch am I on?
git fetch origin                                     # 4. download latest info (my files do not change)
git switch development                               # 5. go to the team branch
git pull --ff-only origin development                # 6. Remote repo -> my computer
git switch -c feature/update-payment-deployment      # 7. my own branch
vim Dockerfile                                       # 8. edit the files
vim deployment.yaml
git status                                           # 9. what changed?
git diff                                             # 10. read my edits
git add Dockerfile deployment.yaml                   # 11. Working area -> Staging area
git diff --staged                                    # 12. check what will be saved
git commit -m "Update payment deployment image"      # 13. Staging area -> Local repo
git fetch origin                                     # 14. any news from the team?
git rebase origin/development                        # 15. my commits on top of the latest
git push -u origin feature/update-payment-deployment # 16. Local repo -> Remote repo. Then open a Pull Request
# ... review, checks, merge on GitHub ...
git switch development                               # 17. back to the team branch
git pull --ff-only origin development                # 18. get the merged code
git branch -d feature/update-payment-deployment      # 19. delete my local branch
git push origin --delete feature/update-payment-deployment  # 20. delete it on GitHub
git fetch --prune                                    # 21. clean old remote branch names
```

### Flow 2: Production bug while I am busy

```bash
git stash push -u -m "wip healthcheck"     # hide my unfinished work (also new files)
git switch main                            # go to production code
git pull --ff-only origin main             # get latest
git switch -c hotfix/db-timeout            # branch for the fix
git add config.yaml                        # Working area -> Staging area
git commit -m "Fix database timeout to 30s"   # Staging area -> Local repo
git push -u origin hotfix/db-timeout       # upload, open PR, merge, deploy
git switch development                     # now copy the fix to development
git cherry-pick 9c8d7e6                    # copy only that commit (ID from git log on main)
git switch feature/healthcheck             # back to my work
git stash pop                              # bring my hidden work back
```

### Flow 3: A bad change was pushed to main

```bash
git switch main                            # shared branch
git pull --ff-only origin main             # latest remote history
git log --oneline -5                       # find the bad commit
git show a1b2c3d                           # read it before undoing
git revert a1b2c3d                         # NEW commit that undoes it
git push origin main                       # upload the revert. Pipeline deploys the fix
```

Do not use `reset --hard` + force push for this. It rewrites shared history.

### Flow 4: I committed `.env` by mistake

```bash
git rm --cached .env                       # stop tracking, keep the file on disk
echo ".env" >> .gitignore                  # never add it again
git add .gitignore                         # Working area -> Staging area
git commit -m "Stop tracking .env"         # Staging area -> Local repo
git push                                   # upload
# IMPORTANT: the password is still in old commits. CHANGE THE PASSWORD NOW.
```

### Flow 5: Before any risky command

```bash
git status                                 # do I have local work?
git log --oneline -5                       # check recent commits
git stash push -u -m "backup before risky operation"   # optional backup of tracked + new files
```

---

## 22. How teams use branches

| Style | How it works | Good for |
|---|---|---|
| GitHub Flow | `main` always deployable. One branch per task, Pull Request, merge, deploy | Most web apps |
| Git Flow | `main` (releases), `development` (new work), `feature/*`, `release/*`, `hotfix/*` | Versioned software with a testing phase |
| Trunk-based | Everyone merges into `main` daily with small changes and automatic tests | Teams with strong CI/CD |

| Merge button on a Pull Request | Result |
|---|---|
| Merge commit | Keeps all commits, adds one merge commit |
| Squash and merge | All your commits become 1 commit on `main` |
| Rebase and merge | Commits go on `main` in a straight line |

Branch names: `feature/login-page`, `bugfix/memory-leak`, `hotfix/prod-crash`.

---

## 23. Quick finder: I want to...

| I want to... | Command |
|---|---|
| See what is going on | `git status` / `git status -s` |
| See my current branch | `git branch --show-current` |
| See edits not yet added | `git diff` |
| See what my next commit has | `git diff --staged` |
| See what the last commit changed | `git diff HEAD~1 HEAD` |
| Compare two commits | `git diff <id1> <id2>` |
| Compare two branches | `git diff main development` |
| Move one file to waiting room | `git add file` |
| Move all changes in this folder | `git add .` |
| Move all changes in the repo | `git add -A` |
| Save to my local repo | `git commit -m "message"` |
| Fix my last commit | `git commit --amend` |
| Upload new branch | `git push -u origin branch` |
| Upload commits | `git push` |
| Download only | `git fetch origin` |
| Download and update | `git pull --ff-only` |
| New branch and move onto it | `git switch -c branch` |
| Join a branch into current | `git merge branch` |
| Hide unfinished work | `git stash push -m "msg"` |
| Bring it back and remove from drawer | `git stash pop` |
| Bring it back and keep in drawer | `git stash apply` |
| Copy one commit to another branch | `git cherry-pick <id>` |
| Throw away file edits | `git restore file` |
| Unstage a file | `git restore --staged file` |
| Undo a local commit, keep changes | `git reset --soft HEAD~1` |
| Undo a pushed commit | `git revert <id>` |
| Find lost commits | `git reflog` |
| Find who changed a line | `git blame file` |
| Search text in project | `git grep "text"` |
| Find the commit that broke things | `git bisect` |
| Preview junk file deletion | `git clean -n` |
| Mark a release | `git tag -a v1.0.0 -m "msg"` |

---

## 24. Dangerous commands: be careful

| Command | Why |
|---|---|
| `git reset --hard` | Deletes local changes and moves the branch back |
| `git clean -fd` | Deletes untracked files and folders. No undo |
| `git clean -fdx` | Also deletes ignored files like `.env` and build output |
| `git push --force` | Can replace remote history and delete teammates' commits |
| `git branch -D name` | Deletes a branch even with unmerged commits |
| `git stash clear` | Deletes every stash |
| `git rebase` | Changes commit IDs. Never on shared commits |
| `git commit --amend` | Changes the last commit's ID. Never after pushing |
| `git restore file` | Throws away edits with no undo |

---

## 25. Final rules

1. Run `git status` before every important command.
2. Never work directly on `main`. Always use a branch.
3. Never `amend`, `reset`, `rebase` or force push commits other people already have.
4. Preview first: `git diff`, `git clean -n`.
5. Practice `reset`, `rebase` and `reflog` in a test repo, not a real project.
6. If a secret was committed, remove it AND change it.

**Best daily flow:**

```
status -> diff -> add -> diff --staged -> commit -> push
```
