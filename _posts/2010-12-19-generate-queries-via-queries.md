---
layout: post
title: generate queries via queries

tags: [.net, c#, mssql]
---

```sql
DECLARE @TableName VARCHAR(MAX);

SET @TableName = 'ChannelEvents';

DECLARE @TableIdentity VARCHAR(MAX);
SET @TableIdentity = (SELECT COLUMN_NAME FROM INFORMATION_SCHEMA.COLUMNS WHERE (COLUMNPROPERTY(OBJECT_ID(TABLE_NAME), COLUMN_NAME, 'IsRowGuidCol') = 1 OR COLUMNPROPERTY(OBJECT_ID(TABLE_NAME), COLUMN_NAME, 'IsIdentity') = 1) AND TABLE_NAME = 'ChannelEvents');

/* Col1,Col2,...,ColN */
DECLARE @tc1 VARCHAR(MAX);
SET @tc1 = (SELECT (SELECT CAST(COLUMN_NAME + ',' AS VARCHAR(MAX)) FROM INFORMATION_SCHEMA.COLUMNS WHERE TABLE_NAME = 'ChannelEvents' AND COLUMN_NAME != @TableIdentity FOR XML PATH ('')));
SET @tc1 = LEFT(@tc1, LEN(@tc1) - 1);

/* @Col1,@Col2,...,@ColN */
DECLARE @tc2 VARCHAR(MAX);
SET @tc2 = (SELECT (SELECT CAST('@' + COLUMN_NAME + ',' AS VARCHAR(MAX)) FROM INFORMATION_SCHEMA.COLUMNS WHERE TABLE_NAME = 'ChannelEvents' AND COLUMN_NAME != @TableIdentity FOR XML PATH ('')));
SET @tc2 = LEFT(@tc2, LEN(@tc2) - 1);

/* Col1=@Col1,Col2=@Col2,...,ColN=@ColN */
DECLARE @tc3 VARCHAR(MAX);
SET @tc3 = (SELECT (SELECT CAST(COLUMN_NAME+'=@' + COLUMN_NAME + ',' AS VARCHAR(MAX)) FROM INFORMATION_SCHEMA.COLUMNS WHERE TABLE_NAME = 'ChannelEvents' AND COLUMN_NAME != @TableIdentity FOR XML PATH ('')));
SET @tc3 = LEFT(@tc3, LEN(@tc3) - 1);

DECLARE @CreateQuery VARCHAR(MAX);
SET @CreateQuery = 'SET NOCOUNT ON; INSERT INTO '+@TableName+' ('+@tc1+') VALUES ('+@tc2+'); SELECT @@IDENTITY AS LAST_ID; SET NOCOUNT OFF;';

DECLARE @ReadAllQuery VARCHAR(MAX);
SET @ReadAllQuery = 'SELECT * FROM '+@TableName;

DECLARE @ReadOneQuery VARCHAR(MAX);
SET @ReadOneQuery = 'SELECT * FROM '+@TableName+' WHERE '+@TableIdentity+' = @'+@TableIdentity;

DECLARE @UpdateQuery VARCHAR(MAX);
SET @UpdateQuery = 'UPDATE '+@TableName+' SET '+@tc3+' WHERE '+@TableIdentity+' = @'+@TableIdentity;

DECLARE @DeleteQuery VARCHAR(MAX);
SET @DeleteQuery = 'DELETE FROM '+@TableName+' WHERE '+@TableIdentity+' = @'+@TableIdentity;

/*SELECT @TableIdentity AS TableIdentity;*/
/*SELECT @tc1;*/
/*SELECT @tc2;*/
/*SELECT @tc3;*/
/*SELECT @CreateQuery;*/
/*SELECT @ReadAllQuery;*/
/*SELECT @ReadOneQuery;*/
/*SELECT @UpdateQuery;*/
/*SELECT @DeleteQuery;*/
```
