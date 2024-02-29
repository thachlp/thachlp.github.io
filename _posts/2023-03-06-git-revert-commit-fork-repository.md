---
layout: post
title: Git revert commits from fork repository
tags: [Git]
comments: true
---
Sometimes, while working on a forked repository, you might find yourself needing to revert your repository to match the upstream repository (the original repository from which you forked). Whether you made a mistake or just need to synchronize with the latest changes, you can do this without losing your contributions on GitHub. Here's a step-by-step guide to help you through the process.

#### Step 1: ensure you are on the main/master branch
#### Step 2: add the original repository as a remote
```bash
git remote add original https://github.com/thachlp/centraldogma.git
```
#### Step 3: fetch the latest changes
```bash
git fetch original
```
#### Step 4: reset your local branch to match the original repository
```bash
git reset --hard original/main
```
Be cautious with this step, as it will overwrite your local changes in the current branch. Ensure you have committed or stashed any work you want to keep before executing this command.
#### Step 5: force push to your fork
```bash
git push --force origin main
```
This step will overwrite the hisotry on your forked repository's remote branch.
