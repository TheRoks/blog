---
title: "SharePoint 2013 and Windows 8 apps - better together Part 1: Introduction, background and considerations"
date: "2013-03-16"
categories: 
  - "rest"
  - "search"
  - "sharepoint"
  - "windows-8"
---

This is the first post as part of a blog series about the integration of using SharePoint 2013 as a datasource for windows 8 apps.

- [Part 1: Introduction, Background and Considerations (this post)](SharePoint 2013 and Windows 8 apps – better together Part 1: Introduction, background and considerations "Part 1: Introduction, Background and Considerations")
- [Part 2: Platform choice, using the right API and data access](http://blog.baslijten.com/sharepoint-2013-and-windows-8-apps-better-together-part-2-platform-choice-using-the-right-api-and-data-access/ "Part 2: Platform Choice, using the right API and data access")
- Part 3: Integrating Search using REST
- Part 4: Authentication

## Introduction

On 8 march 2013, my colleague Ad Reijngoudt (Windows 8 App developer, follow him on twitter: @Areijngoudt) and I spoke on the dutch Techdays on the subject: "SharePoint 2013 and windows 8 app - better together". After this session, we got a lot of questions on several subjects, so I decided to write some blogposts on these subjects.

## Background

My colleague Ad developed a fully functional windows 8 application within 3 days during a workshop with Microsoft in The Netherlands. This application is called "Vergoedingenzoeker" in Dutch (Is it called Compensation finder in correct english?!) and offers functionality to find possible compensations within your health insurance. The downside of this app, was the fact that it contained a static datasource; whenever there is a change within the consumed data, the app needs to be updated and that is, of course, not the most ideal situation. That's why we decided to find out if we could create a nice backoffice for this application.

## The "zorgvergoedingen" app

The vergoedingenzoeker app basically exposes the following functionality: On startup, a selection screen is displayed. Users can navigate to information in two ways

1. Select a compensation level. Based on the selected level (1-4 stars), an overview of all compensations within that level is displayed
2. Select a character, and all compensations of which the title start with that same character are displayed

[![](images/1616.app_5F00_overview_5F00_2.png)](http://bloggingabout.net/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/bas/1616.app_5F00_overview_5F00_2.png)

Whenever a choice has been made, all item titles that fall within the selection criteria, are displayed:

[![](images/2084.vergoedingen_5F00_overview.png)](http://bloggingabout.net/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/bas/2084.vergoedingen_5F00_overview.png)

The last action is to select a subject. After a subject is selected, all details are displayed:

[![](images/4743.vergoedingen_5F00_detail.png)](http://bloggingabout.net/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/bas/4743.vergoedingen_5F00_detail.png)

All this data is stored within SharePoint and within the upcoming blogposts, you'll see how we did this.

## Considerations for using SharePoint 2013 as the backoffice

As we discussed what kind of backoffice we should use, I suggested SharePoint 2013 immediately:

- I love SharePoint! (I can't say if that was the main reason or not? ;)
- We already had existing data in our public site, so why not reuse that data? It prevents maintaining data on multiple platforms, which is always a good idea. (altough this was a MOSS 2007 site, we could make an extract of the data and host import that data into SharePoint 2013)
- The new API's of SharePoint 2013 are very interesting (in my opionion), so this was a good situation to find out about possible limitations
- SharePoint 2013 offers great out of the box functionality, which should easily be integrated with the windows 8 app. Think about Userprofiles, Search, mananged metadata. And all of these service apps are accessible via the new API's

Because of the last three arguments, Ad agreed on using SharePoint 2013 (being an ASP.Net / windows 8 fanatic, he doesn't admit that he loves SharePoint as well ;))

## Upcoming blogposts

The upcoming blogposts will cover the following subjects

- Using the right API and data access - How did we expose the data to the windows 8 app and what choices did we have to make? What tooling did we use?
- Search - How did we implement search functionality in the app. We made some really cool choices :D
- Authentication - How to authenticatie to Office 365. There are already some blogposts on the subject, but we improved it a bit and decided to share this with the community.
