---
layout: post
title:  "SSH File Permissions"
categories: tech
tags:  Linux SSH
excerpt: "If you cannot ssh localhost, check the permission of the files"
---

* content
{:toc}


## For security

It takes me and my colleague some minutes to figure out why our machines cannot ssh their own localhost. We added `-v` to `ssh` for debugging and found that the id_rsa is offered but ignored.
Finally, we checked and changed the file permissions. Then the problem got solved.
The following is a set of correct settings.


## Permissions and commands

Generate keys:
```bash
ssh-keygen -t rsa -b 4096 -N '' -C "yourname@example.com" -f ~/.ssh/id_rsa
```

Add keys:
```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa
```

Change file permissions:
```bash
chmod 700 ~/.ssh
chmod 644 ~/.ssh/authorized_keys
chmod 644 ~/.ssh/known_hosts
chmod 644 ~/.ssh/config
chmod 600 ~/.ssh/id_rsa
chmod 644 ~/.ssh/id_rsa.pub
```