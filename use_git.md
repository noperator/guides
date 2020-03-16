## Helpful resources
- [Git: Fetch and Merge, Don’t Pull](https://longair.net/blog/2009/04/16/git-fetch-and-merge/)
- [Difference between git reset soft, mixed and hard](https://davidzych.com/difference-between-git-reset-soft-mixed-and-hard/)

## Frequently used commands (FUCs)
Roll back a single commit. Previously committed changes are now in the working directory, but not staged. Useful when you try to push, but the updates were rejected because the remote repo contains work that you do not have locally.

```
$ git reset HEAD^
```

Clone a specific branch.
```
$ git clone --single-branch --branch <BRANCH> <REMOTE_REPO>
```

Stop tracking changes to a [*committed* file](https://stackoverflow.com/a/3320183) (and undo it).
```
$ git update-index --assume-unchanged <FILE>
$ git update-index --no-assume-unchanged <FILE>
```

## Avoiding painful merge conflicts during pulls

Sometimes, pulling changes from a remote repository into the current branch can lead to a merge conflict:
```
$ git pull
remote: Enumerating objects: 7, done.
remote: Counting objects: 100% (7/7), done.
remote: Compressing objects: 100% (1/1), done.
remote: Total 4 (delta 3), reused 4 (delta 3), pack-reused 0
Unpacking objects: 100% (4/4), done.
From https://github.com/noperator/dotfiles
   bbb89d4..67b04a6  master     -> origin/master
Auto-merging .bashrc.d/32-git.sh
CONFLICT (content): Merge conflict in .bashrc.d/32-git.sh
Automatic merge failed; fix conflicts and then commit the result.
```

To explain a few of the terms found in the code block above (also explained [here](https://stackoverflow.com/a/18137512)):
- `master` is a local branch
- `origin/master` is a remote branch (i.e., a _local_ (and potentially out of date) representation of the branch "`master`" on the remote repository "`origin`")

The merge conflict above appears during a pull because `git pull` is actually shorthand for `git fetch` followed by `git merge`. Git automatically attempts to merge `master` and `origin/master` even though they've diverged.  A safer way to avoid this merge conflict would be to perform these two actions separately; i.e., `git fetch origin`, resolve any differences between the two branches, and then `git merge origin/master`.

First, fetch changes from `origin`:

```
$ git fetch origin
remote: Enumerating objects: 7, done.
remote: Counting objects: 100% (7/7), done.
remote: Compressing objects: 100% (1/1), done.
remote: Total 4 (delta 3), reused 4 (delta 3), pack-reused 0
Unpacking objects: 100% (4/4), done.
From https://github.com/noperator/dotfiles
   bbb89d4..67b04a6  master     -> origin/master
```

Now that changes are fetched from `origin`, there are a few possible scenarios. Spoiler alert: you can safely merge _unless_ you've made a commit; I spelled all of the possible scenarios out here for my own understanding. Assume the local `master` branch is behind `origin/master`, and:
- There are **no local changes**. You can safely `git merge origin/master`.
  ```
    $ git status
    On branch master
    Your branch is behind 'origin/master' by 1 commit, and can be fast-forwarded.
      (use "git pull" to update your local branch)
  
    nothing to commit, working tree clean
  ```
- There are **uncommitted local changes**. You can safely `git merge origin/master`.
  ```
  $ git status
  On branch master
  Your branch is behind 'origin/master' by 1 commit, and can be fast-forwarded.
    (use "git pull" to update your local branch)
  Changes not staged for commit:
    (use "git add <file>..." to update what will be committed)
    (use "git checkout -- <file>..." to discard changes in working directory)

          modified:   .bashrc.d/32-git.sh

  no changes added to commit (use "git add" and/or "git commit -a")
  ```
- There are **staged local changes** (i.e., added but not committed). You can safely `git merge origin/master`.
  ```
  $ git status
  On branch master
  Your branch is behind 'origin/master' by 1 commit, and can be fast-forwarded.
    (use "git pull" to update your local branch)

  Changes to be committed:
    (use "git reset HEAD <file>..." to unstage)

          modified:   .bashrc.d/32-git.sh
  ```
- There are **committed local changes**. In this case, you can simply `git reset HEAD^` to undo the last commit (i.e., peel it off) and restore the index to the state it was in before that commit, leaving the working directory with the changes uncommitted. Now, merge changes from `origin` and commit again.
  ```
  $ git status
  On branch master
  Your branch and 'origin/master' have diverged,
  and have 1 and 1 different commits each, respectively.
    (use "git pull" to merge the remote branch into yours)

  nothing to commit, working tree clean
  ```

