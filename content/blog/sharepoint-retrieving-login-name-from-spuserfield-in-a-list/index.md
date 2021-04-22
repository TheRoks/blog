---
title: "SharePoint - Retrieving login name from SPUserField in a list"
date: "2009-07-17"
categories: 
  - "caml"
  - "sharepoint"
---

For a long time I didnt know how to retrieve a login name from a SPUser field in a SharePoint List. When a user doesn't have a userProfile, he is stored in the SPUser field as <number>;#<Domain\\loginName>, and when he does have a profile, his name is stored as <number>;#<name of user>. That number doesn't represent the ID in the User Profile Provider, so it wasn't possible to use that ID to retrieve the login name (or other user information).

Today, I accidentally came accros an option that allowed me to retrieve all the "standard" user information:

ID, name of user, loginname, email and some other extra information.

When making a new SPQuery, do the following:

query.ExpandUserField = true;

this option enables the access op extra user information possible. Why didn't I know this before, stupid me ;)?
