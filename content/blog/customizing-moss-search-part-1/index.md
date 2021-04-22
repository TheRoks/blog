---
title: "Customizing MOSS Search - Part 1"
date: "2009-06-18"
categories: 
  - "peoplesearch"
  - "search"
  - "sharepoint"
---

MOSS 2007 and WSS 3.0 are using the same search engine. It only differs in the options that are available to each of the products. As expected, MOSS 2007 has a lot more functionality then WSS 3.0. Where WSS 3.0 is only able to use contextual search (as in: this site, this list), MOSS 2007 can be configurated to search in the whole farm (content sources), to search for certain type of results (defined in scopes), scheduling the crawl and property management (search on metadata).

**Content Sources** are the sources where we can get the information from. As per default, just 1 content source is defined: _Local Office SharePoint Server sites._ Within this Content source, all newly sitecollections are added to be crawled. It is also possible to add other sources like exchange folders, file shares and other websites. Those items are crawled using the default access account. If that account has no access to a source, those items are not crawled, and vice versa. When this account has a to high permission level on those sources, it can be possible that information that is not intended to be searched, is shown into the search results. This is not the case for sharepoint sites that are searched, because the results for these sites are security trimmed.

**Search Scopes** defines the kind of data you are searching for within a certain area. Two default scopes are _"All Sites"_ and "_People"._  The first _scope_ defines that all information within All available sites is searched. The "_People"_ scope is used for the people search. The data that is found within this scope are just user profiles. A third scope can be added, for example "News" In this scope you can specify that all content sources have to be used, but you require that only data with contentType NewsItem is included. This last requirement is called a "Property Query" [![](images/7532.searchScope.JPG)](http://bloggingabout.net/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/bas/7532.searchScope.JPG)

**Managed Properties** are properties that can be used in scopes. Those properties are provisioned by crawled properties that are mapped to this managed properties. For example, after a full crawl, there some new crawled properties are found: _ows\_AssignedTo(Text)_ and _ows\_Assigned\_x0020\_To(Text)._ Those two crawled properties can be mapped to the managed property _AssignedTo._ From now, in a Search Scope with the property Query "AssignedTo = Bas Lijten" can be used to create a scope where all items that are assigned to "Bas Lijten" can be searched. All items that apply to this rule, have a metadata property (crawled property) ows\_AssignedTo or ows\_Assigned\_x0020\_To with the value Bas Lijten

**Configuring People Search** Configuring the **People Search** isn't that hard. At first, you will need of course, user profiles and their user profile properties. You will have to configure the user import for this. After this, decide what user profile properties need to be indexed. With this index,  the user profile property will be part of the People Search Scope Schema. If a property is indexed, you will be able to search for that element, and you will be able to add that propety as a search result. When those configuration settings are made, the userProfile store needs to be crawled. If i am right, this is not enabled by default. This can be enabled by adding the following entry to one of the content sources: sps3://mysite-host/

When this entry is added and a full crawl is started, all userProfiles will be added, together with its properties that can be used in a **scope** now.

In the next parts I will explain how to configure the search centre, how to configure/style the main search box and how to style the peoplesearch page.
