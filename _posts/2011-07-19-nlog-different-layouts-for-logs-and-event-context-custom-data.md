---
layout: post
title: Nlog different layouts for logs and event context custom data

tags: [.net, c#, debug, log, log4net, logger, nlog]
---

To add custom data to log, that can be used in layouts, log message like this:

```csharp
post_data["id"] = res.ToString();
LogEventInfo unpublish_post_data_event_info = new LogEventInfo(LogLevel.Trace, logger.Name, "Upublish data generated");
foreach (string k in post_data.Keys) unpublish_post_data_event_info.Properties[k] = post_data[k];
unpublish_post_data_event_info.Properties["VacancyID"] = item["VacancyID"];
unpublish_post_data_event_info.Properties["PostData"] = encode_post_data(post_data);
logger.Log(unpublish_post_data_event_info);
```

In this way you can access all log properties in layout via event-context variables:

```csharp
${event-context:item=job_category}
```

http://nlog-project.org/wiki/Event-context_layout_renderer

Now in targets config section:

```xml
<target name="unpublish" xsi:type="File" fileName="${basedir}/logs/unpublish.csv">
  <layout xsi:type="CSVLayout">
    <column name="name" layout="${event-context:item=job_title}" />
    <column name="rua_id" layout="${event-context:item=VacancyID}" />
    <column name="habr_id" layout="${event-context:item=id}" />
    <column name="post_data" layout="${event-context:item=PostData}"/>
  </layout>
</target>

<target xsi:type="ColoredConsole" name="console_default" layout="${message}${newline}"/>
<target xsi:type="ColoredConsole" name="console_vacancy_info" layout="________________________________________________________________________________${newline}PROCCESSING VACANCY${newline}http://rabota.ua/company0/vacancy${event-context:item=VacancyID}${newline}Name: ${event-context:item=VacancyName}${newline}RubricIds: ${event-context:item=RubricIDs}${newline}Regions: ${event-context:item=Cities}${newline}"/>
<target xsi:type="ColoredConsole" name="console_post_data_info" layout="POST DATA${newline}Category: ${event-context:item=job_category}${newline}GEO: ${event-context:item=job_country} / ${event-context:item=job_region} / ${event-context:item=job_city}${newline}"/>
```

Here we have few targets, now to all work propertly we need catch specific logs in loggers, this was accomplish wia condition filters.

http://nlog-project.org/wiki/Conditions

Rules section of nlog config:

```xml
<logger name="*" levels="Trace" writeTo="unpublish">
  <filters>
    <when condition="not equals('${message}','Upublish data generated')" action="Ignore" />
  </filters>
</logger>

<logger name="*" minlevel="Trace" maxlevel="Fatal" writeTo="console_default">
  <filters>
    <when condition="contains('${message}','Proccessing vacancy') and level==LogLevel.Trace" action="Ignore" />
    <when condition="contains('${message}','Post data for') and level==LogLevel.Trace" action="Ignore" />
  </filters>
</logger>
<logger name="*" levels="Trace" writeTo="console_vacancy_info">
  <filters>
    <when condition="not contains('${message}','Proccessing vacancy')" action="Ignore" />
  </filters>
</logger>
<logger name="*" levels="Trace" writeTo="console_post_data_info">
  <filters>
    <when condition="not contains('${message}','Post data for')" action="Ignore" />
  </filters>
</logger>
```

So console_default logger catches all messages except Trace that contains specific text. On other side console_vacacny_info and console_post_data_info catches only Trace messages with this specific text. All three of them outputs messages to console, but with different layout.
