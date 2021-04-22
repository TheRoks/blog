---
title: "Solr: Error creating SolrCore when using the Sitecore installation Framework"
date: "2017-10-25"
categories: 
  - "deployment"
  - "feature-2"
  - "search"
  - "sitecore"
  - "solr"
coverImage: "solrerror-1.png"
---

Today I experienced an error while installing Sitecore 9 using the Sitecore installation framework:

_“Install-SitecoreConfiguration : Error CREATEing SolrCore 'xp0\_xdb': Unable to create core \[xp0\_xdb\] Caused by: null”_

Setting the verbose logging option didn’t help and my tries to manually reproduce the issue didn’t work out as well; _Or_ the core was successfully created _or_ I got an error message that the core was already created.

It turned out that there was something wrong with the timing. In the sitecore-solr.json and xconnect-solr.json file a few tasks get executed to create/reset the cores:

- StopSolr - this stops the windows service
- Prepare cores - copies the basic config set to the directory which hosts the index
- StartSolr - starts the windows service
- CreateCores - tells Solr using a http request to actually create the core

In my case, the windows service was still starting, while the http request was executed, which caused the Sitecore installation framework to bug out. The strange part: With Solr 6.2.1 this did not happen, while it happened with Solr 6.6.1.

The solution is quite easy (and I expect that Sitecore had the same experience while developing SIF): the StartSolr task has a parameter named "PostDelay", which initially has been set to 8000. I increased this to 20000 (just a lucky number) and all the errors were gone by the wind :D. See line 16 in the snippet below where I updated the value

<script src="https://gist.github.com/BasLijten/6d58d7623c8838e31be7a1e73da2abe1.js"></script>
