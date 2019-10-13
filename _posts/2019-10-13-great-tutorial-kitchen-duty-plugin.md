---
layout: post
title:  "A great JIRA server plugin tutorial - Kitchen Duty Plugin"
description: "If you don't know about it yet, there's a great tutorial on writing JIRA 7 server plugins."
categories: main
tags: jira, server, plugin, tutorial
comments: true
---

I've recently discovered a great tutorial for writing JIRA plugins - [Kitchen Duty Plugin](https://comsysto.github.io/kitchen-duty-plugin-for-atlassian-jira/) by a company called [Comsysto Reply](https://comsystoreply.de/). It focuses on JIRA 7.

The tutorial covers all the important concepts of writing JIRA server plugins and assumes only JavaEE and Maven knowledge.

What you'll learn:
* setting up Atlassian SDK, Maven project and the IDE (they're using IntelliJ IDEA)
* introduction to OSGi modules
* creating admin page for the plugin
* internationalization
* REST resources
* unit tests
* AUI, jQuery and Soy templates in front end
* persisting data to database using Active Objects

All the source code is available at [the tutorial's GitHub repository](https://github.com/comsysto/kitchen-duty-plugin-for-atlassian-jira).

I have been pleasantly surprised by discovering an [.editorconfig](https://editorconfig.org/) file concept thanks to this tutorial. The file allows to maintain consistent coding style for multiple developers using different IDE's.
