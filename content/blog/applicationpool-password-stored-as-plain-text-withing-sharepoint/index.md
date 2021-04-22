---
title: "ApplicationPool password stored as plain text withing SharePoint"
date: "2009-03-28"
categories: 
  - "applicationpool"
  - "security"
  - "sharepoint"
---

A few days ago I was reading a blog (And I forgot what blog!!) with information that the ApplicationPool password was stored as plain text. If you don't believe me: check the screenshot below:  

[![](images/5621.plaintext.JPG)](http://bloggingabout.net/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/bas/5621.plaintext.JPG)

The password is as well accessible via the objectmodel, when runnin under elevatedPrivilges..

Lessons learned: Always try to have a least-privilegd installation for your SharePoint farm!
