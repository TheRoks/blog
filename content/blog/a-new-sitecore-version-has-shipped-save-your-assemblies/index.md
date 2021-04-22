---
title: "A new Sitecore version has shipped, save your ass(emblies)"
date: "2017-10-19"
coverImage: "assemblies.png"
---

With the release of Sitecore 9.0 at the Sitecore Symposium, a lot has changed. We saw very interesting improvements on Marketing automation, new products like xConnect and Cortex and the new installation framework. With the new release, new assemblies and versions will be shipped, while others are removed. It's important to update your references to these new versions in your projects, so you won't overwite the shipped assemblies with other versions. I made a quick overview on what has changed:

- Which assemblies have been removed?
- Which assemblies have been added?
- Which assemblies had a version change?
- What are the most important observations?

Using Microsoft Excel and its Powerquery I compared the [Sitecore 8.2 update 5 assemblylist](https://dev.sitecore.net/~/media/C768547194504F2D892D22F71F6B8F93.ashx) with the [Sitecore 9](https://dev.sitecore.net/~/media/1F7FE7A2A06542009DEE620DAD62E374.ashx) version and created a list of all assemblies and checked wether or not they were added, removed, had a version change or nothing happened.

My most important observations:

- All Social Connected assemblies got removed. Of course I didn’t read the [release notes](https://dev.sitecore.net/Downloads/Sitecore%20Experience%20Platform/90/Sitecore%20Experience%20Platform%2090%20Initial%20Release/Release%20Notes), but it’s in there: it got removed until futher notice:
- HTMLAgility pack got downgraded from 1.4.6.0 to 1.4.9.5
- Newtonsoft got finally updated to version 9.0.1
- Microsoft Entlib got added

This list can be downloaded [here](http://blog.baslijten.com/wp-content/uploads/2017/10/SitecoreVersion.xlsx).
