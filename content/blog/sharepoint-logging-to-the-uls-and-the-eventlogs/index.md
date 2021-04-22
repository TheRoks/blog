---
title: "SharePoint Logging to the ULS and the eventlogs"
date: "2009-03-25"
---

Tis weekend I came across a function that makes it easy to log to the ULS, Diagnostic Logging. I even didnt know about the namespaces where the classes reside in.

The first one is a class that can log to the ULS, Diagnostic logging: Microsoft.Office.Server.Diagnostics

when using

PortalLog.LogString("Exception Occurred: {0} || {1}", ex.Message, ex.StackTrace);

it's possible to write tot the diagnostic logging. It contains several other functions like a DebugLogString, and

PortalLog.LaunchWatson(ex);

to launch Dr. Watson

When you want to write to the eventlog, use the following namespace and code: Microsoft.SharePoint.Portal.Diagnostics

SafeEventLog l = new SafeEventLog();

l.WriteEntry(ex, EventLogEntryType.Error);
