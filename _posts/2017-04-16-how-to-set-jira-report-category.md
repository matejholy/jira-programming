---
layout: post
title:  "How to set JIRA report category"
description: "How to set category for custom JIRA report. Tested in JIRA 7."
categories: main
tags: jira, report, category
comments: true
---

Reports in JIRA are grouped into four categories, when displayed on reports page. You can choose one of them, but at the time of writing this (JIRA 7.4), you cannot create your own category.

*These instructions have been tested in JIRA 7.2.2.*

__Setting category__

Report category is specified in the `atlassian-plugin.xml`, inside the report's tag, using the `category` tag with a `key` attribute.

For example:
{% highlight xml %}
  <report name="Test Report" i18n-name-key="test-report.name" key="test-report" class="com.example.jira.testreport.TestReport">
    <description key="test-report.description">The Test Report Plugin</description>
    <resource name="view" type="velocity" location="/templates/reports/test-report/view.vm"/>
    <resource name="i18n" type="i18n" location="TestReport"/>
    <label key="test-report.label"></label>
    <category key="issue.analysis" />  <!-- setting category -->
  </report>
{% endhighlight %}

__Available categories__

The possible values of the `key` attribute are:

Key value | Category
--- | ---
`agile` | Agile
`issue.analysis` | Issue analysis
`forecast.management` | Forecast & management
`other` | Other

If you don't specify a category key, the `other` is used as default.

If you use unsupported value, the plugin will fail to load with the following error being logged: _The report module: com.test.report-injection-test:my-report-x specified a category key that is not a valid category: x_.
I used category `key="x"` in this case.