---
title: "Sharepoint: Rewriting zone url to public url"
date: "2009-03-07"
categories: 
  - "alternate-access-mapping"
  - "sharepoint"
  - "zone"
tags: 
  - "alternate-access-mapping-2"
---

Today I ran into a problem: Via the SharePoint search I got some results returned, including a pathName to it's default zone. This zone was used for indexing the sites, but isn't accessible as a production URL due to authentication issues. One solution was to remove the default zone and replace it by the public url via a replace function, but that method of course isn't failsafe in the future (what if the public url changes??)

Gladly, SharePoint has a neat little function delivered in the SPUtitlity class:

SPUtility.AlternateServerUrlFromHttpRequestUrl(Uri url)

Life is great when having a Utility class ;)
