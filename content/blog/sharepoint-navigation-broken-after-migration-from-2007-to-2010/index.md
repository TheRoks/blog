---
title: "SharePoint Navigation broken after migration from 2007 to 2010"
date: "2011-07-06"
categories: 
  - "sharepoint"
  - "sharepoint-2010"
---

When migrating from SharePoint 2007 to SharePoint 2010, it might be possible that your navigation is broken. The error I encountered was the following, but it can occur in more functions within the Navigation namespace.

Object Reference not set to an instance of an object. Stacktrace: at Microsoft.SharePoint.Publishing.Navigation.PortalSiteWebSiteMapNode.FetchDynamicItems(PublishingWeb pubWeb, NodeTypes includedTypes, Boolean& websFetched, Boolean& pagesFetched) at Microsoft.SharePoint.Publishing.Navigation.PortalWebSiteMapNode.PopulateNavigationChildrenInner(NodeTypes includedTypes)

This error can arise when you restored a SharePoint site to SharePoint 2010, and that site was created in a language which had it's default "Pages Library" location changed.

for example

English pages library was located at "Pages" and not changed The German pages library was called "Seiten" (if i recall it correctly), and is not changed The dutch pages library was called "Pages" (and thus, not localized), but has the default location of "Paginas" in SharePoint 2010 (so it's location was changed) When a SP2007 site is migrated to SP2010, the navigation provider expects that any changes to the default location, is stored in a specific property, which, apparently, isn't the case, after a migration.

Luckily, Microsoft provided a powershell for this to fix it. And no, it has not been fixed in SP1. Check the following link for the exact cause and resolution: http://support.microsoft.com/kb/2484317
