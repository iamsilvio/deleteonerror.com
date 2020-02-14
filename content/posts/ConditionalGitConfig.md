---
title: "Conditional Git Config"
subtitle: "Those who can't use their head must use their ... profile!"
date: 2019-06-18T22:47:01+01:00
draft: false
tags: ["Git", "devEnv"]
categories: [itstuff]
---

You have once again committed with your private mail address in a repo where you should actually use your company profile?  
Does this sound familiar to you?  
I know this only too well, sometimes noticed early enough but mostly my private profile ends up in the commit!  
  
So if we can't manage to remember to use the right config by ourselves, then this must happen automatically.  
  
<!--more-->
We have to change our gitconfig `~\.gitconfig`  

``` ini
[user]
  email = priv-mail@example.com

# on Windows
[includeIf "gitdir/i:p:/cy/**"]
  path = ./.gitconfig.corp
```

The `includeIf` can be set to any directory in your file system, in my case my corp repost are all within the same root folder.
All repositories outside this folder will use my private git profile.  
  
Let's create a secondary git config `~\.gitconfig.corp`  

``` ini
[user]
  email = corp-mail@example.com

```

That's all you need, you can test which profile is used in the current path with ...

``` console
git config --show-origin --get user.email

```

The official documentation can be found [here](https://git-scm.com/docs/git-config#_includes)

Have Fun
