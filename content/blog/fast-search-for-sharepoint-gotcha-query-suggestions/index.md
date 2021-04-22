---
title: "Fast Search for SharePoint gotcha - query suggestions"
date: "2011-07-19"
categories: 
  - "fast"
  - "sharepoint"
  - "sharepoint-2010"
---

At the company i work for, we do have around 30(?!) different labels and we decided that all websites of those labels, should be hosted on one platform: SharePoint 2010. This consolidation to one platform should really decrease costs, because we now have one generic way of developing, one platform to take care of and we can reuse a lot of components.  One of the service applications that we wanted to reuse amongst all applications, is the Fast Query Service Application, as we have a Fast instance running.  Our labels are really enthousiast about the Fast Search for SharePoint capabilities due to fact that a lot of interesting features, like visual best bets, can be configured from within the UI. Other things, like new ranking models can easily be configured via powershell. There is just one index, with all public information in it: personal details or things like that, will not be indexed! By configuring scopes in a correct way, we can prevent information from label 1 to appear in the search results on label 2. If, for some reason, those results appeared by accident (wrongly configured scope), that wouldn't be too much of a problem, as that can be fixed real soon.

**The gotcha ;)**

Query suggestions is one of the features that can be manually configured by registering new resourcephrases from within powershell, but it is self-learing too, based on query terms and click-throughs. Sadly enough, this is service application specific. But what does this mean?

Now the following case: We have two labels "Centraal Beheer" and "Interpolis", who are offering the same, competitive products. Imagine that Interpolis demanded us to add a query suggestion for the word "Interpolis". That could easily be added by executing the following powershell:

\[code\] $queryapp = Get-SPEnterpriseSearchServiceApplication 'FastQuery' New-SPEnterpriseSearchLanguageResourcePhrase -Language en-us -Type QuerySuggestionAlwaysSuggest -SearchApplication $queryapp -Name "Interpolis Verzekeringen" \[/code\]

As this query suggestion is added to the service application itself, and not scoped to a custom scope, webapplication or site collection, every web application that consumes this Search Query Application, will suggest the word interpolis. For competitive labels, this will be a major issue! Features as user contexts (till a certain spot) and keywords can be scoped on site-collection scope, suggestions not :( Imagine that the suggestion "Interpolis Verzekeringen" pops up when searching for "Intake Insurance" on the competitive site of "Centraal Beheer"? I wouldn't be too happy if I was a business owner of "Centraal Beheer".

Luckily, this can be solved by adding a second search service application, but if that's the way to go: I don't know. A second search service application can be used, but requires 3 extra databases to be created:

Crawlstore\_DB Query\_DB PropertyStore\_DB For smaller organisations, this shouldn't be too much of a hassle, but in a large company, this can cost a lot of extra money, due to (but not limited to) SLA's, extra administring of databases, storage and backup. And maybe are two service applications easy to manage, but what about 30 service applications (one for each label?).I think that the same architecture that is used for keywords and usercontexts, should be used for the query suggestions. Suggestions would even be manageble on site collections, what would be an extra advantage for FAST.

PS: this problem arises for SharePoint enterprise search as well ;) PS2: Who not using a multi tenant environment? Fast doesn't support that and we don't need it ;)
