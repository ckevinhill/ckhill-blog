---
title: "Azure CDN Invalidation via GitHub Actions"
date: 2020-07-10T10:39:32+08:00
tags: ["azure", "tutorial", "devops"]
---

This is a continuation of the [Site Setup]( {{< relref "posts/site-setup-using-hugo-azure-and-github-actions" >}}) article.  

I have noticed that when I push updates to the site some pages are not always updated as the CDN is not immediately purged.  To fix this I wanted to add CDN expiration/purge as part of the GitHub Action CICD process.

I found initial instruction via [Purging Azure CDN with GitHub Actions](https://medium.com/@shilyndon/purging-azure-cdn-with-github-actions-1c18e2adaf18).  This approach is straight forward using the Azure CLI to establish a Service Principal account with CDN related Roles - however I wanted to understand how to do this via the Azure Portal so I also referenced [How to: Use the portal to create an Azure AD application and service principal that can access resources](https://docs.microsoft.com/en-us/azure/active-directory/develop/howto-create-service-principal-portal).

It actually took me a while to find clear (and current) instructions on how to add a Role to the Service Principal.  The setup in Azure Portal does not feel intuitive to me and there does not seem to be a way to add Roles to the Service Account within the Active Directory or Account Overview areas.  You have to go into the Subscription/IAM section.

Because I used the portal I also needed to manually format the client credentials for GitHub into the specified JSON format:

>{                                                                                 
>    "clientId": "GUID",                             
>    "clientSecret": "GUID",                         
>    "subscriptionId": "GUID",                       
>    "tenantId": "GUID"  
>}