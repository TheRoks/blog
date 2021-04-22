---
title: "How to query xConnect using the tracker ContactID in Sitecore"
date: "2018-11-11"
categories: 
  - "code"
  - "sitecore"
  - "xconnect"
---

There are situations where not all custom facets have been loaded into your session, or where you want to explicitly check for updates on a custom facet in xConnect, for example when external systems might have been making changes to this facet. This blogpost explains how to use the trackerContactID to query xconnect, which can be used to get these custom facets.

In anonymous situations, the _only_ identifier that is available, is the current contact id:

\[code language="csharp"\] var xdbIdentifier = Tracker.Current.Contact.ContactId; \[/code\]

Although it might be attached to other identities, it’s the only identifier that might be available. This ID can be used to setup a new client connection to xConnect and do a lookup for the contact, including one or more custom facets.

The IdentifiedContactReference can be used for the query, it needs to be constructed with a source (which can be any source of identification, for example twitter, facebook or Azure Active Directory) and an ID. In this case, the source is the xDB tracker and the ID is the contactID that is provided by the tracer. The source that needs to be used is “xdb.tracker”. However, the ID that the tracker provides, is a guid with dashes, while the xConnect API expects a guid without dashes. The following extension method can be used to convert this ID:

\[code language="csharp"\] public static string ToXConnectIdentifier(this Guid contactId) { return contactId.ToString("N"); } \[/code\]

Now, the IdentifiedContactReference can be constructed and be used in your (custom) queries:

\[code language="csharp"\] var id = new IdentifiedContactReference("xDB.Tracker", xdbIdentifier.ToXConnectIdentifier()); \[/code\]

Happy querying!
