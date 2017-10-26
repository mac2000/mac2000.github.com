---
layout: post
title: MS SQL Word Stemming

tags: [contains, formsof, freetext, inflectional, mssql, sql, stemming]
---

Find all rows that have any form of some word (aka stemming):

```sql
SELECT TOP 5 Name FROM Vacancy WHERE CONTAINS(Name, 'FORMSOF(INFLECTIONAL,programmers)') AND Name NOT LIKE '%programmers%'
SELECT TOP 5 Name FROM Vacancy WHERE FREETEXT(Name, 'programmers') AND NOT CONTAINS(Name, 'programmers')

-- For russian language add LANGUAGE 1049
SELECT TOP 5 Name FROM Vacancy WHERE FREETEXT(Name, 'киева', LANGUAGE 1049) AND NOT CONTAINS(Name, 'киева')
```

Found at: http://msdn.microsoft.com/en-us/library/ms176078.aspx

Language codes: http://msdn.microsoft.com/en-us/library/ms190303.aspx
