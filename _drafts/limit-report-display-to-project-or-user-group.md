---
layout: post
title:  "Limiting report display to a project or user group"
date:   2016-08-09 0:00:00
categories: main
tags: jira, report
comments: true
---

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
      if (m.find())
        return m.group(1);
      else {
        // get current project from user's history (does not work when switching between recent project from the main menu)
        List<UserHistoryItem> historyList = projectHistoryManager.getProjectHistoryWithoutPermissionChecks(ComponentAccessor.getJiraAuthenticationContext().getLoggedInUser());
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
    return "PRJ".equals(getCurrentProjectKey())
        && (isCurrentUserInGroup("managers") || isCurrentUserInGroup("jira-administrators"));
  }
}
{% endhighlight %}
