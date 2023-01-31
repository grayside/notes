---
title: "GitHub Actions: 'tee' is for Environment Variables"
date: "2023-01-31T10:25:51-08:00"
author: ""
authorTwitter: "" #do not include @
cover: ""
tags: ["til", "gha", "testing", "environment", "shell"]
keywords: ["", ""]
description: ""
showFullContent: false
color: "blue" # ["orange", "blue", "red", "green", "pink"]
---

In 2020 GitHub changed the recommended practice for dynamically setting environment
variables because of a security vulnerability: [GitHub Actions: Deprecating set-env and add-path commands
](https://github.blog/changelog/2020-10-01-github-actions-deprecating-set-env-and-add-path-commands/)

I missed this until I noticed console warnings.

However, sometimes I like to check the environment variable I've just set, and
this model of piping to a file inspired me.

Instead of a simple append operation:

```yaml
name: Set Environment
-  run: echo "MY_VARIABLE=${PWD}/cool/data/path" >> $GITHUB_ENV
```

You can both append and see the result by using `tee -a`:

```yaml
name: Set Environment
-  run: echo "MY_VARIABLE=${PWD}/cool/data/path" | tee -a $GITHUB_ENV
```
