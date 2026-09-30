---
title: "Moving blog.ckhill.com from Azure Blob Storage to Cloudflare Pages"
date: 2026-09-29T20:00:00-04:00
tags: ["azure", "cloudflare", "tutorial", "hugo", "devops"]
draft: false
---

> *This post was written by Muse (Meta's AI assistant), under my direction and review, documenting a migration it carried out on this very blog.*

Six years ago I wrote about [setting up this blog with Hugo, Azure Blob Storage and GitHub Actions](https://blog.ckhill.com/posts/site-setup-using-hugo-azure-and-github-actions/). That setup served me well: Hugo builds the static site, a GitHub Actions workflow deploys it with `hugo deploy` to an Azure Blob Storage `$web` container, and an Azure CDN endpoint in front handled HTTPS and the custom domain. It worked, right up until it didn't — and the bill was the reason.

### Why move

Two things happened. First, I moved the domain's DNS to Cloudflare and the `blog.ckhill.com` record didn't survive the move, so the blog went dark. Second, when I looked at restoring it, the Azure path had gotten materially worse. Microsoft discontinued Azure CDN classic — the exact service my old setup used — and the replacement is Azure Front Door, which carries a **fixed base fee of ~$35/month** before you serve a single byte, plus per-GB egress on top. That's a **$420/year floor** for a blog that gets a trickle of traffic. The old per-GB math (about $0.081/GB in Zone 1, plus ~$0.02/GB for the storage account — maybe $25–40 a year for a small blog) was already hard to justify for a hobby project; the Front Door base fee makes it a non-starter. A fixed monthly fee is simply the wrong shape for something with near-zero traffic.

Cloudflare Pages, by contrast, is **$0** on the free tier: unlimited bandwidth, 500 builds a month, free HTTPS, and my DNS was already on Cloudflare. The math wasn't hard.

### What changed

Old architecture:

1. GitHub Actions builds Hugo, `hugo deploy` pushes to Azure Blob `$web` container
2. Azure CDN endpoint (`ckhillblog.azureedge.net`) serves it with the custom domain
3. Purging the CDN on every deploy via `az cdn endpoint purge`

New architecture:

1. GitHub Actions builds Hugo exactly as before (pinned Hugo 0.68.3, no build drift)
2. The workflow deploys the prebuilt `./public` directory with `wrangler pages deploy` to a Cloudflare Pages project
3. `blog.ckhill.com` is attached as a Pages custom domain, which provisions DNS automatically

The Hugo build step didn't change at all. Only the deploy target moved.

### Setting up the push to Cloudflare Pages

Here's the recipe, start to finish:

**1. Create a Cloudflare API token.** In the Cloudflare dashboard: profile icon → My Profile → API Tokens → Create Token → Create Custom Token. Grant **Account → Cloudflare Pages → Edit** on your account. (Zone → DNS → Edit is handy too if you want API-driven DNS changes.)

**2. Create the Pages project.** Either in the dashboard (Workers & Pages → Create → Pages → Upload assets) or via API:

```
POST https://api.cloudflare.com/client/v4/accounts/<ACCOUNT_ID>/pages/projects
{"name": "ckhill-blog", "production_branch": "master"}
```

**3. Add the token to GitHub.** In the repo: Settings → Secrets and variables → Actions → New repository secret named `CLOUDFLARE_API_TOKEN` with the token value.

**4. Replace the deploy step in the workflow.** The build steps stay the same; swap the Azure deploy for wrangler:

```yaml
- name: Deploy to Cloudflare Pages
  run: npx -y wrangler@3 pages deploy ./public --project-name=ckhill-blog --commit-dirty=true
  env:
    CLOUDFLARE_API_TOKEN: ${{ secrets.CLOUDFLARE_API_TOKEN }}
    CLOUDFLARE_ACCOUNT_ID: <ACCOUNT_ID>
```

`--commit-dirty=true` matters if your repo has a dirty git tree after the build (mine does — `public/` has committed files that Hugo regenerates).

**5. Attach the custom domain:**

```
POST https://api.cloudflare.com/client/v4/accounts/<ACCOUNT_ID>/pages/projects/ckhill-blog/domains
{"name": "blog.ckhill.com"}
```

Note the field is **`name`**, not `domain`. I lost twenty minutes to this: sending `{"domain": ...}` returns the baffling error `8000015: "The domain you have entered contains an invalid TLD"`. With the right field it attaches instantly, provisions the certificate, and creates the DNS record.

Push to `master` and the workflow builds and deploys. Every future push does the same.

### Gotchas worth knowing

* **Fine-grained PATs and workflow files.** If your GitHub token is fine-grained, editing anything under `.github/workflows/` requires the **Workflows** permission (Read and write) — Contents write alone gives you a 403. This one is poorly documented and the error message doesn't tell you what's missing.
* **Stale edge cache on the homepage.** After the migration, `blog.ckhill.com/` served a stale "Hello World!" placeholder while `/about/` was fine — the edge had cached the placeholder from before the domain was attached. A cache purge (dashboard: Caching → Purge Cache → Custom Purge) fixed it. If you only see it in your browser, hard-refresh first.
* **Retire the Azure resources.** The storage account and resource group are now dead weight — delete them so the meter stops completely.

### Bottom line

Same Hugo site, same GitHub Actions muscle memory, zero hosting bill, and deploys that finish in under a minute. The decisive factor wasn't even the per-GB rate — it was Azure replacing classic CDN with Front Door and its $35/month base fee. For a low-traffic blog, a fixed monthly floor is the wrong pricing shape entirely; Cloudflare Pages' free tier fits it exactly. The whole migration — diagnosis, Pages project, DNS, workflow rewrite, verification — took an evening, most of it spent on the two API quirks above. Hopefully this post saves you those twenty minutes.
