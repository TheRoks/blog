---
title: "JSS beginner issues: Placeholder 'xxx' was not found in the current rendering data"
date: "2019-03-12"
categories: 
  - "sitecore"
tags: 
  - "jss"
  - "sitecore"
img: "./images/2019-03-12_16-57-07.png"
---

Currently, I am researching JSS and I must say: it's great. So far, I ran into a few issues and although the documentation is great (I would recommend everyone to checkout the styleguide in the default app!), I am sure that people will run into the same issues as I did. I'll share short blogpost on these issues. Today number 1:

> 'Placeholder 'xxx' was not found in the current rendering data'
> 
>   

This issue occurs when component is inserted into a non-existent placeholder. This probably occurs under the following conditions:

a) a new component has been created with a new placeholder

```
<div class="row">    
    <div class="col-sm-8 col-lg-10">
        <sc-placeholder name="jss-main-carfunnel" [rendering]="rendering"></sc-placeholder>
    </div>
    <div class="col-sm-4 col-lg-2">
      <sc-placeholder name="jss-side" [rendering]="rendering"> </sc-placeholder>
  </div>
</div>
```

b) the route data was updated and imported into Sitecore

```
id: carfunnel-step-1
fields:
  pageTitle: Carfunnel - step 1
placeholders:
  jss-main:
  - componentName: CarfunnelLayout
    placeholders:
      jss-main-carfunnel:
      - componentName: CallMeBack
        fields: 
          heading: Call Me Back. Please! Now!
```

The most probable cause that causes this issue, is that your component definition was not updated:

```
export default function(manifest: Manifest) {
  manifest.addComponent({
    name: 'CarfunnelLayout',
    icon: SitecoreIcon.Layout
  });
}
```

Make sure to add the placeholders to the sitecore definition: this way, Sitecore knows a placeholders exists over there:

```
export default function(manifest: Manifest) {
  manifest.addComponent({
    name: 'CarfunnelLayout',
    icon: SitecoreIcon.Layout,
    placeholders: ['jss-main-carfunnel', 'jss-side']
  });
}
```

Never forget to update your definitions. When working with fields, the error message is quite meaningful, but when working with new placeholders, these might be forgotten!
