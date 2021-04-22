---
title: "LDAP Problems updating LDAP information: saving attribute values gives error: 0x8000500C - The Active Directory Datatype Cannot be Converted to/from a Native DS Datatype / LDAPConnection: The LDAP server is unavailable"
date: "2009-03-22"
categories: 
  - "ad"
  - "ldap"
  - "sharepoint"
  - "userprofile"
---

Our customer had the wish to synchronise some Sharepoint UserProfileProperties of UserProfiles back to LDAP, so that information could be synced with SAP. The attributes that needed to be saved, where inherited from custom schemas in LDAP, which gave us some trouble. When updating attribute values in LDAP, and they inherit from the default scheme, there is really no problem. But at the moment that we tried to update an attribute that inherited from a custom schema, we saw some very weird behaviour:

private void UpdateAttribute(SearchResult sr)

        {

            DirectoryEntry de = sr.GetDirectoryEntry();

            if(de.Properties.Contains("customProperty"))

            {

                de.Properties\["customProperty"\]\[0\] = "bla";

            }

            de.CommitChanges();

        }

threw a com exception: **0x8000500C - The Active Directory Datatype Cannot be Converted to/from a Native DS Datatype.** It even threw the exception at the line:

if(de.Properties.Contains("customProperty"))

When trying to access a property that didn't exist or a property that inherited from the default schema, no exception was thrown. What is the case? There are 2 possibilities:

1) the schema cache can't be updated on our machine, and with the lack of a correct schema, the datatype couldn't be converted.  
2) For some reason, LDAP V2 is used instead of V3, which has the problem that custom attributes can't be loaded

We didn't find a solution for this, but googled brought me an alternative solution:

the System.DirectoryServices.Protocols namespace!  
I tried to make a connection to the LDAP server with the following Code:

LdapDirectoryIdentifier id = new LdapDirectoryIdentifier("LDAP://" + ConfigurationManager.AppSettings\["LdapServer"\], 389);
LdapConnection \_LdapConnection = new LdapConnection(id, AuthType.Basic);
            \_LdapConnection.SessionOptions.ProtocolVersion = 3;
            \_LdapConnection.Bind(); 

But this threw another error: The LDAP server is unavailable. 2 hours of trial and error brought me to the conclusion: leave the "LDAP://" when specifying the server :S

Using LdapDirectoryIdentifier id = new LdapDirectoryIdentifier(ConfigurationManager.AppSettings\["LdapServer"\], 389); everything worked fina, and now we have a 2-way working synchronization between a LDAP store and a SharePoint UserProfileStore!
