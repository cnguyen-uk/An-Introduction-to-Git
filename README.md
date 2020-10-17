# An Introduction to Git
*A compact introduction to using Git.*

Git is a software that allows for distributed version control, whilst GitHub is a cloud-based platform built around Git. Understanding how to use Git is not required to use GitHub, but one of the most popular uses of GitHub is as a remote backup repository for local Git repositories.

Although there are no programming prerequisites required to use Git, the typical Git user will be using it alongside programming projects. In any case, it can be useful to know some file system navigation and file system manipulation [Bash commands](https://github.com/cnguyen-uk/An-Introduction-to-Bash).

Throughout, note that file or directory arguments for commands can also be replaced with absolute or relative paths instead. Note also that may run commands on multiple filenames in a single line, or on all files by using `.` in the place of a filename.

## Table of Contents

- [Basic Git Workflow](#basic-git-workflow)
- [Getting Started](#getting-started)
- [Backtracking in Git](#backtracking-in-git)
  * [The `HEAD` Commit](#the-head-commit)
  * [Double Dashes](#double-dashes)
  * [Discarding Changes](#discarding-changes)
  * [Unstaging Files](#unstaging-files)
  * [Commit Reverts](#commit-reverts)
  * [Reverting Commit Reverts](#reverting-commit-reverts)
- [Git Branching](#git-branching)
  * [Creating Branches](#creating-branches)
  * [Merging Branches](#merging-branches)
  * [Deleting Branches](#deleting-branches)
- [Collaborative Development with Git](#collaborative-development-with-git)
  * [Basic Collaborative Git Workflow](#basic-collaborative-git-workflow)
  * [The `fetch` and `push` Commands](#the-fetch-and-push-commands)

## Basic Git Workflow

The general Git workflow is as follows:

1. Working Directory: This is the workspace for making changes to files, such as additions, deletions and modifications. Using `git init` can create a repository in this workspace.
2. Staging Area: This acts as a layer between the working directory and the repository. Using `git add` we can add changes from the working directory to the staging area, where files are ready to be committed to the repository.
3. Repository: This is where different versions of the project are saved.	Using `git commit` saves changes in the staging area to the repository.

## Getting Started

Once we have navigated to where we wish to work (using the command line), we need to initialise an empty Git repository in the current directory.

```Bash
git init
```

During a project we may wish to check on the status of the working directory, including any files which are currently untracked.

```Bash
git status
```

To add files to the staging area, i.e. to track files, we can use the following syntax:

```Bash
git add filename
```

If a file is modified in the working directory, then it does not affect the same file (with possibly different contents) in the staging area. Often, we will want to see the difference between the file contents before adding it to the staging area.

```Bash
git diff filename
```

Finally, we can permanently store the changes in the staging area to the repository. Typically, we use the `-m` flag to attach messages to the commit explaining what the commit does. The standard convention for commit messages is that they must be in quotation marks; be written in the present tense; and be 50 characters or less.

```Bash
git commit -m "Implement first part of project"
```

We can view earlier versions of a project using the chronologically stored history of commits in a repository, including a timestamp, the author, the commit hash, and the commit message.

```Bash
git log
```

## Backtracking in Git

Sometimes we will want to revert changes that have been made in a project.

### The `HEAD` Commit

It is often useful to know the commit that the project is currently on (often this will be the commit that has most recently been made). This is called the `HEAD` commit. We can use the following command to output the log and the file changes for the `HEAD` commit:

```Bash
git show HEAD
```

### Double Dashes

It may be the case that branches, files, or even commits, share the same names. To provide clear disambiguation, Git uses `--`. In particular, the argument before the `--` must be a branch name or commit name, and the argument after the `--` must be a filename.

Wherever appropriate, if a command allows for a filename, branch name, or commit, as argument, it should be assumed that `--` can be utilised to avoid issues in coinciding names. Note that names of branches, files, or commits can be blank to indicate that we do not want to have them as an argument. Hence, all of the following syntaxes are valid:

```Bash
git command branch_name -- filename
```

```Bash
git command -- filename
```

```Bash
git command branch_name --
```

### Discarding Changes

We know already know that we can use `git diff` to see changes made to files in the working directory before adding them to the staging area. We may then wish to discard those changes.

```Bash
git checkout filename
```

Sometimes there are conflicts which need to be resolved (such as unmerged changes), which will result in the above syntax not working. In this case we will need to specify the position on the current branch (usually `HEAD`, or `HEAD~X` for a commit which is X commits before the `HEAD` commit).

```Bash
git checkout HEAD filename
```

### Unstaging Files

If we've added a file to the staging area by mistake and want to unstage back to the working directory, without discarding changes, then we can use the following syntax:

```Bash
git reset HEAD filename
```

### Commit Reverts

We've already seen that `git log` outputs commit information, including the commit hash. We can use the first seven characters of the hash to revert to a previous commit.

```Bash
git reset commit_hash
```

Note that running `git log` again will reveal that the `HEAD` commit has been reassigned to the commit that we've reverted to - all the other commits that came after are gone.

### Reverting Commit Reverts

If we wish to undo a recent revert, then we can use the following command:

```Bash
git reset HEAD@{1}
```

This works because Git keeps a log of all reference updates, viewable using the `git reflog` command which tracks the `HEAD` changes and how many positions ago. In general, we may revert to X positions ago.

```Bash
git reset HEAD@{X}
```

## Git Branching

Git allows for the creation of *branches* to experiment with multiple versions of a project. The default branch is usually called `master`. The other branches will not affect the `master` branch until they are merged. Branches consist of commits and can be created from any commit, either from the `master` branch, or any other branch.

To see our branches, we can use the following command:

```Bash
git branch
```

The `*` indicates the branch that we are on.

### Creating Branches

We can create a new branch using the following syntax:

```Bash
git branch branch_name
```

We can also switch branches using either of the following syntaxes (the latter was introduced in Git 2.23, Q3 2019, to avoid confusion):

```Bash
git checkout branch_name
```

```Bash
git switch branch_name
```

### Merging Branches

Merging branches generally falls into two scenarios, depending on if the `master` branch has been altered.

Suppose that we've created a new branch called `new_branch`, have made changes to it, and want to merge it with the `master` branch. In this case, the `master` branch is called the *receiver* branch, and the `new_branch` branch is called the *giver* branch.

If the `master` branch has not been altered, then we can merge the `new_branch` branch easily. First, we should ensure that we are on the `master` branch - in general, we should be on the branch that we wish to merge other branches with. Then we can use the following syntax:

```Bash
git merge new_branch
```

This merge is called a *fast forward* since the `new_branch` branch contains the most recent commit, and hence results in the `master` branch being fast forwarded to be up to date with the `new_branch` branch.

If the `master` branch has been altered however, then Git may not know which changes to keep if the alterations have been performed on the same files and lines as in the `new_branch` branch. This is called a *merge conflict*.

In this scenario, Git will leave conflict markers in the conflicting file(s) to be resolved by hand in the working directory. For any given conflicting file, the conflicting line(s) present in the `master` branch will be between the lines beginning `<<<<<<<` and `======`. The line(s) present in the `new_branch` branch will be between the lines beginning `======` and `>>>>>>>`. Once the conflicts have been resolved, we can use `git add` and `git commit` as usual on the conflicting file(s).

### Deleting Branches
Once branches have been merged with the `master` branch, they are typically no longer needed. The following syntax can be used to delete such branches:

```Bash
git branch -d branch_name
```

## Collaborative Development with Git
Another popular usage of Git is as a collaborative tool. In order to collaborate, project members need:

* A local copy of the project.
* A way to track and review other project members' work.
* Access to a definitive version of the project.

All of this can be achieved via *remotes* - a shared Git repository for multiple collaborators. These can exist locally, on a shared network, or on a website such as GitHub. With remotes, collaborators can work independently and then merge changes together when ready.

Before we can begin, we need to create a local copy of the remote.

```Bash
git clone remote_location clone_name
```

After the remote has been cloned, Git gives the remote address the name origin for convenience, although this can be changed. The following command can be used to see a list of a Git project's remotes:

```Bash
git remote -v
```

### Basic Collaborative Git Workflow

The typical collaborative Git workflow is as follows:

1. Fetch and merge changes from the remote.
2. Create a branch to work on a new project feature.
3. Develop the feature on your branch and commit your work.
4. Fetch and merge from the remote again (in case new commits were made during the previous steps).
5. Push your branch up to the remote for review.

Note that steps 1 and 4 safeguard against merge conflicts.

### The `fetch` and `push` Commands

Any time which elapses from the initial cloning period introduces the possibility of changes being introduced to the remote. Instead of cloning the repository again, we can instead bring those changes onto a local *remote branch*.

```Bash
git fetch
```

Once we have fetched any changes, we need to merge the remote branch, typically named `origin/master`, to our local `master` branch.

```Bash
git merge origin/master
```

Once our work is ready to be shared with the other collaborators, we can push our local branch up to the remote, `origin`. From there, people can review the branch and merge the work into the `master` branch, making it a part of the definitive project version.

```Bash
git push origin branch_name
```
