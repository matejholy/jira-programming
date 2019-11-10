---
layout: post
title:  "Processing issues from a JQL search"
description: "How to process issues resulting from a JIRA JQL search."
categories: main
tags: jira, server, issue, jql, search
comments: true
---

Let's look at how you can do something with issues that are a result of a JQL search.

*These instructions have been tested in JIRA 7.13.9.*

The code in this example is a slightly modified version of [an answer given by Henning Tietgens](https://community.atlassian.com/t5/Marketplace-Apps-Integrations/How-to-run-JQL-query-inside-Groovy-script/qaq-p/194691) in Atlassian Community forum.

The idea is to use `SearchService.search()` method to get the list of `Issue` objects.

{% highlight java %}
import org.apache.log4j.Level
import org.apache.log4j.Logger

import com.atlassian.jira.user.ApplicationUser
import com.atlassian.jira.bc.issue.search.SearchService
import com.atlassian.jira.component.ComponentAccessor
import com.atlassian.jira.issue.Issue
import com.atlassian.jira.issue.IssueManager
import com.atlassian.jira.issue.search.SearchResults
import com.atlassian.jira.web.bean.PagerFilter

// set up logging
def log = Logger.getLogger("com.example.jira.script.ProcessJQL");
log.setLevel(Level.INFO);

// define JQL query
String jqlSearch = "project = PROJ AND issueType = Task";

// required objects
SearchService searchService = ComponentAccessor.getComponent(SearchService.class);
ApplicationUser user = ComponentAccessor.getJiraAuthenticationContext().getLoggedInUser();
IssueManager issueManager = ComponentAccessor.getIssueManager();

// validate the query
SearchService.ParseResult parseResult = searchService.parseQuery(user, jqlSearch);
if (parseResult.isValid()) {
  // do the search
  SearchResults searchResult = searchService.search(user, parseResult.getQuery(), PagerFilter.getUnlimitedFilter());
  
  for (Issue issue : searchResult.getIssues()) {
    log.info("Found issue: " + issue.getKey());
  }
} else {
  log.error("Invalid JQL: " + jqlSearch);
{% endhighlight %}

In the code above we're just logging the issue keys. If you need to update the issues, 
use the `issueManager.getIssueObject()` method to get a `MutableIssue` implementation 
and then the `issueManager.updateIssue()` to save the issue to the database:

{% highlight java %}
import com.atlassian.jira.event.type.EventDispatchOption
import com.atlassian.jira.issue.MutableIssue

// ...
for (Issue issue : searchResult.getIssues()) {
  log.info("Found issue: " + issue.getKey());
  
  // to update issue convert it to MutableIssue
  MutableIssue mutableIssue = issueManager.getIssueObject(issue.getId());
  mutableIssue.setSummary(issue.getSummary() + " UPDATED");
  issueManager.updateIssue(user, mutableIssue, EventDispatchOption.ISSUE_UPDATED, false); // false = do not send e-mail notifications
}
// ...
{% endhighlight %}