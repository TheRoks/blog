---
title: "Using the CrossListQueryInfo and CrossListQueryCache"
date: "2009-03-27"
---

Lately, I am relatively a lot working with the CrossListQueryInfo. This is a way to query multiple lists at once, crossing multiple SPWebs, if you want to. And, the main reason why I used it, it's possible to use the cached lists of the sitecollection, to improve performance! The first steps with this CrossListQueryInfo can be quite frustrating, as there isnt too much documentation to find on.

I will start with a code sample, to show how to use this to query multuple webs to retrieve Items with the contentType News Item OR Location News Item from the publishing pages. When you are going to the CrossListQueryInfo, it´s smart to make use of U2U´s Caml Query Builder, to quickly build up thecaml that you want use.  
Things to keep in mind while using the CrossListQueryInfo:

- The results that are returned are ONLY PUBLISHED items. I was breaking my head around this one, because I was sure that my query was good, the CAML Query builder also returned a result, but theCrossListQueryCache didn't. After publishing, all the results showed up properly
- ViewFields: Insert the fields that you want to see. If there is a field inside that doesnt exist in the list that you query, your result will be nill, nada, nothing. Make sure that you put in the INTERNAL field names!
- RowLimit: When filling in value '0', you won't see any results
- When using the CrossListQueryCache , make sure not to use the GetSiteData that uses the SPWeb param: GetSiteData(SPWeb web) and GetSiteData(SPWeb web, SPSiteDataQuery query) DO NOT use caching!!
- and make sure to use [Microsoft.SharePoint.WebPartPages.WebPart](http://msdn2.microsoft.com/en-us/ms461685)! This WebPart inherits from [System.Web.UI.WebControls.WebParts.WebPart](http://msdn2.microsoft.com/en-us/h0t1fxe7) and enhances it with some extra functionality for connected webparts, client-side programming and data caching!! Thanks to [Waldek Mastykarz](http://blog.mastykarz.nl/) for sharing this info.

Below is an example of a query that queries all the Publishing pages libraries that resides in the current SPWeb and all childWebs.

private DataTable ExampleQuery()

        {

            clqi = new CrossListQueryInfo();

 

            // Insert the list types that you want to use. In this case, its the publishing page library (850, see code below)

            clqi.Lists = "<Lists ServerTemplate=\\"" + (int)ListServerTemplateCodes.PageLibrary + "\\" />";

 

            // Insert the fields that you want to see. If there is a field inside that doesnt exist in the list that you query, your result will be nill, nada, nothing.

            // Make sure that you put in the INTERNAL field names!

            clqi.ViewFields = "<FieldRef Name=\\"Title\\" /><FieldRef Name=\\"FileLeafRef\\" /><FieldRef Name=\\"Nieuws\_x0020\_Type\\" /><FieldRef Name=\\"Nieuws\_x0020\_Leverancier\\" /><FieldRef Name=\\"Uitgelicht\\" /><FieldRef Name=\\"Created\\" /><FieldRef Name=\\"Comments\\" />";

 

            // scop to use. Another possibility is SiteCollection

            clqi.Webs = "<Webs Scope=\\"Recursive\\" />";

 

            // turn the cache on

            clqi.UseCache = true;

 

            // if row limit == 0, you will get 0 results

            clqi.RowLimit = 100;

 

            // I know a stringbuilder would be better, but i wanted to show the markup of the query

            clqi.Query = "<OrderBy>" +

                            "<FieldRef Name='Title' />" +

                        "</OrderBy>" +

                        "<Where>" +

                            "<Or>" +

                                "<Eq>" +

                                    "<FieldRef Name='ContentType' />" +

                                    "<Value Type='Text'>News Item</Value>" +

                                "</Eq>" +

                                "<Eq>" +

                                    "<FieldRef Name='ContentType' />" +

                                    "<Value Type='Text'>LocationNews Item</Value>" +

                                "</Eq>" +

                            "</Or>" +

                        "</Where>";

 

            // put the CrossListQueryInfo object into the CrossListQueryCache

            CrossListQueryCache clqc = new CrossListQueryCache(clqi);

 

            // and query the data!

            // make sure: the GetSiteData(SPWeb web) and GetSiteData(SPWeb web, SPSiteDataQuery query) DO NOT use caching!!!

            DataTable tbl = clqc.GetSiteData((SPContext.Current.Site, CrossListQueryCache.ContextUrl());

 

            // return the datatable

            return tbl;

        }

The enum below can be used to make life a little bit easier. I got it from:[http://www.aspenhorizons.com/devblog/?p=29](http://www.aspenhorizons.com/devblog/?p=29)

public enum ListServerTemplateCodes
    {

        GenericList = 100,

        DocumentLibrary = 101,

        Survey = 102,

        LinksList = 103,

        AnnouncementsList = 104,

        ContactsList = 105,

        EventsList = 106,

        TasksList = 107,

        DiscussionBoard = 108,

        PictureLibrary = 109,

        DataSources = 110,

        SiteTemplateGallery = 111,

        UserInformationList = 112,

        WebPartGallery = 113,

        ListTemplateGallery = 114,

        XMLFormLibrary = 115,

        MasterPagesGallery = 116,

        NoCodeWorkflows = 117,

        CustomWorkflowProcess = 118,

        WikiPageLibrary = 119,

        CustomGridForAList = 120,

        DataConnectionLibrary = 130,

        WorkflowHistory = 140,

        GanttTasksList = 150,

        MeetingSeriesList = 200,

        MeetingAgendaList = 201,

        MeetingAttendeesList = 202,

        MeetingDecisionsList = 204,

        MeetingObjectivesList = 207,

        MeetingTextBox = 210,

        MeetingThingsToBringList = 211,

        MeetingWorkspacePagesList = 212,

        PortalSitesList = 300,

        BlogPostsList = 301,

        BlogCommentsList = 302,

        BlogCategoriesList = 303,

        PageLibrary = 850,

        IssueTracking = 1100,

        AdministratorTasksList = 1200,

        PersonalDocumentLibrary = 2002,

        PrivateDocumentLibrary = 2003

    }

more information about this subject can be found at:

[http://sharepoint.nailhead.net/2008/04/musing-on-crosslistquerycache-class.html](http://sharepoint.nailhead.net/2008/04/musing-on-crosslistquerycache-class.html)
