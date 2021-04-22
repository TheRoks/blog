---
title: "SharePoint 2013 geolocation column: a component is not installed"
date: "2012-07-22"
categories: 
  - "geolocation"
  - "sharepoint-2013"
---

Today I was playing with the new geo location column. I added the Bing maps key, created a custom list and added the geo location column via code to that list. Upon submitting a new listitem, I received an error, mentioning that a component was not installed, to be able to use geolocation. This is a strange error, because I was able to add the column programmatically, while adding content to the list, is not possible. After checking the Diagnostic log, I noticed the following error:

**_A required component for using a geolocation field is not installed: Microsoft SQL Server System CLR Types_**

This remembered me that I had read _somewhere_ _(this means: It's not in the geo location MSDN documentation._[_It's written in the documenation on mobile apps and geolocation_](http://technet.microsoft.com/en-us/library/fp161355(v=office.15) "sharepoint 2013 mobile app development and geolocation")_)_ that the Sql Server System CLR Types needed to be installed on the Web Front End servers. The package that needs to be installed is called "SqlSysClrTypes.msi", and depending on the version of SQL that is used, one of the following feature pack needs to be installed:

[Sql Server 2008 R2 SP1 Feature Pack](http://www.microsoft.com/en-us/download/details.aspx?id=26728 "Sql Server 2008 R2 SP1 Feature Pack")

[SQL Server 2012 Feature Pack](http://www.microsoft.com/en-us/download/details.aspx?id=29065 "Sql Server 2012 Feature Pack")

After the installation, I still couldn't add any items to the list. An IIS Reset didnt work for me but a reboot did the trick.

_Update: I found the place where I read about the installation package_
