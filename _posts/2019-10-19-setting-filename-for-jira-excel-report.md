---
layout: post
title:  "Setting filename for JIRA Excel report"
description: "If you don't know about it yet, there's a great tutorial on writing JIRA 7 server plugins."
categories: main
tags: jira, server, report, excel, download, tutorial
comments: true
---

Normally, when you click the Excel View in your report, you get a file named `ConfigureReport!excelView.jspa`.
But what if you want a more user-friendly name that is recognized as an Excel file?

All you need to do is to set a proper response header, in this case `content-disposition`.
Just use the following code in your report's `generateReportExcel()` method:

{% highlight java %}
import java.net.URLEncoder;
import java.nio.charset.StandardCharsets;
import webwork.action.ActionContext;

// ...

public class TestReport extends AbstractReport {
  // get report name from it's definition, encode it as UTF-8 
  String encodedFilename = URLEncoder.encode(getDescriptor().getName(), StandardCharsets.UTF_8.name());
  // replace + character with an URL encoded space character and add an xls extension
  encodedFilename = encodedFilename.replaceAll("\\+", "%20") + ".xls";
  // create content-disposition header value
  String contentDisposition = "attachment;filename*=UTF-8''" + encodedFilename;
  // add the content-disposition HTTP header
  ActionContext.getResponse().addHeader("content-disposition", contentDisposition);
  
  // ...
}
{% endhighlight %}

We use it in an abstract parent of our reports and just call `super.generateReportExcel(...)` from child classes.

Btw. the `content-disposition` format used above might not work in all browsers. If you need to support older browsers, use the `filename` value twice, once with and once without the asterisk:

{% highlight java %}
  String contentDisposition = "attachment; filename=" + encodedFilename + "; filename*=UTF-8''" + encodedFilename;
{% endhighlight %}

The above way of setting the value has been posted in the article [Download non-english filenames](https://fastmail.blog/2011/06/24/download-non-english-filenames/) by Rob Mueller.