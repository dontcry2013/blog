---
title: Git Bisect
date: 2019-12-19 22:38:36
categories: [Tools, Git]
tags:
---

`git bisect` is a powerful tool for quickly determining when a particular bug was introduced into your project.
<!--more-->

 ![git bisect][bisect]

Its principle is very simple, that is, the history of code submission is continuously narrowed down according to the dichotomy. The so-called "dichotomy" is to divide the code history into two, determine whether the problem lies in the first half or the second half, and continue to perform this process until the scope is narrowed to a certain code submission.

This article uses an example to explain how to use this command. Here is a [code base](https://github.com/bradleyboy/bisectercise).

```
$ git clone git@github.com:bradleyboy/bisectercise.git
$ cd bisectercise
```

Open index.html in a browser, there are two buttons, one is to increase the number and the other decrease. But they are acting as a wrong way, we need to fix it. Check the commits history first.

```
$ git log --pretty=oneline
```
We can see there are 101 commits in total, and the very first one is `4d83cf`

## Starts Debugging

The debugging command format like below.

```
$ git bisect start [end_commit] [start_commit]
```
So we use HEAD to indicate the end and `4d83cf` as the start commit.
```
$ git bisect start HEAD 4d83cf
```
After this executed, repo will reset to the 51th commit, which is the middle of all commits.

Test it, find it's OK, so we need to mark it as good.

```
$ git bisect good
```

When you execute the above command, Git automatically switches to the midpoint of the second half (the 76th commit).

Now refresh the browser, click the + button, and find that it can't increase normally. Use the `git bisect bad` command to identify a problem with this submission (p.76).

After executing  `git bisect bad`, Git automatically switches to the midpoint of the 51st to 76th (the 63rd commit).

Next, continue to repeat this process until the successful submission is found. At this point, Git will give the following prompt.

> b47892 is the first bad commit


## Exit

Exit error checking and return to the most recent code submission.
```
$ git bisect reset
```

[bisect]: /img/git-bisect.png
