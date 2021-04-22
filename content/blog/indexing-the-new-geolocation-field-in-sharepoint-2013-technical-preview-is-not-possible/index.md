---
title: "Indexing the new Geolocation field in SharePoint 2013 Technical preview is not possible"
date: "2012-07-25"
categories: 
  - "geolocation"
  - "sharepoint"
  - "sharepoint-2013"
---

As I was playing around with the new geolocation column of SharePoint 2013, I wanted to try to migrate my geosearch solution for Fast Search for SharePoint 2010 to SharePoint 2013.  The addition of the geolocation column was opening up some interesting possibilities. Potentially, this solution would add some (near) out of the box geolocation solutions.

## Unkown Datatype

After adding the geolocation column, adding some data to the last, I started to crawl the content. For some reason, the geolocation crawled property didn't show up. And the reason why was logged to the Diagnostic log:

_07/22/2012 21:14:27.12 mssdmn.exe (0x22BC) 0x1088 SharePoint Server Search Connectors:SharePoint dvs3 High Unknown datatype Geolocation \[sts3util.cxx:6498\] search\\native\\gather\\protocols\\sts3\\sts3util.cxx_

Apparently, the logic to index a geolocation column, hasn't been implemented yet. Let's hope it will be implemented when SharePoint 2013 will get released!

Expect some blogposts with a workaround for this issue soonish :D
