---
title: "Content by query webpart not performing due to misconfiguration"
date: "2013-01-03"
categories: 
  - "sharepoint"
---

Today we were experiencing some major performance issues on some pages on a new website. The pages that were experiencing these performance issues, all made use of the content by query webpart (we didn't have any performance issues before on these pages). The content by query webpart that we were using on the site, used a [pagefield value](http://blogs.msdn.com/b/ecm/archive/2010/05/14/what-s-new-with-the-content-query-web-part.aspx), which was used to lookup some related pages from another list.

## The problem

A quick look at the developer dashboard learned me that something "bad" was happening over there:

At first, critical database errors:

_fa42 A large block of literal text was sent to sql.  This can result in blocking in sql and excessive memory use on the front end.  Verify that no binary parameters are being passed as literals, and consider breaking up batches into smaller components.  If this request is for a SharePoint list or list item, you may be able to resolve this by reducing the number of fields._

In addition to that, some very bad performing sql queries took place (which could be related to the critical database errors).

[![](images/7268.developerDashboard_5F00_CBQWP2.png)](http://bloggingabout.net/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/bas/7268.developerDashboard_5F00_CBQWP2.png)

the queries that are marked in the screenshot above, included a query that created a table variable, and inserted data with "some" insert into select statements. #define some 489.

[![](images/4810.query.png)](http://bloggingabout.net/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/bas/4810.query.png)

Basically, this meant that from 489 different sources, data was inserted into the table variable. This table was queried on its turn, to get the data out if it, that we needed. As described in the small introduction, we just needed to lookup data from one single list! Altogether, it turned into loading times of 10-20 seconds.Due to all the inserts, I suspected that not the specific pages list was queried, but whole the site collection. With 500 Mb of data this can be quite time consuming ;).

A quick glance at the webpart properties learned me that I was right: the whole site was queried, where a much smaller scope was needed (the scope of a single list). Selecting the pages list as the datasource resulted in a much better, acceptable performance: a steady loading time of about 1s.In addition to the performance gain: the database errors got away, too ;).

## Summary

Altough the content by query webpart is a very good and [fast mechanism](http://blog.mastykarz.nl/content-query-web-part-vs-custom-aggregation-web-part/) to gather data, it can be misconfigured in many ways, which causes performance issues. In my case, it even caused database locks, which prevented the whole site from performing at all. When configuring the content by query webpart, always make sure to use a datasource that has the smallest scope possible.
