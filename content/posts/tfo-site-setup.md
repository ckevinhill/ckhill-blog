---
title: "The ToothFairyOffice.com Site Setup"
date: 2020-08-28T17:02:02+08:00
draft: false
tags: ["devops", "tutorial", "azure", "other"]
---

As a parent, what do you do when the tooth fairy accidentally forgets to do their job?  

*Clearly* the only possible option is to:
1. Create an elaborate back-story 
2. Quickly launch a website supporting said elaborate back-story.  

Thus was born into the world: [http://www.toothfairyoffice.com/](http://www.toothfairyoffice.com)

## Domain Registration

The most important part of any web-site is the domain.  In this case I ended up using [Godaddy.com](http://www.godaddy.com).  They were slightly more expensive than some other registrars but I already have the ckhill.com domain registered there so for convenience of having all my domains in one place it was worth the few extra dollars.

> When searching for domains some registrars would list the domain as "premium" and add a huge up-charge (up to 300x!).  It seemed that this varied from registrar to registrar so if you hit a "premium" domain tax on one site it is probably worth trying somewhere else.

Total cost for 1 year of hosting was around 10$.

## Site Template

The hardest part of launching a website is design and the horror and complexity of CSS.  I knew this site was going to be at most 1 or 2 pages and did not require some sort of content management approach (i.e. does not require [hugo builds](/posts/site-setup-using-hugo-azure-and-github-actions) like the blog).  As a starting point I used [site templates from W3CSS site](https://www.w3schools.com/w3css/w3css_templates.asp) - in particular I decided the ["band" template](https://www.w3schools.com/w3css/tryit.asp?filename=tryw3css_templates_band&stacked=h) would provide the basic layout and sections I needed and were already mobile optimized.

Total cost: 0$.

## Artwork

It is amazing how accessible and in-expensive site artwork has become (assuming you aren't just stealing it from Google Images).  I knew I wanted a large header image of a Tooth Fairy but I wanted to make sure to have ethnic diversity which made finding the images more challenging.  I typed in various search phrases in to Google Images (e.g. "tooth fairy", "african american tooth fairy", etc.) until I found a set of artwork that seemed to meet the need.  It led me to [Mujka Cliparts](https://mujka-cliparts.com/) where I was able to buy high-resolution Tooth Fairy clip art with transparent backgrounds.

Modifying (resizing adding backgrounds, etc.) was done  via the opensource [GIMP editor](https://www.gimp.org/) - [installed via Choco](/posts/choco-for-windows) of course!

Total cost: 2$.

## Local Development

Now I needed to modify the static html from the W3CSS band template into something more Tooth Fairy friendly.  I used VSCode and a docker nginx image to make quick local revisions until the site looked like what I wanted

Pull down the nginx docker image:

```
docker pull nginx
```

Run the container with my development folder mounted to the /usr/share/nginx directory (i.e. where html is served from) and map localhost port 8022 to port 80 on the container so that I can access my emerging toothfairyoffice site via http://localhost:8022.
```
docker run --name tfo_web_server -p 8022:80 -v C:\Git-Projects\ckhill_dot_com\tfo\site:/usr/share/nginx/html:ro -d nginx   
```

## Deployment

I used a similar approach as blog.ckhill.com for deployment.  HTML is checked into GitHub and then a GitHub Action is responsible for pushing it into an Azure blob setup as a static website.

An example of this approach can be found in the [Blog Site Setup post](/posts/site-setup-using-hugo-azure-and-github-actions).  

The biggest difference is that instead of using the [pre-built hugo deploy Actions](https://github.com/peaceiris/actions-hugo) I used a more manual implementation of the AZ CLI to batch upload the HTML files to the blob storage:

```
    # Copy /site folder to Azure blob

      - name: Copy files
        run:
          az storage blob upload-batch -d '$web' --account-name ${{ env.AZURE_STORAGE_ACCOUNT }} --account-key ${{ secrets.AZURE_STORAGE_KEY }} -s "./site" --pattern *.*
```

> I don't expect this site to be updated much so I didn't both with the [CDN invalidation](/posts/azure-cdn-invalidation).

The TFO site GitHub Repository (workflow YAML and html) can be found [here](https://github.com/ckevinhill/tfo-site).

Cost: >1$