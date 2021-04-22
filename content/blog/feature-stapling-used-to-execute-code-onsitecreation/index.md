---
title: "Feature stapling used to execute code onSiteCreation"
date: "2009-09-12"
---

Some time ago I wanted to enable some features upon a siteCollection creation, but I couldnt find an eventhandler which allowed me to execute those actions. You could, of course, add or alter the ONET.XML for a new Template, but if you need to run code/activate features on Site Creation for multiple definitions, this can be a hell of a job. One of the possiblities that comes in is calles FeatureStapling. This is a technique used to activate features to siteDefinitions and it works as follows:

It's a feature with a farm-wide scope, which is needed to be able to be activated when a site is created. This could be the content of the stapling.xml

    1 <Elements xmlns="http://schemas.microsoft.com/sharepoint/">

    2   <FeatureSiteTemplateAssociation Id="F6924D36-2FA8-4f0b-B16D-06B7250180FA" TemplateName="GLOBAL" />

    3   <FeatureSiteTemplateAssociation Id="F6924D36-2FA8-4f0b-B16D-06B7250180FA" TemplateName="STS#1" />

    4 </Elements>

as seen, it contains a FeatureSiteTemplateAssociation tag with and ID that references to a feature that needs to be activated for a site. In addition to that, a TemplateName is provided with the templateName of the SiteDefinition that the feature needs to be activated to. This way you can activate features on site creation. When code is put in the OnFeatureActicatedEvent of the feature that is referenced to, you can run code when the site is created!

NOTE: When a SiteDefinition has it's setting "AllowGlobalFeatureAssociations" set to FALSE, the featureStaple won't work. Check for example the STS#1 definition (Blank page)  
(location: %12HIVE%\\TEMPLATE\\1033\\XML\\WEBTEMP.XML)
