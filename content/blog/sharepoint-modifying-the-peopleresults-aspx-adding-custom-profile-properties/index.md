---
title: "SharePoint: Modifying the PeopleResults.aspx - adding custom Profile Properties"
date: "2010-08-21"
categories: 
  - "peoplesearch"
  - "search"
  - "sharepoint"
  - "userprofile"
---

To modify the results of the peopleresults.aspx, a few stepts have to be taken.  
In my case, a Division is added to the results.

at first, make sure that the property that you want to display, is indexed so it will be part of the People Search Scope: Go [http://MOSS2007/ssp/admin/\_layouts/MgrProperty.aspx](http://moss2007/ssp/admin/_layouts/MgrProperty.aspx) and select or add the UserProfileProperty that you want to be part of the search result. Check the indexed box.

[![](images/4705.indexed.JPG)](http://bloggingabout.net/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/bas/4705.indexed.JPG)

Open your own userprofile, and add some data in the field.

After this, start a new full crawl on the contentsource that has the sps:// (people search) tag in it. When its finished, you will find the Division back in your metadata properties:

[![](images/6165.metadataproperty.JPG)](http://bloggingabout.net/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/bas/6165.metadataproperty.JPG)

Now you will be ready to modify your peopleresults.aspx:  
Go to this page, and open the page in edit mode. FInd the searchresults box, and open this one in edit mode, too. Press the "XSL-editor" button in here.  
in here, you can edit the xsl to add the custom userprofile properties to the results.  
I am gonna add this to the office information (I know it doesnt belong there, but hey, its just an example):

find the psrch-Description, and add an xsl:with-param names divi, which selects the division userprofile property from the peoplesearch results.

```xml
<div class="psrch-Description">
                        <xsl:call-template name="DisplayOfficeProfile">
                            <xsl:with-param name="title" select="jobtitle" />
                            <xsl:with-param name="dep" select="department" />
                            <xsl:with-param name="phone" select="workphone" />
                            <xsl:with-param name="office" select="officenumber" />
                            <xsl:with-param name="divi" select="division" />
                        </xsl:call-template>
</div>
```

now, find the DisplayOfficeProfile section and add a xsl-param name and a check + value of that parameter to display it in the results:

```xml
<xsl:template name="DisplayOfficeProfile">
        <xsl:param name="title" />
        <xsl:param name="dep" />
        <xsl:param name="phone" />
        <xsl:param name="office" />
        <xsl:param name="divi" />
        <span class="psrch-Metadata">
            <xsl:if test='string-length($title) &gt; 0'>
                <xsl:value-of select="$title" />
                -
            </xsl:if>
            <xsl:if test='string-length($dep) &gt; 0'>
                <xsl:value-of select="$dep" />
                -
            </xsl:if>
            <xsl:if test='string-length($phone) &gt; 0'>
                <xsl:value-of select="$phone" />
                -
            </xsl:if>
            <xsl:if test='string-length($office) &gt; 0'>
                <xsl:value-of select="$office" />
                -
            </xsl:if>
            <xsl:if test='string-length($divi) &gt; 0'>
                <xsl:value-of select="$divi" />
            </xsl:if>
        </span>
        <br/>
    </xsl:template>
```

Save the template and press apply. Prepare for an error, but don't worry: publish the page and reload it, and your peopleresults will be updated with the custom property
