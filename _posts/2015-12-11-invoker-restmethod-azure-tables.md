---
layout: post
title: Powershell Invoke-WebRequest against Azure Storage Tables
tags: [azure, powershell, storage, tables, authorization]
---

To perform rest requests to azure storage tables you must sign your request, here is sample piece of code demonstrating how can it be done in Powershell:

```powershell
$accountName = 'contoso'
$accountKey = '**************************************************************************************=='
$tableName = 'mytable'

$uri = "http://$accountName.table.core.windows.net/$tableName(PartitionKey='tasksSeattle',RowKey='1')"

$date = [datetime]::UtcNow.ToString('R', [System.Globalization.CultureInfo]::InvariantCulture)

$resource = [uri] $uri | select -ExpandProperty AbsolutePath # Path without query string

$stringToSign = $date + "`n/" + $accountName + $resource

$hasher = New-Object System.Security.Cryptography.HMACSHA256
$hasher.Key = [Convert]::FromBase64String($accountKey)
$signedSignature = [Convert]::ToBase64String($hasher.ComputeHash([Text.Encoding]::UTF8.GetBytes($stringToSign)))
$authorizationHeader = "SharedKeyLite " + $accountName + ":" + $signedSignature

$headers = @{
	Authorization = $authorizationHeader
	Date = $date
}

Write-Host 'Perform first request' -ForegroundColor Cyan
$response = Invoke-WebRequest -Uri $uri -Headers $headers
$response | select StatusCode, StatusDescription, @{n='ETag';e={ $_.Headers.ETag }}
$xml = [xml]$response.Content
$xml.entry.content.properties | fl
```

That code will return something like:

```
PartitionKey : tasksSeattle
RowKey       : 1
Timestamp    : Timestamp
Description  : Take out the trash.
DueDate      : DueDate
Location     : Home
```

Make sure to provide correct values, otherwise you will likely get *Server failed to authenticate the request. Make sure the value of Authorization header is formed correctly including the signa
ture.* error.


Azure tables and conditional headers
------------------------------------

It seems that Azure tables do not support conditional headers like `If-Modified-Since` like blobs do, but there is still one trick.

If you will make `Invoke-WebRequest` instead of `Invoke-RestMethod` you will also get response headers. And there will be **ETag** header - which is entity timestamp.

```powershell
$response = Invoke-WebRequest -Uri $uri -Headers $headers
$response | select StatusCode, StatusDescription, @{n='ETag';e={ $_.Headers.ETag }} | fl
$xml = [xml]$response.Content
$xml.entry.content.properties | fl
```

Will return:

```
StatusCode        : 200
StatusDescription : OK
ETag              : W/"datetime'2015-12-11T10%3A53%3A52.6115577Z'"

PartitionKey : tasksSeattle
RowKey       : 1
Timestamp    : Timestamp
Description  : Take out the trash.
DueDate      : DueDate
Location     : Home
```

And now you can perform requests like:

```powershell
$headers = @{
	Authorization = $authorizationHeader
	Date = $date
	'If-None-Match' = $response.Headers.ETag
}
Invoke-RestMethod -Uri $uri -Headers $headers
```

Which will give you desired **(304) Not Modified**.


Bootstraping example table from php
-----------------------------------

**composer.json**

```json
{
	"repositories": [
		{
			"type": "pear",
			"url": "http://pear.php.net"
		}
	],
	"require": {
		"pear-pear.php.net/mail_mime" : "*",
		"pear-pear.php.net/http_request2" : "*",
		"pear-pear.php.net/mail_mimedecode" : "*",
		"microsoft/windowsazure": "*"
	}
}
```

**sandbox.php**

```php
<?php
require_once 'vendor\autoload.php';
use WindowsAzure\Common\ServiceException;
use WindowsAzure\Common\ServicesBuilder;
use WindowsAzure\Table\Internal\ITable;
use WindowsAzure\Table\Models\EdmType;
use WindowsAzure\Table\Models\Entity;

$accountName = "contoso";
$accountKey = "**************************************************************************************==";
$connectionString = "DefaultEndpointsProtocol=https;AccountName=$accountName;AccountKey=$accountKey";

/** @var ITable $tableRestProxy */
$tableRestProxy = ServicesBuilder::getInstance()->createTableService($connectionString);

try {
	$createTableResult = $tableRestProxy->createTable("mytable");
} catch(ServiceException $ex) {
	if($ex->getCode() !== 409) throw $ex;
}

$entity = new Entity();
$entity->setPartitionKey("tasksSeattle");
$entity->setRowKey("1");
$entity->addProperty("Description", null, "Take out the trash.");
$entity->addProperty("DueDate", EdmType::DATETIME, new DateTime("2012-11-05T08:15:00-08:00"));
$entity->addProperty("Location", EdmType::STRING, "Home");

try {
	$tableRestProxy->insertEntity("mytable", $entity);
} catch(ServiceException $ex) {
	if($ex->getCode() !== 409) throw $ex;
}
```


Some links
----------

[ETag](https://en.wikipedia.org/wiki/HTTP_ETag)

[Specifying Conditional Headers for Blob Service Operations](https://msdn.microsoft.com/en-us/library/azure/dd179371.aspx)

[Authenticating against Azure Table Storage](http://blog.einbu.no/2009/08/authenticating-against-azure-table-storage/)

[Authentication for the Azure Storage Services](https://msdn.microsoft.com/en-us/library/azure/dd179428.aspx)

[How to use table storage from PHP](https://azure.microsoft.com/en-us/documentation/articles/storage-php-how-to-use-table-storage/)
