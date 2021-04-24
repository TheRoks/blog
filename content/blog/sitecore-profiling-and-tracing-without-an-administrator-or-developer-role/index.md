---
title: "Sitecore Profiling and tracing without an administrator or developer role"
date: "2017-08-21"
img: "./images/banner.png"
---

When working on Sitecore projects, there will popup some situations where you want to indicate performance issues. The out of the box capabilities are great, but the require a development role or and administrator account. While this might work in a lot of situations, there are situations where this just isn’t possible. For example, when having one ore more (virtual) extranet users which don’t have the Sitecore developer role and whose identities are needed to make backend calls. Performance issues might appear in those backend calls, but it may be hard to indicate where those performance sinks are located. That’s why I created a solution where the out of the box profiling and tracing options can be used, for _any_ user.

The solution is available on [github](https://github.com/BasLijten/Sitecore.AnonymousProfiler)

The original Sitecore solution contains some hardcoded logic which checks for the admin or a developer role. This is a role that we won’t assign to any extranet-user, ever, not even temporary. The code below is causing this inconvenience and I would recommend Sitecore to make this configurable in the near future 😉

<script src="https://gist.github.com/BasLijten/6b45c1f0426735645329c65eb280defc.js"></script>

That’s why we had to come up with another solution. In this specific solution, the default logic can be overridden by enabling a patch-file. After the patchfile has been overridden, the default querystring querystring parameters for enabling and disabling profiling and tracing can be used. This causes the out of the box functionality to continue to work, while we are able to have this kind of insight for any user and role.

<iframe src="https://www.youtube.com/embed/aHNEMVQUK_4" width="560" height="315" frameborder="0" allowfullscreen="allowfullscreen"></iframe>

I would recommend to disable this functionality file whenever possible and only enable it whenever this is needed: an new requirement would be to make this configurable through settings in Sitecore and get rid of the patch-files. This is on my (very long) to-do list 😉.
