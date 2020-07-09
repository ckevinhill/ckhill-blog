---
title: Site setup using Hugo, Azure Blob and GitHub Actions
date: 2020-07-08T13:35:18+08:00
tags: ["azure", "tutorial", "hugo", "devops"]
---

I've launched [blog.ckhill.com](https://blog.ckhill.com) given the following:
1. I didn't have anything hosted at ckhill.com currently.
2. My dad was asking me questions about HTML and site development and I hadn't done anything in the space in a couple of years so I wanted to see what new approaches had emerged.
3. I wanted to experiment with Azure capability since Azure Cloud is rapidly growing as a viable AWS alternative.
4. I wanted to capture some thoughts as we move from China to Singapore.
5. I want to capture learning as I grow my Data Science skills.

### Tutorials Followed

As with most attempts to learn/do I relied heavily on Google and the postings of others.  From work conversations I was interested in static site deployment approaches with little/zero server infrastructure.  These interests ended up directing me to Andrew Connell's excellent tutorial series:

* [Moved this site to Hugo](https://www.andrewconnell.com/blog/moved-this-site-to-hugo/)
* [Automated Hugo Release with GitHub Actions](https://www.andrewconnell.com/blog/automated-hugo-releases-with-github-actions/)

Unfortunately I initially skipped Andrew's [Automating Hugo releases with Azure Pipelines](https://www.andrewconnell.com/blog/automated-hugo-releases-with-azure-pipelines/) article as I was more interested in GitHub Actions.  This resulted in me not understanding some parts of Hugo deployment setup and I was really stuck on why my "hugo deploy" step was failing with a *Error: no deployment targets found* message in GitHub Actions workflow.  Fortunately I eventually read:
* [Using GitHub Actions and Hugo Deploy to Deploy to AWS](https://capgemini.github.io/development/Using-GitHub-Actions-and-Hugo-Deploy-to-Deploy-to-AWS/)
which clued me into the fact that I needed to update the Hugo config.toml with a deployment target.  Once that was resolved everything worked as expected.

I also occasionally needed to reference the following as my Markdown is really rusty:

* [Markdown Cheatsheet](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet)

The GitHub repository behind this site is now at [ckevinhill/ckhill-blog](https://github.com/ckevinhill/ckhill-blog).

### Tooling

The only tools that were really required for site launch and development included:

* [Git Bash](https://gitforwindows.org/) - used for Git commits/pushes to GitHub.
* [VS Code](https://code.visualstudio.com/) - used for editing configuration and markup files.
* [Hugo](https://gohugo.io/) - used to transform configuration, theme and markup files into static website.
* [Azure Portal](https://azure.microsoft.com/en-us/features/azure-portal/) - used for Azure service configuration.

As a side note one cool aspect of Hugo is that I can leave it running on local environment with *hugo server -D* and get a real-time updated view of edits I make before I push them to Master branch at GitHub for deployment to [ckhill.com](https://blog.ckhill.com).


### Follow-up Questions / Tasks

I have both HTTP and HTTPS enabled for this site as I was unable to figure out how to best redirect HTTP requests to HTTPS.  You are supposed to be able to do this with the Azure Microsoft CDN Rules Engine but I was unable to find these options in Azure Portal.

I am assuming that I will be able to integrate Jupyer Notebook outputs (exported to Markdown or HTML or other) into future posts but I'm not sure how that will work yet.

There is still alot of tweaking/configuration that I need to do for Hugo:
* Implement Google Analytic tracking
* Better understand how taxonomies are managed (i.e. what is the difference between a tag, category and series?)
* What can be configured in the front matter of a post and specifically what does the slug parameter do?
* Modify design templates to eliminate some of the things that annoy me (extra spacing, favicon, etc.)
* Figure out how to get Search to work with a static website.


