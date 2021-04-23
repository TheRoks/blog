---
title: "Warmup your application on Azure App service when scaling up and swapping slots using \"Application Initialization\""
date: "2018-10-29"
categories: 
  - "apps"
  - "azure"
  - "performance"
  - "sitecore"
coverImage: "warmup.jpg"
---

A common problem on Azure web apps when scaling up or swapping slots is "stuttering". At the moment an instance is added to the pool (scale out) or your swap is swapped (reload the app on the slot), your application is "cold , which means that your application on that instance needs to be reloaded. In the case of Sitecore (or other large applications), this may take a while. In this period, visitors may face a long loading time, which may take up to a few minutes.

This stuttering can easily be resolved for Azure App Services, making use of default mechanisms which isn't widely used or known. This mechanism is called "[Application Initialization](https://docs.microsoft.com/en-us/iis/get-started/whats-new-in-iis-8/iis-80-application-initialization)", which was introduced in IIS version 8:

_"Application Initialization can start the initialization process automatically whenever an application is started."_

This process doesn't make the startup process _faster_, but starts the process _sooner._  The fun part is: when making use of scaling within Azure Apps, this mechanism can be used to warmup the specific new azure app service instances.

Within the _web.WebServer_ section of the web.config, the Applicationinitialization node may be added. Within this node, the pages that need to be warmed can be specified over here:

```xml
<web.webServer> <applicationInitialization> <add initializationPage="/" /> <add initializationPage="/page-2" /> </applicationInitialization> </web.webServer> ```

As stated previously, this action starts the process sooner, but doesn't make it faster. This applies to normal IIS as well as Azure App Services. So after a restart, and you'd immediately try to visit that website, you would experience a long loading time. But the Azure App service has a cool feature. When scaling up (manually or by using autoscale) _or_ swapping slots, a new instance is created and started. The initializationPages are being touched internally and Azure App Services _waits_ with the actual swap or addition to the Azure App Service pool until all the pages which have been defined in that section, have been loaded. This will prevent that "cold" applications will not be released, which means that there will be no "stuttering" of the application.
