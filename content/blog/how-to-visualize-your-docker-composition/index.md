---
title: "How to visualize your docker composition"
date: "2020-11-06"
categories: 
  - "sitecore"
img: "./images/image-11.png"
---

After my [previous blogpost](https://blog.baslijten.com/sitecore-10-on-docker-help-to-understand-the-composition-of-the-configuration/ "https://blog.baslijten.com/sitecore-10-on-docker-help-to-understand-the-composition-of-the-configuration/"), I received several questions how I visualized that Docker architecture. This (very short) blogpost will explain how to do this.

The software that I use is a docker-container that is called “[docker-compose-viz](https://github.com/pmsipilot/docker-compose-viz)”, which is (mainly) maintained by [Julien Bianchi](https://github.com/jubianchi).

In order to visualize your composition, do the following:

- Navigate  to your directory, containing your docker-compose.yml
- Switch to linux mode
- Run the following command:

`docker run --rm -it --name dcv -v ${PWD}:/input pmsipilot/docker-compose-viz render -m image docker-compose.yml --output-file=achmea.techday.png --force`

It pulls the latest version of the docker-compose-viz image and runs is againts your docker-compose.yml. with the --output-file parameter, any filename and image type can be set.

The following parameters are also of much interest:

```
--override=OVERRIDE: Tag of the override file to use
--no-volumes: Omit the volume mapping
--no-ports: omit the external exposed ports and their mappings
```

Happy visualizing!
