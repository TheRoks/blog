---
title: "SharePoint 2007 to 2010 migration error when changing the UIVersion and masterpages"
date: "2012-03-27"
categories: 
  - "migration"
  - "sharepoint"
  - "sharepoint-2010"
---

I am with my team currently busy on a migration of our intranet from SharePoint 2007 to SharePoint 2010. Part of this migration, is updating the masterpages, so that we can benefit from every new SharePoint 2010 feature.

## The Problem

Recently, two of our teammembers got the following error message after updating the UIVersion to 4 and setting a specific SP2010 masterpage.

_One or more field types are not installed properly. Go to the list settings page to delete these fields.<nativehr>0x81020014</nativehr><nativestack>owssvr.dll: (unresolved symbol, module offset=000000000022BC69) at 0x000007FEE967BC69 owssvr.dll: (unresolved symbol, module offset=000000000022BE3E) at 0x000007FEE967BE3E owssvr.dll: (unresolved symbol, module offset=000000000022C59A) at 0x000007FEE967C59A owssvr.dll: (unresolved symbol, module offset=000000000022F73C) at 0x000007FEE967F73C owssvr.dll: (unresolved symbol, module offset=0000000000220B01) at 0x000007FEE9670B01 owssvr.dll: (unresolved symbol, module offset=000000000018A8D4) at 0x000007FEE95DA8D4 owssvr.dll: (unresolved symbol, module offset=0000000000196058) at 0x000007FEE95E6058 owssvr.dll: (unresolved symbol, module offset=000000000018E5CF) at 0x000007FEE95DE5CF owssvr.dll: (unresolved symbol, module offset=00000000001943AE) at 0x000007FEE95E43AE owssvr.dll: (unresolved symbol, module offset=000000000000D085) at 0x000007FEE945D085 owssvr.dll: (unresolved symbol, module offset=000000000002781D) at 0x000007FEE947781D mscorwks.dll: (unresolved symbol, module offset=00000000002CB717) at 0x000007FEF9D1B717 Microsoft.SharePoint.Library.ni.dll: (unresolved symbol, module offset=0000000000109079) at 0x000007FEED979079 Microsoft.SharePoint.ni.dll: (unresolved symbol, module offset=00000000019C5FE0) at 0x000007FEF0005FE0 Microsoft.SharePoint.ni.dll: (unresolved symbol, module offset=0000000001AEC6E1) at 0x000007FEF012C6E1 Microsoft.SharePoint.ni.dll: (unresolved symbol, module offset=0000000001AED92A) at 0x000007FEF012D92A </nativestack>_

the strange thing was, we all used the same content databases, same WSP's, same codebase, everything was the same. Searching the internet for this error, didn't bring me a real solution. [Some answers to forum posts](http://social.technet.microsoft.com/Forums/nl/sharepoint2010setup/thread/a30f0ffd-782f-4d4a-9c22-b92bac0fad4f) suggest to force disable the publishing infra structure feature, delete the relationships list and enable the feature again, but that didn't seem to be a very constructive solution  to me: I still didn't know _what_ caused the error, and as some other team members didn't get the error, I suspected that something else was happening.

## The solution

Due to the fact that not everyone in our team got the error when updating, I suspected it was an environmental issue. After running the following powershell command, I found out that everyone was on the same patchlevel:

PS C:\\Users\\ontbsale\\Documents> $farm = Get-SPFarm PS C:\\Users\\ontbsale\\Documents> $farm.BuildVersion

Major Minor Build Revision ----- ----- ----- -------- 14 0 6106 5002

When I checked the installed patches to SharePoint 2010, I found out that on our central development-environment and on my machine, SP1 _including_ the language packs were installed. A quick check on the development machines of my team members learned me that SP1 was rolled out, but that they didn't install the dutch language packs. Due to the fact that our (custom) site definitions were based on the dutch LocalID, it seemed to be that the combination of a dutch site and the lack of the current language specific service pack is a killing combination

## Summary

The experience above is (again) one of those experiences that teach us how important it is to upgrade your SharePoint 2010 environment exactly. Make sure to be on the correct patchlevel (the same as the one that is used on your DTAP-environment) and make sure that _every_ patch that is correlated with that build-version is installed. Not doing that (e.g., just a part of the SP1 packs) can cause problems that seem to be unexplainable. Luckily we solved this problem. If not, we would have a big issue in our migration process: who could ensure us that this wouldn't happen during the go-live weekend?!
