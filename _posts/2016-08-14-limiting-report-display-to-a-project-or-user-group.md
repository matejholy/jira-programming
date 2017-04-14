---
layout: post
title:  "Limiting report display to a project or user group"
excerpt: "How to display your JIRA report only in specific projects and to specific users. Tested in JIRA 7."
categories: main
tags: jira, report
comments: true
---

We needed to display JIRA report only in a certain project and for users in specific groups. The group part was easy, but getting current project's key was not.

I found several solutions on the internet, but none of them worked. In the end I used a combination of parsing HTTP request and the `UserProjectHistoryManager` class.

To hide or show a report, you simply return `true` or `false` in report's `showReport()` method.
{% highlight java %}
public abstract class MyReport extends AbstractReport {

  @Override
  public boolean showReport() {
    return true;
  }
}
{% endhighlight %}

To check whether current user belongs to a group I created a simple method:
{% highlight java %}
protected boolean isCurrentUserInGroup(String groupName) {
  ApplicationUser loggedInUser = ComponentAccessor.getJiraAuthenticationContext().getLoggedInUser();
  return ComponentAccessor.getGroupManager().isUserInGroup(loggedInUser, groupName);    
}
{% endhighlight %}

Now the project key. I got it using the `UserProjectHistoryManager.getProjectHistoryWithoutPermissionChecks()` method, but it doesn't immediately reflect changing the project from the main menu.
So I combined it with extracting the key directly from the HTTP request.

I did not find a way to get `UserProjectHistoryManager` using `ComponentAccessor`, so I used `@Scanned` annotation on the class and `@ComponentImport` in the report's constructor for it to be injected automatically. There was no need to import `UserProjectHistoryManager` in `atlassian-plugin.xml` on JIRA 7, but it seems it might be necessary in older versions.

This is the method for getting current project's key:
{% highlight java %}
protected String getCurrentProjectKey() {
  HttpServletRequest request = ServletActionContext.getRequest();
  if (request != null)
  {
    // extract project's key from request
    Pattern r = Pattern.compile("/projects/([A-Z]+)");
    Matcher m = r.matcher(request.getRequestURI());
    if (m.find())  // this works on the project reports page
      return m.group(1);
    else {  // this works everywhere else
      ApplicationUser loggedInUser = ComponentAccessor.getJiraAuthenticationContext().getLoggedInUser();
      // get current project from user's history (does not work when switching between recent project from the main menu)
      List<UserHistoryItem> historyList = projectHistoryManager.getProjectHistoryWithoutPermissionChecks(loggedInUser);
      if (historyList != null && historyList.size() > 0) {
        Project currentProject = ComponentAccessor.getProjectManager().getProjectObj(Long.parseLong(historyList.get(0).getEntityId()));
        if (currentProject != null)
          return currentProject.getKey();
      }
    }
  }
  
  return null;
}
{% endhighlight %}

To show the report only for project "PRJ" and the "managers" group, I called the above methods like this:
{% highlight java %}
@Override
public boolean showReport() {
  return "PRJ".equals(getCurrentProjectKey()) && isCurrentUserInGroup("managers");
}
{% endhighlight %}

This is the final class with all methods combined:

{% highlight java %}
package com.example.jira.reports;

import java.util.List;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

import javax.servlet.http.HttpServletRequest;

import webwork.action.ServletActionContext;

import com.atlassian.jira.component.ComponentAccessor;
import com.atlassian.jira.plugin.report.impl.AbstractReport;
import com.atlassian.jira.project.Project;
import com.atlassian.jira.user.ApplicationUser;
import com.atlassian.jira.user.UserHistoryItem;
import com.atlassian.jira.user.UserProjectHistoryManager;
import com.atlassian.plugin.spring.scanner.annotation.component.Scanned;
import com.atlassian.plugin.spring.scanner.annotation.imports.ComponentImport;

@Scanned
public abstract class MyReport extends AbstractReport {
  
  private UserProjectHistoryManager projectHistoryManager;
  
  public MyReport(@ComponentImport UserProjectHistoryManager projectHistoryManager) {
    this.projectHistoryManager = projectHistoryManager;
  }

  protected String getCurrentProjectKey() {
    HttpServletRequest request = ServletActionContext.getRequest();
    if (request != null)
    {
      // extract project's key from request
      Pattern r = Pattern.compile("/projects/([A-Z]+)");
      Matcher m = r.matcher(request.getRequestURI());
      if (m.find())  // this works on the project reports page
        return m.group(1);
      else {  // this works everywhere else
        ApplicationUser loggedInUser = ComponentAccessor.getJiraAuthenticationContext().getLoggedInUser();
        // get current project from user's history (does not work when switching between recent project from the main menu)
        List<UserHistoryItem> historyList = projectHistoryManager.getProjectHistoryWithoutPermissionChecks(loggedInUser);
        if (historyList != null && historyList.size() > 0) {
          Project currentProject = ComponentAccessor.getProjectManager().getProjectObj(Long.parseLong(historyList.get(0).getEntityId()));
          if (currentProject != null)
            return currentProject.getKey();
        }
      }
    }
    
    return null;
  }
  
  protected boolean isCurrentUserInGroup(String groupName) {
    ApplicationUser loggedInUser = ComponentAccessor.getJiraAuthenticationContext().getLoggedInUser();
    return ComponentAccessor.getGroupManager().isUserInGroup(loggedInUser, groupName);    
  }
  
  @Override
  public boolean showReport() {
    // show report only for project "PRJ" and "managers" group
    return "PRJ".equals(getCurrentProjectKey()) && isCurrentUserInGroup("managers");
  }
}
{% endhighlight %}
