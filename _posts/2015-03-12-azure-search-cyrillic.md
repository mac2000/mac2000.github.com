---
layout: post
title: Azure Search Cyrillic
tags: [azure, search, stemm, lucene, nlp]
---

Azure Search does not work pretty well at this moment with stemmers also there is no ngram etc. But there is still a way to make this work

Here is my experiment, suppose we have "Vacancies" index with some job offers in Russian, we want to search over them

All requests will be made with Powershell for simplicity

To reproduce them you gonna need your API key

Each vacancy in index has just one field "Position" against which search will be performed

**Note:** For powershell to produce right json do not forget to add something like `-Depth 10` to `ConvertTo-Json` calls

**Note:** For things to work propertly do not forget to convert your data into UTF-8 with `[System.Text.Encoding]::UTf8.GetBytes(...)`


**Setting up**

```powershell
$Headers = @{
	'Content-Type' = 'application/json; charset=utf-8'
	'api-key' = '********************************' # Provide Your API key
}

Invoke-RestMethod -Method Delete -Uri 'https://testrus.search.windows.net/indexes/vacancies?api-version=2015-02-28-Preview' -Headers $Headers
```


Search with default analyzers
-----------------------------

**Create index**

```powershell
$IndexDefinition = @{
	'name' = 'vacancies'
	'fields' = @(
		@{
			'name' = 'VacancyId'
			'type' = 'Edm.String'
			'searchable' = $False
			'filterable' = $False
			'sortable' = $False
			'facetable' = $False
			'key' = $True
			'retrievable' = $True
		},
		@{
			'name' = 'Position'
			'type' = 'Edm.String'
			'searchable' = $True
			'filterable' = $True
			'sortable' = $True
			'facetable' = $True
			'key' = $False
			'retrievable' = $True
		}
	)

}
Invoke-RestMethod -Method Post -Uri 'https://testrus.search.windows.net/indexes?api-version=2015-02-28-Preview' -Headers $Headers -Body ($IndexDefinition | ConvertTo-Json -Depth 10)
```


**Insert some data**

```powershell
$Documents = @{
	'value' = @(
		@{
			'VacancyId' = '1'
			'Position' = 'Менеджер по продажам в Киеве' # Translation: Sales manager in Kiev
		},
		@{
			'VacancyId' = '2'
			'Position' = '1-С Программист Киев' # Translation: 1-C programmer Kiev
		},
		@{
			'VacancyId' = '3'
			'Position' = '1-С Программист во Львове' # Translation: 1-C programmer Lviv
		},
		@{
			'VacancyId' = '4'
			'Position' = 'Acme ищет менеджера по продажам' # Translation: Acme search sales manager
		}
	)
}
Invoke-RestMethod -Method Post -Uri 'https://testrus.search.windows.net/indexes/vacancies/docs/index?api-version=2015-02-28-Preview' -Headers $Headers -Body ([System.Text.Encoding]::UTf8.GetBytes(($Documents | ConvertTo-Json -Depth 10)))
```

Notice that first two vacancies has `Киев` word at the end (It is capital city of Ukraine) and notice that first vacancy has additional letter `е` at the end

So, what I want is to perform search over `киеве` and get two first vacancies (search request and index should be stemmed)

```powershell
Invoke-RestMethod -Method Get -Uri 'https://testrus.search.windows.net/indexes/vacancies/docs?api-version=2015-02-28-Preview&search=киеве' -Headers $Headers | select -ExpandProperty value
```

```
@search.score VacancyId Position
------------- --------- --------
	0,74075186 1         Менеджер по продажам в Киеве
```

But there is only one :(


Azure search and Apache Lucene?
-------------------------------

Apache has made greate product called `Apache Lucene` which is used by many projects (eg: Apache Solr, ElasticSearch etc) it has freaking amount of features and analyzers

Azure Search provides us ability to use preconfigured Lucene analyzers

```powershell
$IndexDefinition = @{
	'name' = 'vacancies'
	'fields' = @(
		@{
			'name' = 'VacancyId'
			'type' = 'Edm.String'
			'searchable' = $False
			'filterable' = $False
			'sortable' = $False
			'facetable' = $False
			'key' = $True
			'retrievable' = $True
		},
		@{
			'name' = 'Position'
			'type' = 'Edm.String'
			'searchable' = $True
			'filterable' = $True
			'sortable' = $True
			'facetable' = $True
			'key' = $False
			'retrievable' = $True
			'analyzer' = 'ru.lucene' # <--- Here is tricky part
		}
	)

}
Invoke-RestMethod -Method Post -Uri 'https://testrus.search.windows.net/indexes?api-version=2015-02-28-Preview' -Headers $Headers -Body ($IndexDefinition | ConvertTo-Json -Depth 10)
```

Do not forget to delete index first, otherwise you will get error `Cannot create index 'vacancies' because it already exists.`

Now we can insert data absolutely like before and try run our search again

```powershell
Invoke-RestMethod -Method Get -Uri 'https://testrus.search.windows.net/indexes/vacancies/docs?api-version=2015-02-28-Preview&search=киеве' -Headers $Headers | select -ExpandProperty value | ft -AutoSize
```

```
@search.score VacancyId Position
------------- --------- --------
	0,8465736 1         Менеджер по продажам в Киеве
```


So here is funny stuff, from one side Azure giving us serious analyzer tool, but only with preconfigured options which is not working :)


Microsoft Natural Language Processing
-------------------------------------

Thank to gods Azure provide to us another probably even cooler way to analyze our data with their NLP (which is by the way used in Office and Bing)

```powershell
$IndexDefinition = @{
	'name' = 'vacancies'
	'fields' = @(
		@{
			'name' = 'VacancyId'
			'type' = 'Edm.String'
			'searchable' = $False
			'filterable' = $False
			'sortable' = $False
			'facetable' = $False
			'key' = $True
			'retrievable' = $True
		},
		@{
			'name' = 'Position'
			'type' = 'Edm.String'
			'searchable' = $True
			'filterable' = $True
			'sortable' = $True
			'facetable' = $True
			'key' = $False
			'retrievable' = $True
			'analyzer' = 'ru.microsoft' # <--- Microsoft NLP can stemm russian words
		}
	)

}
Invoke-RestMethod -Method Post -Uri 'https://testrus.search.windows.net/indexes?api-version=2015-02-28-Preview' -Headers $Headers -Body ($IndexDefinition | ConvertTo-Json -Depth 10)
```

As usual do not forget delete old index first, and now magic happens:

```powershell
Invoke-RestMethod -Method Get -Uri 'https://testrus.search.windows.net/indexes/vacancies/docs?api-version=2015-02-28-Preview&search=киеве' -Headers $Headers | select -ExpandProperty value | ft -AutoSize
```

```
@search.score VacancyId Position
------------- --------- --------
	0,30007723 1         Менеджер по продажам в Киеве
	0,30007723 2         1-С Программист Киев
```

At last, we got our two documents and all seems to work right

**Tip:** Do not forget that you can have in our index `PositionRaw`, `PositionLucene`, 'PositionMicrosoft` fields with different analyzers to perform queries against all analyzers. Unfortunatelly it is not so elegant like in ElasticSearch, but come on Azure Search released few days ago :)


Azure Search Suggesters and Analyzers
-------------------------------------

There is always something wrong when all seems to be good :)

```powershell
$IndexDefinition = @{
	'name' = 'vacancies'
	'fields' = @(
		@{
			'name' = 'VacancyId'
			'type' = 'Edm.String'
			'searchable' = $False
			'filterable' = $False
			'sortable' = $False
			'facetable' = $False
			'key' = $True
			'retrievable' = $True
		},
		@{
			'name' = 'Position'
			'type' = 'Edm.String'
			'searchable' = $True
			'filterable' = $True
			'sortable' = $True
			'facetable' = $True
			'key' = $False
			'retrievable' = $True
			'analyzer' = 'ru.microsoft'
		}
	)
	'suggesters' = @(
		@{
			'name' = 'sg'
			'searchMode' = 'analyzingInfixMatching'
			'sourceFields' = @('Position')
		}
	)

}
Invoke-RestMethod -Method Post -Uri 'https://testrus.search.windows.net/indexes?api-version=2015-02-28-Preview' -Headers $Headers -Body ($IndexDefinition | ConvertTo-Json -Depth 10)
```

If you will try create index with suggesters over fields that use **any** analyzer you will get `Field 'Position' in suggester 'sg' uses a custom analyzer, suggesters are not currently supported with custom analyzers.`

Hope to see this feature somewhere in the future
