---
layout: post
title:  "Accessing Tempo timesheets from JIRA plugin"
categories: main
tags: jira, tempo, report, rest, http
comments: true
---

How to access Tempo timesheets using `RequestFactory`.

{% highlight java %}
package com.example.jira.reports;

import java.text.SimpleDateFormat;
import java.util.Calendar;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;

import com.atlassian.jira.component.ComponentAccessor;
import com.atlassian.jira.config.properties.APKeys;
import com.atlassian.jira.plugin.report.impl.AbstractReport;
import com.atlassian.jira.web.action.ProjectActionSupport;
import com.atlassian.plugin.spring.scanner.annotation.component.Scanned;
import com.atlassian.plugin.spring.scanner.annotation.imports.ComponentImport;
import com.atlassian.sal.api.net.Request;
import com.atlassian.sal.api.net.RequestFactory;
import com.atlassian.sal.api.net.Response;
import com.atlassian.sal.api.net.ResponseException;
import com.atlassian.sal.api.net.ReturningResponseHandler;

@Scanned
public class MyReport extends AbstractReport {

  // TODO replace with your real Tempo API token
  private static final String TEMPO_API_TOKEN = "xxx-yyy-zzz";
  private String JIRA_BASE_URL;
  private RequestFactory requestFactory;

  public MyReport(@ComponentImport RequestFactory requestFactory) {
    JIRA_BASE_URL = ComponentAccessor.getApplicationProperties().getString(APKeys.JIRA_BASEURL);
    this.requestFactory = requestFactory;
  }

  private String getWorklogs(Date dateFrom, Date dateTo) {
    // to convert dates to the format required by Tempo
    SimpleDateFormat dateFormatter = new SimpleDateFormat("yyyy-MM-dd");
    // request parameters
    String params = "format=xml&diffOnly=false&tempoApiToken=%s&dateFrom=%s&dateTo=%s";
    params = String.format(params, TEMPO_API_TOKEN, dateFormatter.format(dateFrom), dateFormatter.format(dateTo));

    String url = JIRA_BASE_URL + "/plugins/servlet/tempo-getWorklog/?" + params;
    Request request = requestFactory.createRequest(Request.MethodType.GET, url);

    ReturningResponseHandler<Response, String> responseHandler = new ReturningResponseHandler<Response, String>() {
      @Override
      public String handle(Response response) throws ResponseException {
        return response.getResponseBodyAsString();
      }
    };

    String response;
    try {
      response = (String) request.executeAndReturn(responseHandler);
    } catch (ResponseException e) {
      response = e.getMessage();
    }

    return response;
  }

  @Override
  public String generateReportHtml(ProjectActionSupport arg0, Map arg1) throws Exception {
    Map<String, Object> velocityParams = new HashMap<String, Object>();
    
    Calendar dateFrom = Calendar.getInstance();
    // subtract a month from current date
    dateFrom.add(Calendar.MONTH, -1);
    // get worklog xml
    String worklogXml = getWorklogs(dateFrom.getTime(), Calendar.getInstance().getTime());
    velocityParams.put("worklogs", worklogXml);
    
    return descriptor.getHtml("view", velocityParams);
  }
}
{% endhighlight %}
