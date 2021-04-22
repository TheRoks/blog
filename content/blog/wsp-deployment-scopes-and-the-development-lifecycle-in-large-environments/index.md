---
title: "WSP Deployment scopes and the development lifecycle in large environments"
date: "2012-07-21"
categories: 
  - "sharepoint"
  - "sharepoint-2010"
tags: 
  - "deployment"
---

At our company, we use sharepoint to host all of our websites. As SharePoint is quite scalable, this is of course no problem and the production farm can handle it easily. These websites need to be developed and be tested, before they can be deployed to the production farm. This process is nothing special and doesn't diverge from most popular software development methodologies.

## Development Lifecycle

As stated before: Our development process is not something special: we develop, test it, the customer accepts the product and finally we take the product into production.

 

[![](images/8168.Deployment1.png)](http://bloggingabout.net/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/bas/8168.Deployment1.png)

 

Most of the developers that I know, develop their products on their own systems and they test it locally before the product is pushed to the development farm. At those companies, there is just one project (2 at most) which makes use of the development farm. The developers are free to deploy whatever and whenever they want, because they don't have to take other projects into account.

 

But our situation is quite different.We host a large amount of portals and websites (that number is still growing) and part of those existing portals are  continously upgraded. As opposed to the situation above (with a single project working at a farm), we do have 10+ projects working simulteaniously at a single farm.

 

## Deployment cycle

When just developing websites in ASP.Net, this isn't much of a problem, as all sites can easily be copied to it's location. As we all know, the deployment scenario in SharePoint, is different. Developers create WSP's, and those WSPs are deployed to the farm. In our case, code is checked into TFS, TFS builds the WSP's and some tooling, ROSS, gets triggered and deploys the generated WSP's to the development farm. That same tool deploys the solutions to Test, Acceptance and Production environments. This tool has been configured in such a way, that a solution only can be deployed to the production farm_,_ when it has passed the Development, Test and Acceptance stages.

 

[![](images/1070.Deployment2.png)](http://bloggingabout.net/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/bas/1070.Deployment2.png)

 

This automated deployment is very useful, but causes some problems for us. Due to the fact that we have multiple projects at the same central development farm, which makes it easy for developers to deploy solution pacakges, this causes a lot of retractions and deployments of WSP's. Every single WSP retraction and addition, causes the IIS application pools to recycle. This can (sometimes) take quite some time, can be frustating, and is, sadly enough, inherent to SharePoint Development. These deployments can cause other projects to recycle too. This is mainly due to the scope of the WSP's being deployed: when a WSP is deployed with a global scope, or is retracted, this affects ALL webapplications on the farm. Every single application gets a "free" application pool recycle. With two projects, this doesn't have to cause much problems: even when no deployment planning is made, those projects don't have to suffer much from each other.

 

[![](images/5127.deploymenttimeframe1.png)](http://bloggingabout.net/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/bas/5127.deploymenttimeframe1.png)

 

But when more projects come in, things get worse, as seen in the diagram below. The availability of all web applications gets lower, which results into less testing time per app, and each deployment of a WSP blocks all other projects from deploying/testing. It's not only annoying, it also cost's a lot of serious money. The next diagram is just an indication with 2 deployments per project a day and assumes that a deployment on a farm with 1 webapplication takes as long as a deployment to a farm with 10+ webapplications. In my experience this is not the case at all: the more applications that co-exist on a farm, the more time a (global) solution deployment takes.

 

[![](images/1205.deploymenttimeframe2.png)](http://bloggingabout.net/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/bas/1205.deploymenttimeframe2.png)

 

A solution for this problem is easy: split de big central Develoment farm into several small ones, as shown in the picture below:

 

 

[![](images/8561.Deployment3.png)](http://bloggingabout.net/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/bas/8561.Deployment3.png)

 

This is a situation that, in my opinion,  has a lot advantages (aside from the availability options), and fits into the cloud strategy of microsft (hosted development images), but doesn't solve all problems. Where the problems will disappear for developers, the problems will stay the same for the Test, Acceptance and Production environments. Not as much as at the development stage, but the deployments are still blocking other projects. The Test environment is heavily used by our testers and needs a lot of uptime, too. When looking from a business perspective: it's notacceptable that a commercial site, which sells products, has downtime because an other site isbeing deployed to production.

 

## The Solution

The solution to get rid of the blocking deployments, but keep the flexibility of deploying "any time", needs a small hack. The deployment scope of the solution needs to be changed from "Global" to "WebApplication". This is not a setting in the package property window, but is determined "deploy time". In short: whenever there is an artefact or configuration option that needs to be deployed at the webapplication scope, the solution scope is "webapplication". In all other situations, the WSP will be globally deployed. The easiest way to do this, is to manually add a dummy safecontrol to the manifest.xml.  _Note: this doesn't work in all cases. There are a few situations where a solution only can globally be deployed to a farm. Try to bundle these functionalities into a single package, to keep downtime as low as possible._

 

[![](images/2604.deploymenttimeframe3.png)](http://bloggingabout.net/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/bas/2604.deploymenttimeframe3.png)

 

As seen in the picture above, in the most ideal situation, projects don't block each other from testing. The only influence that projects have on each other, is the fact that wsp's can't be deployed simultaneously.

 

## Conclusion

SharePoint solution deployments can be quite time consuming, and when not properly planned and configured, it can even block other projects when working simultaneously on a farm. The more projects that are using the farm, the more projects that can suffer from each other. A simple solution for this, is to force the solutions to be deployed at webapplication scope. This can be achieved by manually adding a safecontrol to the project.
