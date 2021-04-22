---
title: "CrossListQueryInfo not able to return MultiLookup Fields"
date: "2009-08-14"
categories: 
  - "caml"
  - "crosslistqueryinfo"
  - "query"
  - "sharepoint"
---

Some time ago [I wrote a blogpost about how to use the CrossListQueryInfo and CrossListQueryCache](http://bloggingabout.net/blogs/bas/archive/2009/03/27/using-the-crosslistqueryinfo-and-crosslistquerycache.aspx) to be able to find any item within a site within a very small amount of time. Hower, since today I know that this method cant be used when you are trying to retrieve items that contain a MultiLookupField in the ViewFields.

I came accross this limitation when I accidentally changed a LookupField into a MultiLookupField. Suddenly, my webpart that made use of the CLQI didn't return any items anymore, although I was sure that I didn't modify any code. For some reason I immediately had to think about the Database diagram that i created some time ago and I opened the database to check something.

Although all ListItems with its data are stored into the AllUserData table, there is some data that is stored somewhere else: the AllUserDataJunctions. This table stores the multilookup data. This might be the reason that the CLQI isn't able to display the multilookup results.

Anybody has any thoughts on this?
