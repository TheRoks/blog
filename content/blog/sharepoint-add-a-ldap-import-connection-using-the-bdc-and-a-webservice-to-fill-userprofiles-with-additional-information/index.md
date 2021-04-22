---
title: "SharePoint: Add a LDAP import connection using the BDC and a webservice to fill userprofiles with additional information"
date: "2009-03-18"
categories: 
  - "ad"
  - "bdc"
  - "ldap"
  - "sharepoint"
  - "ssp"
  - "userprofile"
  - "webservice"
---

At the company that I work for at the moment, we have several AD's and a separate LDAP store with additional user information. For that company we are developing a new Intranet in which we want to add their accounts (account + email) and provision their userProfiles with a lot of extra information like their manager, location, division, telephone numbers etc. Adding a AD import connection and a LDAP import connection isn't possible, because the accountName isnt stored into the LDAP directory, but they unique key that is used is the email address.

In this post I will explain how to write a webservice to retrieve AD information and how to create a BDC to use this webservice to provision the UserProfiles with data.

At first, we need to create a webservice. Add a reference to System.DirectoryServices and create tweo classes:

one to create a UserProfile that is returned, and on to make a LDAP connection. In my case, they are called "UserProfile and LDAP"

\[Serializable\]
public class UserProfile
    {
        public string Email { get; set; } // Unique for each profile
        public string Location { get; set; }
        public string Manager { get; set; }
 
        // assumption that all LDAP values are strings and single value
        public UserProfile FillUserProfile(SearchResult sr)
        {
            Email = sr.Properties\["mail"\]\[0\] as string;
            Manager = sr.Properties\["manager"\]\[0\] as string;
            Location = sr.Properties\["location"\]\[0\] as string;
            return this;
        }
    }
 
 
public class LDAP
    {
        /// <summary>
        /// The server domain name without any protocol specification.
        /// </summary>
        public string LdapServer { get; private set; }
        /// <summary>
        /// The user name for connecting to LDAP without any attributes
        /// </summary>
        public string LdapUsername { get; private set; }
        /// <summary>
        /// The password for connecting to LDAP
        /// </summary>
        public string LdapPassword { get; private set; }
        /// <summary>
        /// The block size for reading search results from LDAP
        /// </summary>
        public int LdapPageSize { get; private set; }
 
        public DirectoryEntry LdapDirectoryServer { get; private set; }
        public DirectorySearcher LdapDirectorySearcher { get; private set; }
 
        public LDAP()
        {
            LdapServer = string.Format("LDAP://{0}", ConfigurationManager.AppSettings\["LdapServer"\]);
            LdapUsername = string.Format("uid={0}", ConfigurationManager.AppSettings\["LdapUsername"\]);
            LdapPassword = ConfigurationManager.AppSettings\["LdapPassword"\];
            LdapPageSize = Convert.ToInt32(ConfigurationManager.AppSettings\["LdapPageSize"\]);
            // Connect tro LDAP
            LdapDirectoryServer = new DirectoryEntry(LdapServer, LdapUsername, LdapPassword, AuthenticationTypes.ServerBind);
            LdapDirectorySearcher = new DirectorySearcher(LdapDirectoryServer);
            // Filter, should maybe be qualified further to return only the entries for people           
            LdapDirectorySearcher.PageSize = LdapPageSize;
            AddLdapDirectorySearcherProperties(LdapDirectorySearcher);
        }
 
        private static void AddLdapDirectorySearcherProperties(DirectorySearcher LdapDirectorySearcher)
        {
            LdapDirectorySearcher.PropertiesToLoad.Add("mail");
            LdapDirectorySearcher.PropertiesToLoad.Add("manager");
            LdapDirectorySearcher.PropertiesToLoad.Add("location");
        }
 
        private SearchResult GetSearchResult(string email)
        {
            LdapDirectorySearcher.Filter = "(mail=" + email + ")";
            return LdapDirectorySearcher.FindOne();
        }
 
        public UserProfile FindUser(string email)
        {
            SearchResult sr = GetSearchResult(email);
            if (sr != null)
            {
                return new UserProfile().FillUserProfile(sr);
            }
            return null;
        }
    }

now create the Webservices. Three Methods are required, but we are just going to need one working method, the one that finds a userProfile when a email address is specified:

\[WebService(Namespace = "http://tempuri.org/")\]
    \[WebServiceBinding(ConformsTo = WsiProfiles.BasicProfile1\_1)\]
    public class BDCWebService : System.Web.Services.WebService
    {
        \[WebMethod(Description = "Returns null")\]
        public List<string> GetAllUserEmail()
        {
            /\*
            \* IdEnumerator
            \*/
            return null;
        }
 
        \[WebMethod(Description = "Returns a single User Profile Entity by by specifying an Email address")\]
        public UserProfile GetUserProfileByEmail(string email)
        {
            /\*
            \* Specific Finder
            \*/
            LDAP ldap = new LDAP();
            return ldap.FindUser(email);
 
        }
 
        \[WebMethod(Description = "returns null")\]
        public List<UserProfile> GetUserProfiles()
        {
            return null;
        }
    }

You an test your LDAP connection settings, username and password by starting your webservice, select the GetUserProfileByEmail method and specify your own email address. If it works, the next step can be taken: creating the BDC.

To create a BDC resource file, you need to download the SharePoint resource kit and install the ApplicationDefinitionDesigner.

Open up the ApplicationDefinitionDesigner and  click on "Add LOB system -> Connect to webservice". Add the location of your webservice and click "connect"

[  
![](images/1234.1-_2D00_-specify-url.JPG)](http://bloggingabout.net/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/bas/1234.1-_2D00_-specify-url.JPG)

Right click on Entities -> Add Entities -> OK -> Add WebMethod.  
Drag the three webmethods on the blue canvas one by one.  
[![](images/4846.2-_2D00_-Add-entities.JPG)](http://bloggingabout.net/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/bas/4846.2-_2D00_-Add-entities.JPG)  
Click on Entity 0 and modify its name into UserProfile  
RightClick on Identifier and add an identifier. Name it "Email" as well its displayname.  
Open up methods -> GetAllUserEmail -> return -> return and RightClick on it -> Add TypeDescryptor. Select as identifier: Email\[UserProfile\]  
Open up methods -> GetUserProfileByEmail -> email -> email and set it's identifier to  Email\[UserProfile\]  
do the same for GetUserProfileByEmail -> return -> return -> email and set it's identifier to  Email\[UserProfile\]

Open up methods -> GetUserProfiles -> return -> return -> item -> email and set it's identifier to Email\[UserProfile\]

add instances:  
1) open up Methods -> GetAllUserEmail -> instances and Add an instance. Choose as type IdEnumerator and set its name as "GetAllUSerEmailInstance"  
2) open up Methods -> GetUserProfileByEmail -> instances and add an instance. Choose specificFinder as type and name it: GetUserProfileByEmailInstance  
3) open up Methods -> GetUserProfiles -> instances and add an instance. Choose finder as type and name it: GetUserProfilesInstance

Rightclick on the rootnode and choose "export". Save it to your HDD.

Now we are ready to import the BDC:  
open your SharedService Provider and navigate to Import Application Definition. Select your model and import it.  
Navigate to manage connections and Create a new connection:  
[![](images/5621.3-_2D00_-add-BDC-connection.JPG)](http://bloggingabout.net/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/bas/5621.3-_2D00_-add-BDC-connection.JPG)

at last you need to map your user profile properties to the BDC mapping.  
When you are doing a new (incremental or full) import of your user profiles, the BDC import will start right after the AD import has been finished.

Please keep in mind to give every account enough rights to update the user profiles!  
1) give the AppPoolID rights on the BDC connection  
2) make sure the account has enough rights on the sharepoint\_config and ssp\_config db, otherwhise, the userprofiles some exceptions will be thrown.
