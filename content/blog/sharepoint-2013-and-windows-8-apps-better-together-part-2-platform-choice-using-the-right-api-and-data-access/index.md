---
title: "SharePoint 2013 and Windows 8 apps - better together Part 2: Platform choice, using the right API and data access"
date: "2013-03-17"
categories: 
  - "code"
  - "rest"
  - "sharepoint"
  - "windows-8"
---

This is the second post as part of a blog series about the integration of using SharePoint 2013 as a datasource for windows 8 apps.Find the index at [SharePoint 2013 and Windows 8 apps - better together Part 1: Introduction, background and considerations](http://blog.baslijten.com/sharepoint-2013-and-windows-8-apps-better-together-part-1-introduction-background-and-considerations/ "Part 1: Introduction, background and considerations")

**NOTE: this blogpost provides information on how to consume rest-services via c-sharp. A lot of plumbing was done by [wictor wilen on authentication](http://www.wictorwilen.se/Post/How-to-do-active-authentication-to-Office-365-and-SharePoint-Online.aspx), [Luc Stakenburg](http://allthatjs.com/2012/03/28/remote-authentication-in-sharepoint-online/) and [jmservera](http://sharepointwinrt.codeplex.com/). In the following blogpost I'll show how we used that information and optimized the code by jmservera (a bit).**

#### Choice of platform

For the choice on the platform, there are just a few choices:

• Locally • On permise installation • Office 365 The on premise installation was not an option at all, as the existing infrastructure wasn't available to us at the time that we needed it. That left us the two other options. Because there was just the need to add/read data and consume some services, the decision was an easy one: All the functionality that was needed was available within office 365. As Microsofts direction to the cloud is pretty clear, this was a smart choice. Note: sign up for a developer account at [http://msdn.microsoft.com/en-us/library/office/apps/fp179924(v=office.15)](http://msdn.microsoft.com/en-us/library/office/apps/fp179924(v=office.15).

#### Api Choice

##### Data entry

To be able to expose data to the windows 8 app, the first action was to get data into the SharePoint site. The following chart helped us out to make the choice for the right API. As we didn't have access to the server, PowerShell or the Server OM API wasn't an option. Our choice was to use the .Net CSOM, which was used to quickly build a .Net Console application. As a lot of articles have been written on this subject, I will skip that part in this article. (use MSDN for more information on this subject)

[![](images/1300.sp2013api.png)](http://bloggingabout.net/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/bas/1300.sp2013api.png)

source: [MSDN](http://msdn.microsoft.com/en-us/library/jj164060.aspx)

##### Windows 8 app

Because the windows 8 app was already build in C#/.Net, two possibilities were left: the .Net CSOM and the REST/Odata EndPoints. As the .net CSOM API is quite easy to use, well documented and provides us the possiblity of using Linq to objects, we tried to use that API. (download the SharePoint 2013 Client Components SDK [here](http://www.microsoft.com/en-us/download/details.aspx?id=35585)). Too bad a unsolvable error showed up after the first build: Cannot resolve assembly or Windows Metadata file "System.Web.Services.dll". The reason behind this error is because of the dll not being available within the .Net framework for Windows Store Apps. This forced us to use the REST/Odata EndPoints within our application.

[![](images/8272.cannotresolveAssembly.png)](http://bloggingabout.net/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/bas/8272.cannotresolveAssembly.png)

#### REST/OData

With the introduction of SharePoint 2013, Microsoft introduced several REST based service interfaces: • /\_api/site/ • /\_api/web/ • /\_api/search/ • /\_api/SP.UserProfiles.PeopleManager/ • /\_api/publishing/ We made use of both the web and the search REST API. Whereas building up the request is quite easy, consuming the data can be challenging, when not making use of some amazing tools that are available out there. As microsoft used the [OData](http://www.odata.org/) protocol for its REST-service, 2 types of data representations can be used: ATOM or JSON. In this application, the JSON format was used because it is more lightweight. In the following paragraphs I'll tell what kind of tooling we made use of.

##### Getting the data - SharePointWinRT + modifications

Before a restful service can be consumed, a connection needs to be setup. Someone (jmservera) did already do a lot of the plumbing to setup an authenticated connection for office 365. Make sure to check out his codeplex project: [http://sharepointwinrt.codeplex.com/](http://sharepointwinrt.codeplex.com/). In a later blogpost I'll tell how we optimized parts of that library and what parts we optimized. Our altered library contains a few helper functions and one of them is a function to asynchronously get a JSON string from a stream (authentication is handled within that function):

[![](images/6747.asyncGetJSON.png)](http://bloggingabout.net/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/bas/6747.asyncGetJSON.png)

As a JSON object is nothing more than plain text, it's initially impossible to work with typesafe objects. Writing a custom parser causes a lot of work and is very error prone. Luckily, there are several good tools to solve this problem!

##### TypeSafe objects from JSON strings

To parse the JSON (type-unsafe) object into a typesafe object, several steps need to be taken. At first, a datamodel must be created. Several options are possible here:

###### Json2csharp.com

At the time of developing the windows 8 app, we were aware of just one solution: [http://json2csharp.com/](http://json2csharp.com/). This is a website that consomes data in JSON format and generates C# classes from it (note, use fiddler to get the json messages):

[![](images/7801.json2Csharp.png)](http://bloggingabout.net/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/bas/7801.json2Csharp.png)

###### ASP.NET and Web Tools 2012.2 RTM

Another solution that I was unaware of, shipped with the ASP.Net and Web Tools 2012.2 RTM and it's called "paste JSON as c# classes":

[![](images/7532.pasteJsonAsClasses.png)](http://bloggingabout.net/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/bas/7532.pasteJsonAsClasses.png)

This results (also) in a set of generated .Net classes which can be used in conjunction with the JSON data object. It provides the same functionality as json2chsarp does, but it's completely integrated with visual studio! Now that the datamodels are created, the JSON object can be parsed into that new datamodel.

##### Using JSON.Net to finally parse the JSON string

JSON.Net is a codeplex project that allows to us to convert JSON to .Net objects. The previously generated classes are needed to parse the JSON string into. The string gets deserialized and the root object ends up being instantiated. Not only is does JSON.Net provide much more functionalities than the built-in solution, it's much more faster too.

[![](images/4540.Deserialize.png)](http://bloggingabout.net/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/bas/4540.Deserialize.png)

Cool thing to notice: LINQ can be used on the returned object:

[![](images/0624.LINQ.png)](http://bloggingabout.net/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/bas/0624.LINQ.png)

#### Using the data

Now the data is parsed into a type-safe object, which can be queried with LINQ, the data can easily be integrated with the Windows 8 application using the common databinding methods that are usually used in windows 8 apps.

#### Summary

This blogpost showed how easy it is to consume rest-services within a windows 8 application. Using the followin steps, it's easy to convert a string containing a JSON object, to a typesafe object on which a LINQ query can be executed. • Authenticate and get data from O365 using the SharePointwinRT library ([http://sharepointwinrt.codeplex.com/](http://sharepointwinrt.codeplex.com/)) • Generate .Net classes using the ASP.Net and web tools "Paste JSON as classes functinality ([http://go.microsoft.com/fwlink/?LinkID=275131](http://go.microsoft.com/fwlink/?LinkID=275131)) • Parse the retrieved json string into the datamodel using JSON.Net ([http://json.codeplex.com/](http://json.codeplex.com/))

Next blogpost shows us how the search capabilities of SharePoint 2013 are used within this application. It contains both a technical as a functional part.

Happy coding!
