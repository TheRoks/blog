---
title: "SharePoint 2010 signout different behaviours based on the number of selected Authentication Types"
date: "2012-12-29"
categories: 
  - "authentication"
  - "sharepoint-2010"
---

While working on our [custom ADFS login](http://bloggingabout.net/blogs/bas/archive/2012/12/28/customizing-adfs-login-for-sharepoint-2010-how-we-did-it.aspx "Custom ADFS login for SharePoint 2010") component and deployed this version to our DTAP street, we saw different behaviours when signing out of a site, under different circumstances. Wen users tried to logout via the page "/\_layouts/signout.aspx" users sometimes where redirected back to the root of the site and in some cases users got the message "please close the browser to signout". As I was curious why this happened, I decided to check a few things out.

## In wat cases does what behaviour occur?

After comparing our webapplication settings, we found out that the only difference that we had, was that on our Acceptance environment we had just a single Trusted Identity Provider selected, while on our Test environment we had both windows authentication and a Trusted Identity Provider selected. As this was a nice start, we decided to disable/enable a few identity providers and enable/disable windows authentication. As we couldn't find a real pattern, I decided to fire up reflector and started to do some real digging ;).

[![](images/4403.signout-code.png)](http://bloggingabout.net/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/bas/4403.signout-code.png)

As seen in the red section, a few decisions are made. First: ClaimsBasedAuthentication needs to be set. This is always in our case, so that one was easy. The code that is executed afterwards, is more interesting: A decision is made, based on the number of authentication providers **_OR_** WindowsIntegratedAuthentication is not turned on. It's easier to see what happens when a table is used. At the bottom of the table the actual action is included: **S**tay or **R**edirect

<table><tbody><tr><td>windows authentication</td><td>0</td><td>0</td><td>0</td><td>1</td><td>1</td><td>1</td><td>1</td><td>1</td><td>1</td><td>1</td><td>1</td></tr><tr><td>Integrated</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>1</td><td>0</td><td>1</td><td>0</td><td>1</td></tr><tr><td>Trusted Identity Provider 1</td><td>1</td><td>0</td><td>1</td><td>0</td><td>0</td><td>1</td><td>1</td><td>0</td><td>0</td><td>1</td><td>1</td></tr><tr><td>Trusted Identity Provider 2</td><td>0</td><td>1</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>1</td><td>1</td><td>1</td></tr><tr><td>num</td><td>1</td><td>1</td><td>2</td><td>1</td><td>1</td><td>2</td><td>2</td><td>2</td><td>2</td><td>3</td><td>3</td></tr><tr><td>use windows integrated</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>1</td><td>0</td><td>1</td><td>0</td><td>1</td></tr><tr><td>num!=1 or !windows integrated</td><td>1</td><td>1</td><td>1</td><td>1</td><td>0</td><td>1</td><td>1</td><td>1</td><td>1</td><td>1</td><td>1</td></tr><tr><td>expected action</td><td>R</td><td>R</td><td>R</td><td>R</td><td>S</td><td>R</td><td>R</td><td>R</td><td>R</td><td>R</td><td>R</td></tr><tr><td>actual action: S(tay)/R(edirect)</td><td>S</td><td>S</td><td>R</td><td>R</td><td>S</td><td>R</td><td>R</td><td>R</td><td>R</td><td>R</td><td>R</td></tr></tbody></table>

This table learns us the following: based on the selected options, we are expecting different behaviour then the actual behaviour: In cases where windows authentication is disabled (and thus, no windows integrated authentication) AND there is just 1 identity provider selected, a redirect to the rootweb of our sitecollection is expected, but this is not the case: users stay on the same page and get forced to close the browser to logout, altough this isn't needed. Manual navigation to the website learns us that the user has been logged out.

## Why no redirection?

to find out why this didn't happen, I wrote a small bit of code to checkout the value of iisSettings.UseWindowsIntegratedAuthentication when windows authentication is _disabled:_ This value seems always to be true when windows authentication has been disabled! When filling out these values in the table, the behaviour can be explained:

<table><tbody><tr><td>windows authentication</td><td>0</td><td>0</td></tr><tr><td>Integrated</td><td>1</td><td>1</td></tr><tr><td>Trusted Identity Provider 1</td><td>1</td><td>1</td></tr><tr><td>Trusted Identity Provider 2</td><td>0</td><td>1</td></tr><tr><td>num</td><td>1</td><td>2</td></tr><tr><td>use windows integrated</td><td>0</td><td>1</td></tr><tr><td>num!=1 or !windows integrated</td><td>0</td><td>1</td></tr><tr><td>expected action</td><td>S</td><td>R</td></tr><tr><td>actual action: S(tay)/R(edirect)</td><td>S</td><td>R</td></tr></tbody></table>

## Summary

Signing out within SharePoint 2010 or SharePoint 2013 leads to unexpected/unexplainable behaviour, because it's unclear when users are redirected to the rootWeb of the sitecollection, of are forced to close their windows. This behaviour has to do with the fact that, the value of "IisSettings.UseWindowsIntegratedAuthentication" is set to true when Authentication type "windows authentication" is set to false.I have a gut feeling that this value must explicitly set to false, whenever there is no windows authentication. The combination of multiple identity providers vs a single authentication provider leads to the difference in redirecting the user to the rootweb vs forcing the user to close the browser. I do believe that this is a bug and this will be submitted to Microsoft in the upcoming year (2013).
