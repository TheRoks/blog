---
title: "Automate your pipeline part 1: World’s fastest delivery pipeline for Sitecore on Azure"
date: "2018-10-16"
categories: 
  - "sitecore"
img: "./images/usain.jpg"
---

On October 9th I presented for the 3rd time at the Sitecore Symposium. In my [previous blogpost](https://blog.baslijten.com/my-sitecore-symposium-session-worlds-fasted-delivery-pipeline-for-sitecore-on-azure/) I described shared how I felt during the creation of this presentation and on the day itself. In this series of blogposts I’ll describe every subject I discussed during my presentation, which will, in the end, enable you to setup your own fully automatic deployment pipeline using standard Microsoft technologie such as msbuild, msdeploy, nuget and Azure DevOps. This blogpost is just a container for all the upcoming blogposts (which is subject to change). When you are missing a subject, feel free to get in touch with me.

This blogpost is part 1 of the series “Automate your pipeline”. The blogposts may or may not be released in the order as posted in the list below,

- part 1: Automate your pipeline part 1: World’s fastest delivery pipeline for Sitecore on Azure (this blogpost)
- part 2: Provisioning vs deployment
- part 3: different types of modules
- part 4: automate msdeploy all the things!
- part 5: create your own packages using msdeploy – the Sitecore baseline
- part 6: Patch the unpatchable – how to handle changes to the web.config
- part 7: how to build your business application
- part 8: how to deploy your business application
- [part 9: speed up your deployments - parallel deployments](http://blog.baslijten.com/speed-up-your-deployments-parallel-app-service-deployments-in-azure-devops/)
- part 10: speed up your deployments - unicorn
- part 11: speed up your deployments - does size matter?
- part 12: deploy with the speed of light
- part 13: release toggles (release toggles)

## Purpose of this series

The main purpose of this blogpost is to share all knowledge that we have build up during our journey to a fully, fast, automated CI/CD pipeline. We prefer just to make use of default Microsoft technology, which means we try use msbuild, msdeploy, Azure DevOps and powershell wherever possible and applicable, without making it overly complex.

The original presentation can be found [here](https://www.slideshare.net/baslijten/worlds-fastest-delivery-pipeline-for-sitecore-on-azure)

\[slideshare id=119510920&doc=sitecoreonazurecicdv2-181015193918\]
