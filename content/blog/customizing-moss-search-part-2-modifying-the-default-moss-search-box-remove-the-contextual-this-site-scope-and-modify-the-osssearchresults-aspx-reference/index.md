---
title: "Customizing MOSS Search - Part 2: Modifying the default Moss Search box, remove the contextual (this site) scope and modify the OSSSearchResults.aspx reference"
date: "2009-06-22"
categories: 
  - "search"
  - "sharepoint"
---

When you want to customize the default **WSS Search** box, this isn't too hard. It ships into the ContentLightUp feature, which contains a feature.xml and a searcharea.xml. In that searcharea.xml a reference to the searcharea.asxc exists. Because SharePoint works with **delegates** with searchboxes, all you need to do is copy that feature, modify its guid, lower the sequence (so that control will be loaded) and change the reference to your CustomWSSsearcharea.ascx.

<Control  
        Id="SmallSearchInputBox"  
        Sequence="99"   
        ControlSrc="~/\_controltemplates/CusomWSSsearcharea.ascx">  
    </Control>  
Modifying the OOB Moss Search Control is different. The MOSS Feature consists of the same files: a feature.xml and a searcharea.xml. The searcharea.xml doesn't a contain a reference to a user control, but to the SearchBoxEx class in a certain assembly. It isn't possible to style this control too much, because the output html is a tablerow. Luckily, this class isn't sealed, so it's easy to derive from this searchBox. Michiel Lankamp did this before, and reused the original controls of the searchbox, but put them into some div's. I modified it a bit in such a way that it did work for me ;). He hided the Table with the search controls and rebuilded a structure from divs, where he placed the used controls in. This gives a lot more flexibility and the original Properties can be reused. Please note that the only thing you need to do, is override the CreateChildControls..

protected override void CreateChildControls()
{       
    // build the original controls first: we can reuse some of them
    base.CreateChildControls();
 
    // if no controls exist, return
    if ( Controls.Count < 1 )
    {
        return;
    }
 
 
    HyperLinkLoc hlink = null;
 
    // as the output html is a tablerow, we have to find a table with a tablerow in it ;) 
    foreach (Control c in Controls)
    {
        if (c is Table)
        {
            var table = c as Table;
 
            // Loop through cells to find the go button
            foreach (TableCell cell in table.Rows\[0\].Cells)
            {
                if (cell.Controls.Count > 0)
                {
                    if (cell.Controls\[0\] is HyperLinkLoc)
                    {
                        hlink = cell.Controls\[0\] as HyperLinkLoc;
                        break;
                    }
                }
            }
            //hide the table, because we are going to rebuild the search control
            table.Visible = false;
 
            //add new controls
            if (m\_ddlScopes != null)
            {
                var scopeDiv = new HtmlGenericControl("div");
                scopeDiv.Attributes.Add("class", "custom-search-dropdown");
                scopeDiv.Controls.Add(m\_ddlScopes);
                Controls.Add(scopeDiv);
            }
 
            if (m\_searchKeyWordTextBox != null)
            {
                var searchDiv = new HtmlGenericControl("div");
                searchDiv.Attributes.Add("class", "custom-searchTextBox");
                searchDiv.Controls.Add(m\_searchKeyWordTextBox);
                Controls.Add(searchDiv);
            }
 
            if (hlink != null)
            {
                var goDiv = new HtmlGenericControl("div");
                goDiv.Attributes.Add("class", "custom-searchGo");
                goDiv.Controls.Add(hlink);
                Controls.Add(goDiv);
            }
 
            if (ShowAdvancedSearch)
            {
                var advancedDiv = new HtmlGenericControl("div");
                advancedDiv.Attributes.Add("class", "custom-advancedSearch");
                var hl = new HyperLink
                {
                    Text = "Advanced Search",
                    NavigateUrl = AdvancedSearchPageURL
                };                       
                advancedDiv.Controls.Add(hl);
                Controls.Add(advancedDiv);
            }
            break;
        }               
    }
}

Using this method, all of the original Properties can be reused.  
Two of those properties are asked a lot about:  
DropDownMode and SearchResultPageURL

When wanting to search just the site that you are visiting, it could be an option to hide the WSS Search Contextual Scope "this site: <sitename>". This can be disabled by setting the property **_"DropDownMode"_** to**_"ShowDD\_NoContextual"_**. of course this can be done in code or in the feature ;). When this scope _is_ used, the visitor is redirected to the  OSSSearchResults.aspx. this is a out of the box search results page, which resides in the layouts directory on the file system. A lot of people want to customize this page, or want to have another searchresults page which handles their query. According to the documentation, this can be done by modifying the  SearchResultPageURL property. By default, this is the OSSSearchResults.aspx. When changing this into CustomSearchResults.aspx, it is expected that the redirect does work. But for some reason, this isn't the case. When the (obfuscated) base.CreateChildControls has been hit, the redirected page for the Contextual search is set back to OSSSearchResults.aspx.

After a bit hacking around, It made clear that the property "UseSiteDefaults" was set to true.After putting it to false, I was finally redirected to my custom searchresultspage, but On all Scopes. The Search Centre that i had specified in the sitecollection, was completely neglected. Specified scopes and people search didn't work anymore, so using this parameter was not an option for me. Surfing the internet gave me the following 2 options:

- create a HTTP handler to redirect
- write an own searchbox class / user control with some own javascript to handle the scopes.

Both options werent the solution for me, because:

- for every request for a page, the HTTP Handler verifies the request URL and decides wether or not it needs to be redirected
- Writing an own search box control is too much work. Being a developer, I am lazy. And: "Je deteste javascript un peux" ;) (and don't tell me my french sucks ;))

For some reason, I didn't find any solution on the whole internet, that provided an easy to implement solution, that redirected the OSSSearchResults.aspx _and_  was able to use the site defaults, so that no extra work on scoping was needed. Suddenly I got the solution!  
This can be done with **reflection**, I managed to set the private member of the SearchBoxEx Class that stored the link to the OSSSearchResults.aspx. The only caveat that exists, is that you need the ReflectionPermission. When the DLL is in the GAC and/or the web.config trust level is set to full, this isnt too much of a problem (normally ;)). If it isn't you will need to deliver a permissionset that allows your assembly or class to access the Non-public members when reflecting that class. Below is the code to redirect to the right page:

protected override void CreateChildControls()
{
    // the private member that stores the redirect address
    const string memberFIeld = "m\_strOssSearchResultsUrl";
 
    // store the redirect url for later.
    // when debugging, the SearchResultPageUrl from the Feature Properties, is CustomOSSSearchResults.aspx
    var Url = SearchResultPageURL;
  
    // at this moment, the search centre is set, and all links are filled:
    // for example, SearchResultPageURL is set to OSSSearchResults.aspx(altough we specified CustomOSSSearchResults.aspx.
    // this is because "UseSIteDefaults"  is set to true in the feature settings.
    base.CreateChildControls();
 
    // In the code below, we try to find the private Member "m\_strOssSearchResultsUrl" of SearchBoxEx, the
    // class that is derived from.
    // Make sure to have reflectionPermission, otherwhise the BindingFLags.NonPublic is ignored!
    // when running in GAC or having full trust in the web.config, this isnt an issue
    Type baseType = GetType().BaseType;
    FieldInfo fi = baseType.GetField(memberFIeld, BindingFlags.NonPublic | BindingFlags.Instance);
    if (fi != null)
        fi.SetValue(this, Url); // replaces URL with our redirection
}
