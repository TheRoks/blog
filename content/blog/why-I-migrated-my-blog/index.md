---
title: Why I migrated my blog from wordpress to Gatsby and how it feels
date: 2021-04-26
description: explanation on the reasons why I migrated from wordpress from wordpress to gatsby
img: ./images/2021-04-26_14-51-47.png
tags: []
---

Finally, my new blog is live. For months, I had the idea to migrate my blog to a new CMS, but every single day that I decided to start, there was a reason not to start. Speaking on conferences, work, days off, speding time with family, you name it. But, the last week, there was no reason not to do it. I was on holiday leave with my family and every day, I woke up early, having a few choices: gaming, running or developing. I chose for a mix of these activities and in just two short mornings, I was able to export my old blogposts, setup gatsby _and_ fix the theme that I am using. My final thought: why didn't I do this much sooner?

## The reasons behind the migration

My previous blog was running on wordpress on an azure app service for ages, using a MySQL database hosted externally. Every now and then, I faced problems with this blog: 
* **problems with the external database** - When I decided to host wordpress on an azure app service, there wasn't a service for MySQL yet, so I was forced to make use of an external provider. Although it costed me a few bucks per month, it was still a few bucks per month
* **wordpress runtime issues** - Every now and then, Wordpress needs an update. As I still don't have to much faith in automatic upgrade processes, I needed to do this manually. And although upgrades are very well regulated at my employer, I didn't pay enough attention to these upgrades myself
* **Problems with the PHP runtime** - I was running an older version of PHP. Whenever I tried to upgrade the PHP version, my blog broke. As I am not a PHP expert, and I definitely do not want to become one, I needed to migrate to another technology stack
* **problems with my MSDN subscription** - I am running my app service on Azure, on an MSDN subscription which has been provided by my employer. Recently, they migrated the subscriptions and azure benefits, which forced me to move my resources over as well. As I didn't have a "redeploy button", this costed my quite some effort. Since then, I realized that I was to much dependent on my employer in order to host my blog.
* **No automatic (re)-deployment** All my blog-upgrades where manual actions, the source code was somewhere in a zip-archive and an automatic ~~build and~~ deployment wasn't in place at all. As I always tell my colleagues to have all the CI/CD pipelines in place, I realized that I should practice what I preached. As I didn't want to invest time in a technology stack that I didn't like, I had to move over
* **I wanted to learn more about React and static sites** - I don't have to become an expert, but I want to learn the basics and I want to be able to talk on acceptable level with our javascript experts. One of my colleagues ([Stefan "Sitecore Rocky" Roks](https://theroks.com)) recently migrated to gatsby as well and he was really enthousisatic about it. That made me move over as well   

## My first thoughts

well, why didn't I move over any time sooner? Using gatsby is a *blast*. It's fast, it's easy to learn, easy to deploy and easy to modify. For _my_ usecases, it does what it needs to do and even more. Within half an hour, I was able to export my existing Wordpress blogposts _and_ incorporate them a fresh gatsby environment. However, there were a few caveats and issues with the theme that I used, but after I understood the workflow, I was able to fix them in a few minutes. When looking back, there was NO reason to postpone this activity, as it was easy and smooth. I am happy and I hope that I'll be able to write a lot new blogposts in the future!
