---
title: Undo commits in Git
date: 2020-04-27T11:23:53.417451Z
description: How to undo the most recent commits in Git
---

One of the most common tasks when using Git is undoing commits on a local branch. Let’s look at how we can do that:

## Undo last commit on the local branch

```
git reset --hard HEAD^
```

For Windows, you can use:

```
git reset --hard HEAD~1
```

This will undo the last commit on the checked out local branch.

_**Note:** If you want to undo the commit but keep the changes around you can remove the `--hard` flag._

## Undo multiple previous commits

```
git reset --hard HEAD~n
```

`n` is the number of comments you want to undo. For instance, if you wanted to undo the last 3 commits you would use the following:

```
git reset --hard HEAD~3
```

But, these commands only change the local branch.

If you have already pushed the commits that you want to undo to the remote repository then you will need to force push the changes to the remote repository after running the above commands to undo the commits.

## Undo previous commits on remote

```
git push origin branch_name --force-with-lease
```

Once the commits are undone locally, you can force push them to the remote repository. _**Force pushing**_ is generally considered bad practice and for the right reasons because it unconditionally overwrites the remote branch with whatever you are pushing from the local branch which can possibly overwrite someone else’s work. Therefore, it's better to use `--force-with-lease` instead. It is a safer version of force because its default behaviour makes sure you do not overwrite changes made by others. You can read further on how it works in [**this blog post by Atlassian Bitbucket**](https://blog.developer.atlassian.com/force-with-lease/).

## Undo using commit ID

Every commit in git has a SHA1 hash value as its ID. You can even choose to undo to a specific commit using its commit ID.

```
git reset --hard commit_id
```

For example:

```
git reset --hard a6dc390
```

This will undo everything back to that specific commit.