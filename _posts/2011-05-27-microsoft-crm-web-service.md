---
layout: post
title: Microsoft CRM web service

tags: [.net, asp.net, c#, crm, crmservice, networkcredential]
---

Интеграция с MS CRM по средствам встроенного сервиса дает возможность получать\изменять данные в системе.

**Полезные ссылки:**

http://example.com:5555/mscrmservices/2006/crmservice.asmx - сам сервис

http://example.com:5555/mscrmservices/2006/metadataservice.asmx - мета данные

http://example.com:5555/sdk/list.aspx - список сущностей

Для подключения к студии надо использовать вот такие ссылки:

http://example.com:5555/mscrmservices/2006/crmservice.asmx?WSDL&uniquename=COMPANYNAME

http://example.com:5555/mscrmservices/2006/metadataservice.asmx?WSDL&uniquename=COMPANYNAME

По ним будут доступны xml файлы описывающие сервис, со всеми кастомными полями и т.п. и именно они будут нужны для подключения к студии.

**Добавление сервиса к проекту.**

Для WinForms - правый клик по проекту **Add Web Reference**, в URL вбиваем путь к сохраненному xml файлу, в Web reference name вбиваем имя создаваемой ссылки.

![screenshot](/images/wp/111.png)

Для asp.net - правый клик по проекту **Add Service Reference**, далее клик по кнопке **Advanced...** и еще раз клик по кнопке **Add Web Reference..**. и дальше то же самое как и для обычных апликух.

http://msdn.microsoft.com/en-us/library/cc151015.aspx

http://msdn.microsoft.com/en-us/library/cc151014.aspx

Теперь после того как сервисы прикручены можно с ними работать.

**Пример получения информации и компании из CRM**

```csharp
MetadataService ms = new MetadataService();
CrmService s = new CrmService();
s.Credentials = new NetworkCredential("LOGIN", "PASSWORD", "COMPANYNAME");
s.Credentials = new NetworkCredential("LOGIN", "PASSWORD", "COMPANYNAME");

EntityMetadata em = ms.RetrieveEntityMetadata("organization", EntityFlags.All);
List<string> attrs = new List<string>();
foreach(AttributeMetadata am in em.Attributes) {
    attrs.Add(am.Name);
}
//MessageBox.Show(string.Join("\r\n", attrs.ToArray()));

ColumnSet colSet = new ColumnSet();
colSet.Attributes = new string[] { "accountid", "name", "new_notebookid", "createdon" };
QueryExpression query = new QueryExpression();
query.EntityName = EntityName.account.ToString();
query.ColumnSet = colSet;

//////
ConditionExpression condition = new ConditionExpression();
condition.AttributeName = "createdon";
condition.Operator = ConditionOperator.LastXDays;
condition.Values = new object[] { 1 };

// Build the filter based on the condition.
FilterExpression filter = new FilterExpression();
filter.FilterOperator = LogicalOperator.And;
filter.Conditions = new ConditionExpression[] { condition };

// Set the Criteria field.
query.Criteria = filter;
//////////////////////////

RetrieveMultipleRequest req = new RetrieveMultipleRequest();
req.Query = query;

RetrieveMultipleResponse response = (RetrieveMultipleResponse)s.Execute(req);

string res = "";
List<TMP> items = new List<TMP>();
foreach (BusinessEntity item in response.BusinessEntityCollection.BusinessEntities)
{
    account a = (account)item;

    items.Add(new TMP(a.accountid.Value.ToString(), a.new_notebookid, a.name));
}

dataGridView1.DataSource = items;
```

Собственно вот как это может выглядеть:

![screenshot](/images/wp/22.png)

Чтобы не заморачиваться с ColumnSet - можно юзать вот такую штуку:

```csharp
query.ColumnSet = new AllColumns();
```

Взято отсюда: http://community.dynamics.com/roletailored/customerservice/b/cscrmblog/archive/2009/01/07/building-rich-client-dashboards-for-microsoft-dynamics-crm-with-windows-presentation-foundation.aspx

Пример выуживания данных сразу из двух сущностей:

```csharp
String fetch = "<fetch mapping='logical'>" +
"<entity name='account'>" +
"<attribute name='new_notebookid'/>" +
"<attribute name='name'/>" +

"<link-entity name='systemuser' link-type='inner' from='systemuserid' to='ownerid'>" +
"<attribute name='lastname'/>" +
"</link-entity>" +

"<filter type='and'>" +
"<condition attribute = 'name' operator='like' value='%Саша%'/>" +
"</filter>" +

"</entity>" +
"</fetch>";
String result = s.Fetch(fetch);
```

Взято из этих мест:

http://msdn.microsoft.com/en-us/library/ms914457.aspx

http://social.microsoft.com/Forums/en/crmdevelopment/thread/5ef585a5-58bc-46cd-a134-a46869d0684c

http://www.stunnware.com/crm2/topic.aspx?id=FindingData6

http://mymscrm3.blogspot.com/2008/01/fetchxml-query-examples.html

**Вытягивание связанных записей**

Задачи за последние N дней плюс кастомное поле из account

```csharp
string fetchXml = @"
<fetch mapping=""logical"" count=""5"">
    <entity name=""activitypointer"">
    <attribute name=""subject"" />
    <filter>
        <condition attribute=""createdon"" operator=""last-x-days"" value=""3"" />
        <condition attribute=""statuscode"" operator=""eq"" value=""1"" />
    </filter>
    <link-entity name=""account"" from=""accountid"" to=""regardingobjectid"">
        <attribute name=""name"" />
        <attribute name=""new_notebookid"" />
    </link-entity>
    </entity>
</fetch>
";
```

выглядит вот так:

![screenshot](/images/wp/23.png)

**fetch link-entity aggregate**

Вытягиваем количество действий для конкретного аккаунта:

```csharp
string fetchXml = @"
<fetch mapping=""logical"" count=""50"" aggregate=""true"">
    <entity name=""activitypointer"">
    <attribute name=""activityid"" aggregate=""count"" alias=""activitypointer_count"" />
    <link-entity name=""account"" from=""accountid"" to=""regardingobjectid"">
        <attribute name=""new_notebookid"" />
    <filter>
        <condition attribute=""new_notebookid"" operator=""eq"" value=""99"" />
    </filter>
    </link-entity>
    </entity>
</fetch>
";
```

![screenshot](/images/wp/31.png)

**fetchxml into datagridview**

```csharp
string fetchXml = @"
<fetch mapping=""logical"" count=""50"" aggregate=""true"">
    <entity name=""activitypointer"">
    <attribute name=""activityid"" aggregate=""count"" alias=""activitypointer_count"" />
    <link-entity name=""account"" from=""accountid"" to=""regardingobjectid"">
        <attribute name=""new_notebookid"" />
    <filter>
        <condition attribute=""new_notebookid"" operator=""eq"" value=""99"" />
    </filter>
    </link-entity>
    </entity>
</fetch>
";

String result = s.Fetch(fetchXml);

StringReader lxmlStringReader = new StringReader(result);
DataSet ds = new DataSet();
ds.ReadXml(lxmlStringReader);

dataGridView1.DataSource = ds.Tables[1];
```

Avoid CRM 5000 limit
--------------------

By default CRM Web Services is limited to return only first 5000 records. Here is how you can retrieve all records:

```csharp
MetadataService ms = new MetadataService();
CrmService s = new CrmService();
s.Credentials = new NetworkCredential("USERNAME", "PASSWORD", "DOMAIN");
ms.Credentials = new NetworkCredential("USERNAME", "PASSWORD", "DOMAIN");

int i = 1;
bool finished = false;
DataSet dsAll = new DataSet();
while (finished == false)
{
    String fetch = @"<fetch mapping='logical' page='" + i + @"' count='5000'> <!-- add page and count attributes to avoid 5000 limit -->
    <!-- http://example.com:5555/sdk/list.aspx - list of available entities can be found here-->
    <entity name='account'>
        <!-- <all-attributes /> - to retrieve all attributes -->
        <attribute name='new_notebookid'/>
        <attribute name='name'/>
        <link-entity name='systemuser' link-type='inner' from='systemuserid' to='ownerid'>
            <attribute name='fullname'/>
            <attribute name='firstname'/>
            <attribute name='lastname'/>
        </link-entity>
        <!-- filtering example
        <filter type='and'>
            <condition attribute = 'name' operator='like' value='%Саша%'/>
        </filter>
        -->
    </entity>
    </fetch>";

    String result = s.Fetch(fetch);
    DataSet ds = new DataSet();
    StringReader reader = new StringReader(result);
    ds.ReadXml(reader);
    dsAll.Merge(ds);
    // if there is <resultset morerecods="1"> in response
    if (ds.Tables[0].Rows[0]["morerecords"].ToString() == "0") finished = true;
    else i++;
}

grid.DataSource = dsAll.Tables[1];
grid.DataBind();
```
