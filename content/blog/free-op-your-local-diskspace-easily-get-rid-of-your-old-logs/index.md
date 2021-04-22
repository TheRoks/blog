---
title: "Free op your local diskspace - easily get rid of your (old) logs"
date: "2019-08-04"
categories: 
  - "code"
  - "iis"
  - "powershell"
  - "sitecore"
---

I bet that a lot of people have this issue: Having a lot of (old) Sitecore installations that you don't want to remove, as you aren't sure whether or not there is still some valuable configuration in it. With a default installation, these installations grow over time, as they are running by default and are (thus) generating logs. I never change the logging settings to just generate logs for one day, which means they will eat up a _lot_ of diskspace, especially the xConnect ones, as they might generate logs up to 1Gb per logfile in size!

The following powershell line can be used to delete all the logs which are older than 2 days:

<script src="https://gist.github.com/BasLijten/8eb0d09ad36666725e538b03d69250a6.js"></script>
