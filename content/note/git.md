---
title: git
date: 2015-05-26 00:00:00 Z
---

* Fetch bare repository

http://stackoverflow.com/questions/10696718/git-fetch-fails-to-work-on-bare-repo-but-git-pull-works-on-normal-repo
http://stackoverflow.com/questions/7274697/how-do-i-pull-to-a-bare-repository

    git config remote.origin.fetch +refs/heads/*:refs/heads/*
    git fetch
