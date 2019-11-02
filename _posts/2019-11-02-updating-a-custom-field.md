---
layout: post
title:  "Updating a custom field with and without keeping history"
description: "How to update JIRA custom field with and without keeping issue history."
categories: main
tags: jira, server, keep, update, history, custom field
comments: true
---

There are at least two ways how to update a custom field that I know of. Depending on your needs, choose whether you want to store a change record on issue history or not.

*These instructions have been tested in JIRA 7.13.9.*

## Updating a custom field without keeping history

If you want to update custom field value but do not want the change to appear in issue history, use `updateValue` method on `CustomField` class.

Here's an example:
{% highlight java %}
import com.atlassian.jira.component.ComponentAccessor
import com.atlassian.jira.issue.ModifiedValue
import com.atlassian.jira.issue.MutableIssue
import com.atlassian.jira.issue.fields.CustomField
import com.atlassian.jira.issue.util.DefaultIssueChangeHolder

// custom field id
final long MY_CUSTOM_FIELD_ID = 10000;

// get issue to update
MutableIssue issue = ComponentAccessor.getIssueManager().getIssueByKeyIgnoreCase("PROJ-1");

// custom field reference
CustomField myCustomField = ComponentAccessor.getCustomFieldManager().getCustomFieldObject(MY_CUSTOM_FIELD_ID);

// construct the new value
ModifiedValue newValue = new ModifiedValue(issue.getCustomFieldValue(myCustomField), "new value");
// update the field to "new value"
myCustomField.updateValue(null, issue, newValue, new DefaultIssueChangeHolder());
{% endhighlight %}

## Updating custom field and keeping history

For the change to appear in the issue history, use `setCustomFieldValue` on `Issue` class to set the value and then `IssueManager`'s method `updateIssue` to persist the change. Use the last parameter of the method to specify whether you want e-mail notifications to be sent or not.

{% highlight java %}
import com.atlassian.jira.component.ComponentAccessor
import com.atlassian.jira.event.type.EventDispatchOption
import com.atlassian.jira.issue.CustomFieldManager
import com.atlassian.jira.issue.MutableIssue
import com.atlassian.jira.issue.fields.CustomField
import com.atlassian.jira.user.ApplicationUser

// custom field id
final long MY_CUSTOM_FIELD_ID = 10000;

// get issue to update
MutableIssue issue = ComponentAccessor.getIssueManager().getIssueByKeyIgnoreCase("PROJ-1");

// custom field reference
CustomField myCustomField = ComponentAccessor.getCustomFieldManager().getCustomFieldObject(MY_CUSTOM_FIELD_ID);

// get logged in user
ApplicationUser user = ComponentAccessor.getJiraAuthenticationContext().getLoggedInUser();

// update value
issue.setCustomFieldValue(myCustomField, "new value");
ComponentAccessor.getIssueManager().updateIssue(user, issue, EventDispatchOption.ISSUE_UPDATED, false); // false = do not send e-mail notifications
{% endhighlight %}

## Updating custom fields of different types

The examples above assumed the use of a simple text custom field. When dealing with other types, check out this great example right in the Adaptavist Library: [Update the Value of Custom Fields through the Script Console](https://library.adaptavist.com/entity/update-the-value-of-a-custom-field-in-jira).