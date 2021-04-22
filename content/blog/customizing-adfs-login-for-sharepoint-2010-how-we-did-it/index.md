---
title: "Customizing ADFS login for SharePoint 2010: how we did it"
date: "2012-12-28"
categories: 
  - "authentication"
  - "code"
  - "sharepoint"
  - "sharepoint-2010"
---

In SharePoint 2010 the possibility of claims based authentication was introduced. The out of the box experience of this functionality is often OK, for example in cases of corporate intranets and extranets, but it doesn’t always fulfill the requirements of internet facing websites which require authentication.  This blogposts describes why we wanted to implement the active login scenario and learns us what kind of problems we encountered (and nailed ;))

## Login requirements

1. At our company we own a lot of different (insurance) labels and a big part of those labels want users to be able to login via a username  (for example their email-address) and password. When a user logs in with his username (e.g. user@outlook.com) on site of label A, he must be able to login with that same email address to site of label B. These accounts attached to the sites may not be the same physical account.
2. When a user logs in, he may never get redirected to another domain.
3. Login in when filling out a form, to retrieve his contact details that are already attached to his account, without losing context
4. Custom styling for the login control to be in line with the labels’ look and feel.
5. Being able to only supply ADFS login to endusers, but supply ntlm based authentication too, for the search service application. This is needed because our sites are hosted within Host named site collections (HNSC), multiple zones aren’t possible in here.
6. Because we are using HNSC, and don’t use any form of content deployment, we need to be able to offer authentication for our employees too, to be able to edit content. They may only be able to login from within the corporate network, otherwise, only a customer login must be offered.

### Non functional

- We want to make use of one single infra-structure for cost-reduction purposes. With tons of sites (and thus url’s), it isn’t very costs-effective to build a separate environment for every site.

## Passive login

The out of the box experience that comes with SharePoint 2010 is the so called “passive requestor profile” or “passive login” and works as displayed in the flow below. _Please note that users are being redirected to a Resource STS. SharePoint thinks it’s an Identity Provider, but in our infrastructure, this is not the case. Why, is explained further on._

[![](images/1184.passive-login-flow.png)](http://bloggingabout.net/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/bas/1184.passive-login-flow.png)

As seen in the diagram, whenever a user needs to login to a SharePoint site, the user is redirected up to 7(!!) times before he gets his page served. As this may be acceptable behavior when accessing a page to check out his customer information, this is not acceptable whenever a customer is in the 3rd step of a  flow when he wants to purchase an insurance. In addition to this: the OOTB functionality isn’t able to hide login options based on your source (in- or outside the corporate network)

## Claims based authentication infra-architecture for internet sites

To be cost effective, we have a single Resource STS for claims augmentation and a single IP-STS for the identity claims. . At the SharePoint STS we don’t make use of any claims augmentation of transformation, because this can lead to problems in a multi farm (publishing/consuming) setup.

[![](images/6443.cba_2D00_setup.png)](http://bloggingabout.net/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/bas/6443.cba_2D00_setup.png)

As seen in the diagram, the trusted Identity Provider for the SharePoint STS, is in fact, a resource STS. The real identity providers are federated with this resource-STS. This opens up the possibility to use different identity providers and handle resource claims augmentation on a single place. The problem with this setup, is that the federated STS-ses are accessible via only url. This is contrary to the requirement REQ2: a user may never leave the domain when logging in.

## Our solution

Our solution for this problem, is to create a custom ADFS login control that can be re-used on pages and/or webparts. This is called “Active requestor profile” or “Active Login”.  In fact, the solution exists out of two parts 1)    Determine user login source and type 2)    The login itself

### Determine user login source and type

Below is a small flow diagram to determine whether a user is a customer or an employee:

[![](images/2110.determineusertype.png)](http://bloggingabout.net/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/bas/2110.determineusertype.png)

“Determine origin” is executed without any user interaction. Whenever a user tries to login, we need to determine their origin. This can be done in various manners, but we decided to make use of the [X-Forwarded-For header](http://en.wikipedia.org/wiki/X-Forwarded-For "X Forwarded For"). It’s a HTTP-request header, that is inserted from our load balancer. External users get the address of the reverse proxy injected, internal users their own IP address. According to Wikipedia (and everything that’s on Wikipedia, is true ;)), it is the de facto standard for identifying the originating IP-address. Whenever the IP in the XFF HTTP header is not equal to the IP of the reverse proxy, a user selection type screen is show. Otherwise, only the customer login screen will be shown.

[![](images/0815.usertype-selection.png)](http://bloggingabout.net/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/bas/0815.usertype-selection.png)

### Active Login

As seen in the diagram below, this authentication method is much more user friendly, as there are just two redirections: the one to the login page that hosts the user login control and one redirection, back to the source URL, when the user has been authenticated. Please note that no difference between customers and employees has been made here, as the login logic is almost the same.

[![](images/0172.active-login.png)](http://bloggingabout.net/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/bas/0172.active-login.png)

The downside of this approach is that it requires custom code and needs to handle all the logic that ADFS was handling in the passive scenario:

- Get identity claims
    - Multiple identity providers – choose which IP to use (for example, based on selected userType)
        - Customers
        - Employees
        - Government
        - Etc.
- Augment resource claims
- Pass Token to SharePoint

Liam Cleary has done excellent research on this subject and wrote some code for it. I suggest that you read his[blogpost](http://blog.helloitsliam.com/Lists/Posts/Post.aspx?ID=76 "custom ADFS login"). The code is not production ready, but provides a great base to start out on your own active login user control.

#### Login control

Where liam’s code sample provided an application page with an asp.net login control we decided to create a custom login control, which subclassed System.Web.UI.WebControls.Login. We had a few reasons to subclass this control. One of those reasons is the fact that this Login control is used for forms based authentication. See the highlighted sections in the following, reflected, codesnippet:

[![](images/0841.attemptlogin.png)](http://bloggingabout.net/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/bas/0841.attemptlogin.png)

#### Formsbased authentication

As seen in the first highlighted section, this login control places a Forms Authentication cookie, which caused problems when trying to logout. After logging out, the cookie wasn’t deleted, which caused an error. Users had to close the browser and open it again to be able to login again.

#### Templating

Another reason was more practical. In the existing asp.net login control, templating is already implemented. By subclassing this control, a custom look and feel can easily be implemented, as the templated controls are inherited. The control can even be reused on any sitepage/masterpage/pagelayout: no application page is needed anymore. This makes it possible to give each label its own markup and messages:

[![](images/2068.login.png)](http://bloggingabout.net/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/bas/2068.login.png)

#### Subclassed ADFSLogin control

The subclassed ADFSLogin control is not too complex: the default OnBubbleEvent logic has been overridden, just to be able to implement custom AttemptLogin logic (this can’t be overridden as it is a private function. I didn’t want to use any form of reflection ;)). The AttemptLogin logic doesn’t contain much custom logic: it only got rid of the two highlighted sections.

#### Issues

- Redirecting after SPFederationAuthenticationModule.SetPrincipalAndWriteSessionToken will throw a ThreadAbortedException. When caught, the redirect will still occur and the login behavior is as intended.
- applying changes to the web.config isn't as trivial as expected. overwriting sections with SPWebConfigModifications won't work; another approach is needed for this. (MOre on this in a later blogpost)

## Summary

This article described what exactly passive and active login is and learned us why we chose for the active login scenario and what challenges we have met. As the standard login methodology doesnt satisfy the business requirements, another solution has been created. this solution has the following advantages:

- a custom adfs login control minimizes redirect traffic to a minimum
- own authentication logic can be implemented
- a custom adfs control provided ultimate flexibility to the business. If build properly, new IP- or Resource-STS'ses can be added on the fly.
- the control is ultimately stylable using, for example SharePoint designer.
