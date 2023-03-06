---
layout: post
title: Git revert commits from fork repository
tags: [git]
comments: true
---
I messed up my fork repository today, and I want to roll back commits so that it updates with the original repository. There are numerous ways to accomplish it; here is one:
#### Make sure you are on master branch
1. git remote add original https://github.com/xxx
2. git fetch original
3. git reset --hard original/master
4. git push --force origin master

Cheer!
