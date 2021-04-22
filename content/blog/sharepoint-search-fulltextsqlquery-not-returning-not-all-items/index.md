---
title: "SharePoint Search FullTextSqlQuery not returning not all items"
date: "2009-03-31"
categories: 
  - "fulltextsqlquery"
  - "scope"
  - "search"
  - "sharepoint"
---

Today I was breaking my head on some stupid problem within SharePoint Search. We created a site with about 50 dummy pages to test some cross webapp news aggregation webpart that I created. For some reason just 4(!) news items were found, each one existing in another subweb. In the SSP the View Scopes page also showed the magic number of '4' items found.

THe problem? I forgot to set the following property to false, which is set to true by default:

query.TrimDuplicates = false;

It makes sure that items that look like each other, won't be returned as a result. Due to the fact that i had 50 of the same dummy pages, just one item per subweb was returned.
