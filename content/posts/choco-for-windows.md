---
title: "Choco for Windows"
date: 2020-07-09T10:11:16+08:00
tags: ["devops"]
draft: false
---

I recently got a new computer [Dell Precision 5540](https://www.dell.com/en-sg/work/shop/workstations/precision-5540-mobile-workstation/spd/precision-15-5540-laptop) so I needed to re-install all the applications I use.

Typically I would go site to site downloading installation packages and then clicking through various "OK" and "I agree" buttons.  However I stumbled across [Chocolatey](https://chocolatey.org/) which is a package manager for Windows - similar to something like apt-get or pip.

My experience so far has been very positive.  I've been able to install VS Code, Miniconda3, Notepad++, Hugo, Firefox and a few other applications with no issue.

Installation syntax is very simple and straight-forward:  
`choco install firefox`

You can choose a specific installation version with:  
`choco install hugo --version 0.64.3`

You can find packages available for installation with:  
`choco find firefox`

You can also see what packages are outdated (i.e. could be updated) with:  
`choco outdated`

Finally you can choose to "pin" installations to a certain version to prevent them from being updated in the future:  
`choco pin add -n hugo`