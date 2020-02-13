---
title: "Conditional Git Config"
date: 2019-06-18T22:47:01+01:00
draft: true
tags: ["Git", "devEnv"]
categories: [itstuff]
---

TODO: Intorduction

Your global git config `.gitconfig`

``` ini
[user]
  email = priv-mail@example.com

# on Windows
[includeIf "gitdir/i:p:/cy/**"]
  path = ./.gitconfig.corp
```

Your secondary git config `.gitconfig.corp`

``` ini
[user]
  email = corp-mail@example.com

```

you can test if the right config value ...

``` console
git config --show-origin --get user.email

```

The official documentation can be found [here](https://git-scm.com/docs/git-config#_includes)
