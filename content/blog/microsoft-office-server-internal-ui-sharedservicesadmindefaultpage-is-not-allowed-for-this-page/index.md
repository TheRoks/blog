---
title: "'Microsoft.Office.Server.Internal.UI.SharedServicesAdminDefaultPage' is not allowed for this page"
date: "2009-02-06"
categories: 
  - "sharepoint"
---

Yesterday,  I wanted to visit my SSP admin page, but for some reason, i got the following error:

_"The base type 'Microsoft.Office.Server.Internal.UI.SharedServicesAdminDefaultPage' is not allowed for this page"_

I had, and still have, no clue what caused the problem, but I solved it by adding the following line to the web.config:

<SafeControl Assembly="Microsoft.Office.Server.UI, Version=12.0.0.0, Culture=neutral, PublicKeyToken=71e9bce111e9429c" Namespace="Microsoft.Office.Server.WebControls" TypeName="\*" />
