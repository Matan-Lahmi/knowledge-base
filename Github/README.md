# Git

Commands I actually use. Not a full git tutorial.

## Contents
- [Working Directory / Staging / Local Repo](#working-directory--staging--local-repo)
- [Git vs GitHub](#git-vs-github)
- [git log](#git-log)
- [Remote, Push, Pull](#remote-push-pull)
- [Branches](#branches)
- [.gitignore](#gitignore)
- [Conventional Commits](#conventional-commits)
- [Merge](#merge)
- [Tags & Releases](#tags--releases)
- [Pull Request](#pull-request)

---

## Working Directory / Staging / Local Repo

Three stages a change goes through before it's actually saved in history.

- **Working Directory** - the files as they are right now, on disk
- **Staging Area** - files marked "ready to be committed" (`git add`)
- **Local Repository** - the actual saved history (`git commit`), lives in `.git/`

```bash
git status             # check what state your files are in
git add <file>          # working dir -> staging
git add .                # stage everything
git commit -m "message"   # staging -> local repo
```

---

## Git vs GitHub

- **Git** - the tool itself. Version control, works fully offline, no internet needed.
- **GitHub** - a website that hosts git repos remotely + adds stuff on top (PRs, Actions, Issues). Git existed before GitHub and works without it.

---

## git log

See commit history.

```bash
git log
git log --oneline          # short version, one line per commit
git log -n 5                 # last 5 commits only
```

---

## Remote, Push, Pull

```bash
git remote -v                  # see remotes
git remote add origin <url>      # connect local repo to a remote

git push origin main
git pull origin main             # fetch + merge in one command
```

---

## Branches

```bash
git branch                    # list branches
git checkout -b <name>          # create + switch to new branch
git checkout main                 # switch back
git branch -d <name>                # delete branch
```

---

## .gitignore

File telling git which files/folders to never track (never stage, never commit). One pattern per line.

```
node_modules/
*.log
.env
```

---

## Conventional Commits

Commit message format, so history is readable and consistent.

```
feat: add new endpoint for login
fix: correct wrong port in service.yaml
chore: update dependencies
docs: update README
refactor: simplify deployment logic
```

---

## Merge

Combines changes from one branch into another.

```bash
git checkout main
git merge <branch-name>
```

Resolving complex conflicts visually via VS Code's UI rather than CLI to avoid mistakes.

---

## Tags & Releases

Marks a specific commit as a version, doesn't change over time (unlike a branch).

```bash
git add .
git commit -m "message"
git tag v1.0.0
git push origin main
git push origin v1.0.0
```

Used mainly to trigger GitHub Actions pipelines on specific version.
---

## Pull Request

Way to propose merging one branch into another on GitHub, usually with review before it's merged. Used this to merge into `main` when practicing GitHub Actions.
