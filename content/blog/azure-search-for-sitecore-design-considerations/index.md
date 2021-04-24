---
title: "Azure Search for Sitecore design considerations"
date: "2019-04-15"
categories: 
  - "azure"
  - "search"
  - "sitecore"
tags: 
  - "sitecore"
img: "./images/image-23.png"
---

Azure Search makes up for a significant amount of costs within a Sitecore on Azure setup; apart from these costs, there are often issues with Azure Search and clients tend to move away from Azure Search and spin up a new VM with Solr - But is this really mandatory?

this blogpost is part of a [series of blogposts](https://blog.baslijten.com/sitecore-on-azure-design-considerations-to-be-more-cost-efficient-and-have-more-performance/)

## Sitecore developers are spoiled

Most Sitecore developers are very spoiled; on premise hardware is virtually unlimited. For example: when working with Solr, there is no need to take care of any limitations - but things change when the workload moves to Azure PaaS, as Azure Search does have some limitiations. But how severe are these limitations?

## Azure Search limits

When looking at the [Azure Search limits](https://docs.microsoft.com/en-US/azure/search/search-limits-quotas-capacity) there are a few important ones to take care of:

<table class="wp-block-table"><tbody><tr><td>&nbsp;</td><td>Free</td><td>Basic</td><td>S1</td><td>S2</td><td>S3</td><td>S3 HD</td><td>L1 + L2</td></tr><tr><td>Indexes</td><td>3</td><td>15</td><td>50</td><td>200</td><td>200</td><td>3000</td><td>10</td></tr><tr><td>Fields</td><td>1000</td><td>100</td><td>1000</td><td>1000</td><td>1000</td><td>1000</td><td>1000</td></tr><tr><td>Scale units</td><td>0</td><td>3</td><td>36</td><td>36</td><td>36</td><td>36</td><td>36</td></tr><tr><td>Costs</td><td>0</td><td>62.18</td><td>206.84</td><td>827.38</td><td>1654.76</td><td>1181.35</td><td>2363.32</td></tr></tbody></table>

### Indexes

Azure Search has an maximum index limit - while Sitecore comes with 13 indexes out of the box (including the xdb and xdb\_rebuild index), this leaves very few possiblities open for custom indexes. This also means that the Free tier and L1 + L2 tiers cannot be used by Sitecore, as they don't support enough indexes. Another consequence is that the basic and S1 tiers cannot host multiple Sitecore environments, if you would share an Azure Search service over multiple instances.

### Fields

Every tier, except the basic tier, supports 1000 fields. This 1000-field limit is strict and could lead to severe problems when not being monitored correctly. Due to the 100 field limit on the basic tier, this basic tier can not be used without any modifications to Sitecore (more on that later)

### Scale units

Scale units are the amount of replicas multiplied by the amount of partitions. The default settings are "1", but when more concurrent searches or faster queries are needed, a replica or partition could be added. To explain how this works, the "telephone book analogy" could be used:

#### telephone book analogy

Imagine a single telephone book with 1000 telephone numbers. Just one person could use that telephone book to lookup a certain telephone number. To be able to have multiple persons search through that index, a _replica_ of that telephone book should be created. For every replica, an extra user can execute a concurrent search through the index.  
  
While the index with telephone numbers grows, it might take longer and longer to do a proper lookup for a telephone number. An increase to 10000 telephone numbers in the _index_ might lead to a performance degradation. In this case, one more more _partitions_ could be added: the index would be split into a partitioned phonebook with last names starting with A-M and  
a partion with lastnames starting with N-Z. To be able to lookup a telephone number with multiple concurrent users, each partition would need its own replica. In the end you would and up with an 'X' amount of partitions, each with 'Y' replicas, which would lead to a total of X\*Y books: the amount of Scale units in terms of search.

## Getting the most out of Azure Search for Sitecore

Getting the most out of Azure Search isn't too hard, the only thing is that you might to unlearn some old habits. In the end it might lead to a bit more configuration, while it might save tons of money

- Work around the 1000 field-limit
    - Index only what you need
    - Create custom indexes
- Cache, Cache, Cache
- Scale your Search service (or don't)
- Consider an external Search solution

### Work around the 1000 field-limit (aka the friends with benefits method)

Most Sitecore developers have a background with Solr, which doesn't have too many limitations. It could virtually host an unlimited amount of fields, which is not the case with Azure Search. All tiers, except the basic tier, can hold a maximum of 1000 fields, due to the way Azure Search copes with it's memory. This limit can be reached really fast, when working with small, autonomous templates, SXA, and/or multiple languages. This is due to the fact that Sitecore has a default setting on its master and web indexes named "indexAllFields='true'".

#### Set 'indexAllFields to 'false'

This will prevent all fields from being indexed. Sitecore does come with a configuration in which all required fields for the content editor view will work (title, body, buckets and some other computed fields) - thus this will greatly reduce the amount of fields.  
  
The drawback, however, is that your complete search functionality will break, as no custom field is in the index anymore.

#### Create a custom index for 'web' and 'master' and put your custom fields in these indexes

Using this pattern, will keep the out of the box index very clean _and_ fast. Sitecore could add whatever field they want to this clean index, without affecting _your_ indexes. Add all fields that are used within your custom search functionality into the custom indexes. This will lead initially to a bit of custom configuration, but it will lead to very small and manageable indexes as well.

#### Extra free benefits - faster indexing actions and faster queries

**Faster indexing actions**  
The extra benefits in this approach are great: because just a very small subset of all data is included, the (re)-index of the Sitecore content decreases from several minutes to several seconds. This helps in the blue green deployments and uptime of your environment.

**the search queries are faster** _**and**_ **smaller**  
When querying Azure search, all fields in the index are returned - even if they are empty. When having 900+ fields, this will lead to very large responses (900 fields per result) and long response times (as Azure Search searches through all fields).  
  
After the optimization of having a custom index with only the fields that are really needed, this will mean that for each result only a few fields are returned and because of the small dataset, the query is much faster than before.

**Move to Plan B (the pricing tier) - More and faster queries for lower costs**

Due to this approach, its very likely that your sitecore indexes would stay below the 100 fields. This means that the the "Azure Search S1' level could be reduced to the "Azure Search tier B" level. This could lead to a costs reduction of 206-64 = 142 EUR per environment, just by using a different configuration. Especially for your dev, test and QA environments.  
  
_Note: I have not researched the impact of xConnect for this approach  
Note2: Tier 'B has a maximum of 3 Scale units. If you need more query power, you should still consider the S1 tier. But this is often not applicable for dev and test environments._

### Cache, Cache, Cache

Although search is a very fast pattern to lookup data, it still takes time. When your result set is always the same (for example, when looking up a certain set of articles of a specific category), this dataset should be cached and not be executed on every request. With an increasing load on your webserver, this would lead to an increasing load on your search service, which could mean that an increase in replicas would be needed. When having just one partition, this could be,- EUR 64 (tier B), EUR 206 (S1) or EUR 827,- (S2), but when working with multiple partitions, this could lead to a significant increase of costs.  
  
An easy approach is to just turn on your output cache, as you would normally do on your Sitecore renderings, other approaches might involve custom Sitecore caches. The key in here is: be as cheap as possible

### Scale your Search

After these changes, it might be possible that you could scale down your search services. Make sure to execute some performance tests on your QA environment before you scale down, or you might end up in error with your production environment. Scale down _could_ mean: scale down to a lower tier _or_ reduce the number of replicas and partitions.

### Consider an external Search solution

Yes. Although developer tend to solve everything with custom code and own frameworks, a site that heavily depends on search might benefit from external search solution such as Coveo. They offer a _lot_ of functionalities to the content editors out of the box for a price that no developer could build it themselves. 2000,- a month sounds a lot, but this is barely 20 hours of development for a single developer: I bet that in those 20 hours no deployments, bug fixes, new functionality and a fully scalable solution could be build.  
  
moving to an external search solution would mean that that vendor takes care of the performance for the frontend searches and you would only have to take care about the performance on content management and xConnect environments.

## Indications that you might need to redesign your search approach

The best indicator is the Azure Search Metrics. In this overview, you can display graphs on query latency, query throttling and the amount of queries. When there is any sign of query throttling, this means that these queries are not executed and thus, leads to problems in your portal.

![](https://i0.wp.com/blog.baslijten.com/wp-content/uploads/2019/04/image-21.png?fit=625%2C273&ssl=1)

Query throttling kicks in  

The issue in the picture above happened due to a misconfiguration, all other days show correct behaviour. The graph below gives more food for thought. The red lines are search queries, while the purple lines are throttled queries. An interesting fact: during the throttling almost no queries were possible: something really messed that search service up during that time!

![](https://i2.wp.com/blog.baslijten.com/wp-content/uploads/2019/04/image-22.png?fit=625%2C92&ssl=1)

## Gotchas

#### Moving up or down a service tier is not possible

When deciding to go with a "Tier B" search, keep in mind that upgrading to a "Tier SX" is not possible. You would require to delete your Search service, and recreate it, to get a service with the same name. Getting the same apikeys is _not_ possible.

#### SXA

SXA adds all the template fields to the index by default. By turning this off, you would have to manually add all the fields. While SXA would only need actions within the Sitecore environment and no configuration changes, this strategy might not be a suitable solution for you.

## Summary

Azure search costs can "rise out of the pan" (this is a dutch saying) increase very fast and lead to errors pretty quick, when not correctly scaled. By being "cheap on resources", these issues could easily be overcome, which would still allow you to run your Sitecore on Azure completely as a PaaS setup, instead of having to add an IaaS based Solr instance.
