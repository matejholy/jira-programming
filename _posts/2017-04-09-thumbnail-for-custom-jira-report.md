---
layout: post
title:  "How to set a thumbnail for custom JIRA report"
description: "How to set your own thumbnail for custom JIRA report. Tested in JIRA 7."
categories: main
tags: jira, report, thumbnail
comments: true
---

How to set your own thumbnail for custom JIRA report? I'll show you in this post.

*These instructions have been tested in JIRA 7.2.2.*

__Create the thumbnail image__

Create the thumbnail as .png image. The _largest usable size is 268 x 140 px_. If the image is smaller, it will be displayed in the center of the thumbnail area. Save the image in the `*plugin_root*\src\main\resources\images` folder.

__Add some css__

Add the following code to your report's css file (the one specified in the `web-resource` section of the `atlassian-plugin.xml`). You can use any css class name you want, but the `:before` selector is important.
Optionally, add width and height properties that match the thumbnail size (e.g. `width: 268px; height: 140px;`) to speed up page display.

{% highlight css %}
body .test-report-thumbnail:before {
  background-image: url(images/test-report-thumbnail.png);
  background-repeat: no-repeat;
}
{% endhighlight %}

__Modify your atlassian-plugin.xml__

Add a new context `com.atlassian.jira.project.reports.page` to the `web-resource` in your `atlassian-plugin.xml` file:

{% highlight xml %}
  <web-resource key="test-report-resources" name="test-report Web Resources">
    <dependency>com.atlassian.auiplugin:ajs</dependency>
    <resource type="download" name="test-report.css" location="/css/test-report.css"/>
    <resource type="download" name="test-report.js" location="/js/test-report.js"/>
    <resource type="download" name="images/" location="/images"/>
    <context>test-report</context>
    <context>com.atlassian.jira.project.reports.page</context> <!-- added context -->
  </web-resource>
{% endhighlight %}

Staying in the `atlassian-plugin.xml`, specify the class you used in the css file using the `thumbnail` tag in the report section:

{% highlight xml %}
  <report name="Test Report" i18n-name-key="test-report.name" key="test-report" class="com.example.jira.testreport.TestReport">
    <description key="test-report.description">The Test Report Plugin</description>
    <resource name="view" type="velocity" location="/templates/reports/test-report/view.vm"/>
    <resource name="i18n" type="i18n" location="TestReport"/>
    <label key="test-report.label"></label>
    <thumbnail cssClass="test-report-thumbnail" /> <!-- added thumbnail css class -->
  </report>
{% endhighlight %}

That should do it, enjoy your own thumbnail.