---
title: "Fast Search for SharePoint: crawler process blocked due to memory capacity limit"
date: "2012-08-22"
categories: 
  - "fast"
  - "sharepoint-2010"
---

This week I encountered the problem that the indexation of content didn’t seem to end. After 5 hours of crawling, my 1000 html pages of content still weren't indexed by the Fast crawler. After putting on my Sherlock Holmes outfit, I started to investigate this problem. It turned out to be a real adventure to find a solution for this problem.

## Investigating the problem

Firing up the ULS-viewer learned me that my development machine was too low on resources:

``` 
_08/22/2012 21:22:36.40         mssearch.exe (0x09D4)                           0x1448        SharePoint Server Search              Content Plugin                        dtv8        Medium          At memory capacity. Load is 82%, configured to block at **80%**. have been waiting 01:57 to queue this document  \[documentmanager.cpp:969\]  d:\\office\\source\\search\\native\\gather\\plugins\\contentpi\\documentmanager.cpp_ 
```

This meant that 6.4Gb of the available 8Gb of RAM was in use. The easy solution was to kill some processes, and after shutting down some applications, I saw my CPU usage increase, which meant that Fast was crawling my content again…. for a minute or two.  So I decided to kill some more, somewhat useless, processes: Fast resumed to crawl my content again. But the process of a blocking crawl process repeated itself and it started out to be a race to keep memory usage under the 80%. Every minute I checked out if I could kill some (useless) processes to keep that memory usage below 80%: this killed my productivity. I tried to kill some of the more heavier processes to finish the crawl and even disabled some services and service applications: but for some reason my memory consumption was still too high.

## Solving the problem

I decided that, if I couldn't manage my memory to keep it below the magic number, I should change that magic number into something else. Of course today was my lucky day: a smart search query on google for "fast search sharepoint memory capacity block" didn't return any relevant results. Using my brains and a swiss-army knife was now my only option to get a solution for this challenge. I wasn't sure if this was a Fast specific problem, or a SharePoint search problem, and after starting (and finishing) a crawl using the Fast Query Service Application (which, in fact, is a SharePoint Server Search Application), showed me that the problem didn't arise here. I concluded that this was a fast specific problem. I started to check out on configuration files, Fast cmdlets and their parameters, configuration options in the central admin, but as I didn't know what to look for exactly, this turned out to be a hopeless action. Finally, I even decided to manually check out the registry, and, of course, this action turned out to be useless too. This made me starting to believe that this "configurable" setting wasn't so configurable.

As a last hope, I fired up the sysinternals process monitor tool ([http://www.sysinternals.com/](http://www.sysinternals.com/)), which shows all filesystem/registry/network activity for all processes, and configured it to display all registry and filesystem activity for the mssearch process. After restarting this sharepoint search service, I did a search for "memory", and after a few hits, I just "knew" why I couldn't find any configuration setting related to this memory limit: It just wasn't there!

[![](images/3276.procmon1.png)](http://bloggingabout.net/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/bas/3276.procmon1.png)

Manually adding this key to the registry was, of course, not the solution. After each restart of the service, the setting got deleted from the registry. But after taking a closer look at those registry keys, I remembered that the FastConnector:Collection and FastConnector:ContentDistributor were Fast specific service application settings and tried to add this key to the Extended Properties of my Fast Service Application

[![](images/1581.registry1.png)](http://bloggingabout.net/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/bas/1581.registry1.png)

The key was added using the following powershell script:

```powershell

 _$ssa = Get-SPServiceApplication | where {$\_.DisplayName -eq "FastContentService" } $property = New-SPEnterpriseSearchExtendedConnectorProperty -SearchApplication $ssa -Name "MaxMemoryLoad" -Value "82"_ 
```

The MaxMemoryValue of "82" was taken on purpose, because I wanted to make sure that this new value was showing up in the logs when crawling content, to make sure it was really this setting. After restarting the Search service, process monitor told me that the key was loaded!

[![](images/5516.procmon2.png)](http://bloggingabout.net/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/bas/5516.procmon2.png)

After checking out the ULS log, I was sure that I found the correct parameter:

```
08/22/2012 22:27:22.69         mssearch.exe (0x0A78)                           0x1648        SharePoint Server Search              Content Plugin                        dtv8        Medium          At memory capacity. Load is 90%, configured to block at **_82%_**. have been waiting 02:27 to queue this document  \[documentmanager.cpp:969\]  d:\\office\\source\\search\\native\\gather\\plugins\\contentpi\\documentmanager.cpp
```

## Summary

Changing the maximum memory load isn't hard, it's just undocumented. I strongly advise to not to change this value in production environments, as I don't know if this is supported by Microsoft (it might be for a reason that there is no documentation on this subject). For development environments, this configuration setting is quite useful, as those enviroments, usually, take up a lot of resources.  I would suggest that shutting down unnecessary processes should always be the first option prior to adjust this configuration.

Update: limiting the memory consumption of your sql server should be a very interesting option too, to limit your memory usage.
