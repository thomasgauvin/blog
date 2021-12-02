---
layout: post
title: How to setup a subdomain for a Github Pages static site with Namecheap
permalink: setting-up-a-subdomain-github-pages-namecheap
date: 2021-12-01 22:01:00 -5
---
I wanted to host my blog on a separate domain and repository from my personal website to give it its own space. I had already configured my apex domain (thomasgauvin.com) to point to my thomasgauvin.github.io repository, following [the official Github guide](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site#configuring-a-subdomain).

So now, I needed to configure my subdomain (blog.thomasgauvin.com) to point to my separate blog repository. Over the years, Github has made it easier to set this up, but this has resulted in quite a few documentation sites being out of date ([this StackOverflow thread](https://stackoverflow.com/questions/9082499/custom-domain-for-github-project-pages) helped clear things up).

Nevertheless, setting up a subdomain is quite easy now (when you know what you're doing!).

## 1. Add the subdomain to your Namecheap Advanced DNS configuration

From your Namecheap dashboard, click manage on the domain you want to add your subdomain.

![](../uploads/2021-12-01-22-40-55.png)

From the your domain management settings, click Advanced DNS. 

![](../uploads/2021-12-01-22-38-41.png)

Add a new CNAME Record, with your subdomain as Host and your {user}.github.io domain as the Target. TTL can be left as automatic. (You'll notice that I already have other records. I had already configured these in the past to link thomasgauvin.com to thomasgauvin.github.io. They are not relevant to our configuration of the subdomain however.)

![](../uploads/2021-12-01-22-43-09.png)

## 2. Configure your Github pages to support the subdomain

From your repository settings, navigate to the Pages tab.

![](../uploads/2021-12-01-22-47-44.png)

Add your subdomain as the custom domain. 

![](../uploads/2021-12-01-22-49-31.png)

If this works, Github will show a message indicating so.

![](../uploads/2021-12-01-22-49-50.png)

And that's it! We've successfully configured our subdomain to point to our Github Pages. This process is quite convenient to setup a custom domain, and it also makes it super easy to setup and enforce HTTPS on our website as well.