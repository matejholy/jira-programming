---
layout: post
title:  "Creating Tempo 8 Worklog using a ScriptRunner script"
excerpt: "How to create Tempo 8 worklog in a ScriptRunner script. Tested in JIRA 7."
categories: main
tags: jira, tempo, worklog, scriptrunner
comments: true
redirect_from:
  - /main/2016/08/09/create-tempo-worklog.html
---

Adaptavist has [a nice example of how to create a worklog](https://scriptrunner.adaptavist.com/latest/jira/plugins/working-with-tempo.html). However, after upgrading to Tempo 8, the API changed and simply passing a map of attribute names and values to the `create()` function was no longer supported.

So how do we do it in Tempo 8 when we want to specify worklog attributes?

Tell ScriptRunner that we want to work with Tempo and get instance of the `TempoWorklogManager` using the `@WithPlugin` and `@PluginModule` annotations:
{% highlight groovy %}
import is.origo.jira.plugin.common.TempoWorklogManager

import com.onresolve.scriptrunner.runner.customisers.PluginModule
import com.onresolve.scriptrunner.runner.customisers.WithPlugin

@WithPlugin("is.origo.jira.tempo-plugin")

@PluginModule
TempoWorklogManager tempoWorklogManager
{% endhighlight %}

Then we get the `WorkAttribute` by calling the `WorkAttributeService`. In this case, we'll be setting a Tempo Account attribute: 
{% highlight groovy %}
WorkAttribute attribute
    = workAttributeService.getWorkAttributeByKey("_Account_").getReturnedValue();
{% endhighlight %}

After that, we get an instance of the `WorkAttributeValue` and set its value using a `WorkAttributeValue.Builder` class:
{% highlight groovy %}
// use account key, not name
WorkAttributeValue account = new WorkAttributeValue(new Builder(0, attribute, "AD"));
{% endhighlight %}

Finally, we'll put the account attribute in an `ArrayList` and call `tempoWorklogManager.create()`. Here's the complete script:

{% highlight groovy %}
import is.origo.jira.plugin.common.TempoWorklogManager

import org.apache.log4j.Level
import org.apache.log4j.Logger
import org.joda.time.LocalDateTime

import com.atlassian.jira.bc.ServiceOutcome
import com.atlassian.jira.component.ComponentAccessor
import com.atlassian.jira.user.ApplicationUser
import com.atlassian.jira.util.ErrorCollection
import com.onresolve.scriptrunner.runner.customisers.PluginModule
import com.onresolve.scriptrunner.runner.customisers.WithPlugin
import com.tempoplugin.core.datetime.api.TempoDateTime
import com.tempoplugin.core.workattribute.api.WorkAttribute
import com.tempoplugin.core.workattribute.api.WorkAttributeService
import com.tempoplugin.core.workattribute.api.WorkAttributeValue
import com.tempoplugin.core.workattribute.api.WorkAttributeValueService
import com.tempoplugin.core.workattribute.api.WorkAttributeValue.Builder


@WithPlugin("is.origo.jira.tempo-plugin")

@PluginModule
TempoWorklogManager tempoWorklogManager
@PluginModule
WorkAttributeService workAttributeService
@PluginModule
WorkAttributeValueService workAttributeValueService

Logger log = Logger.getLogger("com.example.jira.script.CreateWorklog");
log.setLevel(Level.INFO);

final String targetIssueKey = "ISSUE-1";
final ApplicationUser worklogAuthor = ComponentAccessor.getJiraAuthenticationContext().getLoggedInUser();
final long logTimeInSeconds = 60 * 60; // one hour 
final TempoDateTime worklogDate = TempoDateTime.ofLocalDateTime(new LocalDateTime());  // current date

// create attribute object. In this case, we'll be setting the value of the Account attribute.
// (error checking of getWorkAttributeByKey() omitted for simplicity)
WorkAttribute attribute = workAttributeService.getWorkAttributeByKey("_Account_").getReturnedValue();
// create WorkAttributeValue and set its value
WorkAttributeValue account = new WorkAttributeValue(new Builder(0, attribute, "AD")); // use account key, not name
// create attribute list
ArrayList attributes = new ArrayList();
attributes.add(account);

ServiceOutcome<Long> idOutcome 
    = tempoWorklogManager.create(targetIssueKey,
                                 worklogDate,
                                 "Worklog description",
                                 attributes,
                                 logTimeInSeconds,
                                 0l,  // billedSeconds - not using them
                                 0l,  // remainingEstimate - not using either
                                 worklogAuthor,
                                 new HashMap() // analyticsOrigin - no idea what that is
                                 );

// result and error processing
if (idOutcome.isValid())
  // log the id of the created worklog (not necessary, just an example of accessing the return value)
  log.info("created worklog with id " + idOutcome.getReturnedValue());
else {
  // log general and field errors
  ErrorCollection ec = idOutcome.getErrorCollection();
  for (String message : ec.getErrorMessages())
    log.error("error: " + message);
  Map<String, String> em = ec.getErrors();
  for (String key : em.keySet())
    log.error("field error: " + key + "=" + em.get(key));
}
{% endhighlight %}

The `create()` function called in the script is deprecated, but works for now.

If you know of a better way of creating a worklog or have any feedback on the code, let me know. 
