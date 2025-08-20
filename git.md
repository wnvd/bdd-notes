## Git Course
Git has high level (porcelain) commands and low level (plumbing)
commands.

The porcelain commands are the ones that you will use most often as
to interact with your code.
```git
git status
git add
git commit
git push
git pull
git log
```
Some plumbing commands (we will mostly do high level commands)
```git
git apply
git commit-tree
git hash-object
```

There are several stages
Untracked -> not being tracked by Git
Staged -> marked for inclusion in the next commit
Committed -> add to the commit history

`git status` shows you the current state of the repo.

A commit is a snapshot of the repository at a given time.

A git repo is list of long list of commits where each commit represents
the full state of the repository at a given point of time.


You can rely on commit up to 7 characters

The commit hashes are unique, there are a lot of things that effect:
- The commit message
- The author's name and email
- The date and time
- Parent (previous) commit hashes

Git is made up of objects that are stored in the `.git/objects` directory.
A commit is just a type of object

NOTE: 
The `--no-pager` flag will print out to the std out and not in some
kind of 'display only program like less'

# Trees and Blobs
- trees -> git's ways of storing directories.
- blob -> git's ways of storing a file.

Last time when we used `git cat-file -p <hash>` we didn't not see the
contents of the file that are related to that commits. That is because
blob object stores it (and has turned it into raw bytes)

You can see the contents of the file in the commit, if we use the 
`git cat-file -p <hash>` on **object hash** then **tree hash**  then
**blob hash** which will give the contents of the commit.

# Storing Data
Git stores an entire snapshot of files on a per-commit level. This was
a surprise to me! I always assumed each commit only stored the changes
made in that commit.

# Optimization
Git compresses and packs files to store them more recently.
Git deduplicates files that are the same across different commits.
If a file doesn't change between commits, Git will only store it once.

# Config
```git
git config --list --local
git config --unset <key-in-config>

# weirdly git allows multiple keys
git config --unset-all <same-key>
```

You can also remove and entire section
```git
git config --remove-section section
```

There are several locations where Git can be configured.
From more general to more specific, they are:

- system: /etc/gitconfig, a file that configures Git for all users
on the system
- global: ~/.gitconfig, a file that configures Git for all projects
of a user
- local: .git/config, a file that configures Git for a specific
project
- worktree: .git/config.worktree, a file that configures Git for
part of a project


You will mostly be using global `--global` or local `--local`

Overriding:
If you set a configuration in a more specific location, it will
override the same configuration in a more general location. For
example, if you set user.name in the local configuration, it will
override the user.name set in the global configuration.

# Branches
A Git branch allows you to keep track of different branches separately

# Git Rename Branch:
`git branch -m oldname newname`

To get default global default branch name
`git config --global --get init.defaultBranch`

# New Branch
There are two ways to create a new branch:

New user friendly
`git branch my_new_brach`

Old way
`git switch -c my_new_branch`


# Switching Branches
You can use `git switch <branch>`(new way) or
`git checkout <branch>` (old way)


# Git Log
`git log` shows you history of commits in your repo
There are some flags that are useful from time to time

The first is `--decorate` which can be one of:
- `short` (default)
- `full` (shows the full ref name)
- `no` (no decoration)

`git log --decorate=full`
You should see that instead of just using your branch name,
it will show the full ref name. A ref is just a pointer to a commit.
All branches are refs, but not all refs are branches.

The Second is `--oneline`. This flag shows  you a more compact
view of the log.

`git log --oneline`
This flag will show you a more compact view of the log.

You can also do something like this
`git log --decorate=full --oneline`

# Git Graph
You can look as a graph of all branches using the following command 
```git
git log --oneline --graph --all
```

```git
git --no-pager log --oneline --decorate --graph --parents  
```
`--parents` in the above commands give you the parent hash of the
commit.

# Fast Forward Merge
This is the simplest type of merge. Git automatically does a fast 
forward merge for you.
It just moves the pointer from the base to the tip of the feature
branch.


With fast forward merge no merge commit is created.

This is a common workflow when working with Git on a team of developers:

- Create a branch for a new change
- Make the change
- Merge the branch back into main (or whatever branch your team dubs
the "default" branch)
- Remove the branch
- Repeat

Fast-Forward merge happens because the branch you are merging to already
has all the changes.

# Delete Branch

`git branch -d <branch-name>`

# Rebase
Rebase is the most controversial command in git.
What is means is that you rebase you divergent branch on top of the 
branch you are rebasing to.

NOTE:
You can create and switch to a new branch also by giving it a HASH
of the commit to branch off of
`git switch -c <branch-name> <COMMITHASH>`

So if we want to rebase to bring changes from main onto a feature
branch, we would run the following command on branch whose base we
want to rebase.

`git rebase main`

This will do the following:
- Checkout the latest commit from `main` into temporary location.
- Replay each commit from the feature branch one at a time onto this
temporary location
- update the feature branch to point to the last replayed commit in
the temporary location. 
- The rebase does not affect the main branch; the feature branch now
includes all the changes from main.


# When to rebase
git rebase and git merge are different tools.

An advantage of merge is that it preserves the true history of the project.
It shows when branches were merged and where. One disadvantage is that it
can create a lot of merge commits, which can make the history harder to read
and understand.

A linear history is generally easier to read, understand, and work with.
Some teams enforce the usage of one or the other on their main branch, but
generally speaking, you'll be able to do whatever you want with your own branches.

# WARNING
You should never rebase a public branch (like main) onto anything else.
Other developers have it checked out, and if you change its history, you'll
cause a lot of problems for them.

However, with your own branch, you can rebase onto other branches
(including a public branch like main) as much as you want.

It's generally okay to merge changes into your own private branch, or to
rebase your branch onto a public branch

# The Reset Soft
The `git reset` command can be used to undo the last commit(s) or any
changes in the index (staged but not committed changes) and worktree
(unstaged and not committed changes)

`git reset --soft <COMMITHASH>`

The `--soft` option is useful if you just want to go back to a
previous commit, but keep all of your changes. Committed changes
will be uncommitted and staged, while uncommitted changes will remain
staged or unstaged as before.

If we don't want to keep the changes we use:
`git reset --hard <COMMITHASH>`
This is useful if you just want to go back to a previous commit and
discard all the changes.

### WARN:
**ALWAYS BE CAREFUL WHEN USING** `--hard`. Things are deleted for 
good.

This will reset your working directory and index to the state of that
commit, and all the changes made after that commit are lost forever.

There are advanced (safe ways) to undo changes.


# Remote
In git, another repo is called a "remote." The standard convention is
that when you're treating the remote as the "authoritative source of
truth" (such as GitHub) you would name it the "origin".

By "authoritative source of truth" we mean that it's the one you and
your team treat as the "true" repo. It's the one that contains the
most up-to-date version of the accepted code.

You can set remote using
`git remote add <name> <url>`

You can get remote URL/URI
`git remote get-url origin`

# Fetch
Adding a remote to our Git repo does not mean that we automatically
have all the contents of the remote.

First, we need to fetch the contents.

`git fetch` 
This downloads copies  of all the contents of the `.git/objects` directory
(and other book keeping information) from the remote repo into your 
current one.

# Not Fetched 
Just because we fetched all of the metadata from the remote repo
doesn't mean we have all the files.

# Log remote
The `git log` command isn't only useful for the you local repo.
You can also log the commits of the remote repo as well

`git log <remote-name>/<branch-name>`

# Merge
Just as we merged branches within a single local repo, we can also
merge branches between local and remote repos.

`git merge <remote-name>/<branch-name>`

NOTE:
You can list remote using following command
`git ls-remote`

# Push
The `git push` command pushes(sends) local changes to any remote.
In our case Github. For example, to push our local main branch's
commits to the remote origin's main branch we would run:

`git push origin main`

Alternative options
You can also push a local branch to a remote with a different name;
`git push origin <local-branch>:<remote-branch>`
This one is less common but nice to know

You can also delete a remote branch by pushing an empty to it:
`git push origin :<remote-branch>`

# Pull
Fetching is nice, but most of the time we want the actual title changes
from a remote repo, not just the metadata.

`git pull [<remote>/<branch>]`

The [...] syntax means that the bracketed remote and branch are optional.
If you execute git pull without anything specified it will pull your
current branch from the remote repo.

# Pull Requests (Important)
On GitHub, a pull request is a way to propose changes, typically to
the rest of your team, or to the maintainer of a project you're
contributing to.

Pull requests allow team members to see what changes are being
proposed and to discuss them before they are merged into the main codebase.
 
You can make a pull request a pushing to a new remote branch to Github.
This could have name same as your local branch.

`git push origin add_classics:add_classics`

# Workflow

You can  config Git to rebase by default on pull, rather than
merge so I keep a linear history. If you want to do the same, you
can add this to your global Git config:

`git config --global pull.rebase true`

# Solo Workflow
When I'm working by myself, I usually stick to a single branch, main.
I mostly use Git on solo projects to keep a backup remotely and to keep 
a history of my changes. I only rarely use separate branches.

- Make changes to files
- git add . (or git add <files> if I only want to add specific files)
- git commit -m "a message describing the changes"
- git push origin main
- It really is that simple for most solo work. git log, git reset, and
some others are, of course, useful from time to time, but the above
is the core of what I do day-to-day.

# Team Workflow
When you're working with a team, Git gets a bit more involved 
(and we'll cover more of this in part 2 of this course). Here's what I do:

- Update my local main branch with git pull origin main
- Checkout a new branch for the changes I want to make with `git switch -c <branchname>`
- Make changes to files
- `git add .`
- `git commit -m "a message describing the changes"`
- `git push origin <branchname>` (I push to the new branch name, not main)
- Open a pull request on GitHub to merge my changes into main
- Ask a team member to review my pull request
- Once approved, click the "Merge" button on GitHub to merge my changes into main


- Delete my feature branch, and repeat with a new branch for the next set of changes

# Nested Gitignore
Your .gitignore file does not necessarily need to be at the root of your project.

It's fairly common to have multiple .gitignore files in different directories
throughout a project. A nested .gitignore file only applies to the directory
it's in and its subdirectories.


# Patterns in gitignore
You can negate a pattern by prefixing it with an exclamation mark (!).
For example, to ignore all .txt files except for important.txt, you could
use the following pattern:
```git
*.txt
!important.txt
```
## Comments
```git
# Ignore all .txt files
*.txt
```
