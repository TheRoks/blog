---
title: "Setting a default value for a sitecolumn in a pagelayout"
date: "2009-03-13"
categories: 
  - "contenttype"
  - "sharepoint"
  - "sitecolumn"
---

Today we came across some weird bug within SharePoint. We needed to provide a default value for configuration options in a pagelayout. We verified everything on a development server and everything worked fine. When we deployed this code to the test environment, we found out that the default values were not being set.

What was the problem? To be able to set default values to a sitecolumn, you need to set the contenttype as default in the pages library, or the contenttype needs to be deployed on siteCollection level.
