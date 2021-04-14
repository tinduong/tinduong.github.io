---
title: Save your day
tags: [live-saver,day-saver,hair-saver]
style: fill
color: primary
description: A list of tip that save you from wasting your time.
---


## This command will save your life ##

dotnet tool install -g --ignore-failed-sources [package]

## Keep your local clean

If you have not clean your local repo for a while and you work on a project with many branches created. It's time to clean this up. I mean NOW...but Do you delete branches 1 by 1. It will work but will take forever if you have more than a few branches...

Here a quick solution for window:
```git
git checkout develop; git remote update origin --prune; git branch -vv | Select-String -Pattern ": gone]" | % { $_.toString().Trim().Split(" ")[0]} | % {git branch -D $_}
```

