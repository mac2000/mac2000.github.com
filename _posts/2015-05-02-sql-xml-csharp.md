---
layout: post
title: Dealing with XML in SQL Server and C#
tags: [sql, xml, csharp]
---

Did you know that you can retrieve hierarchical data from SQL?

There is huge amount ot tips how to get comma separated lists from SQL server, like this:

```sql
SELECT
P.PostId,
P.Title,
STUFF((
    SELECT DISTINCT ',' + C.Name
    FROM PostCategories AS PC
    INNER JOIN Categories AS C ON PC.CategoryId = C.CategoryId
    WHERE PC.PostId = P.PostId
    FOR XML PATH(''), TYPE).value('.', 'NVARCHAR(MAX)'), 1, 1, '') AS Categories
FROM Posts AS P
```

which will return something like this:

PostId | Title             | Categories
-----: | ----------------- | ----------
1      | Composer tutorial | IT
2      | Going agile       | IT,Management
3      | XSL your XML      | IT

**I am wondering why do I need convert nice XML from inner select into comma separated list string to explode it later in C# at all?!**

Sample database
---------------

We are going to build following database:

<amp-img src="http://yuml.me/e8e4027f.svg" alt="Database" width="320" height="477"></amp-img>

Here is sample SQL on which tests will be made:

```sql
-- Cleanup
IF OBJECT_ID('dbo.PostCategories', 'U') IS NOT NULL DROP TABLE dbo.PostCategories
GO

IF OBJECT_ID('dbo.Posts', 'U') IS NOT NULL DROP TABLE dbo.Posts
GO

IF OBJECT_ID('dbo.Authors', 'U') IS NOT NULL DROP TABLE dbo.Authors
GO

IF OBJECT_ID('dbo.Categories', 'U') IS NOT NULL DROP TABLE dbo.Categories
GO

-- Schema
CREATE TABLE dbo.Authors (
    AuthorId INT IDENTITY(1, 1) PRIMARY KEY,
    FullName NVARCHAR(100) NOT NULL,
    BirthDate DATE
)
GO

CREATE TABLE dbo.Posts (
    PostId INT IDENTITY(1, 1) PRIMARY KEY,
    Title NVARCHAR(100) NOT NULL,
    AuthorId INT NOT NULL,
    CONSTRAINT FK_Posts_Authors FOREIGN KEY (AuthorId) REFERENCES dbo.Authors (AuthorId) ON DELETE CASCADE ON UPDATE CASCADE
)
GO

CREATE TABLE dbo.Categories (
    CategoryId INT IDENTITY(1, 1) PRIMARY KEY,
    Name NVARCHAR(100) NOT NULL
)
GO

CREATE TABLE dbo.PostCategories (
    PostId INT NOT NULL,
    CategoryId INT NOT NULL,
    CONSTRAINT FK_PostCategory_Posts FOREIGN KEY (PostId) REFERENCES dbo.Posts (PostId) ON DELETE CASCADE ON UPDATE CASCADE,
    CONSTRAINT FK_PostCategory_Categories FOREIGN KEY (CategoryId) REFERENCES dbo.Categories (CategoryId) ON DELETE CASCADE ON UPDATE CASCADE
)
GO

--Data
SET IDENTITY_INSERT dbo.Authors ON
INSERT INTO Authors (AuthorId, FullName, BirthDate) VALUES
    (1, 'Alexandr Marchenko', '1985-03-11'),
    (2, 'Maria Marchenko', '1988-06-11')
SET IDENTITY_INSERT dbo.Authors OFF
GO

SET IDENTITY_INSERT dbo.Categories ON
INSERT INTO Categories (CategoryId, Name) VALUES (1, 'IT'), (2, 'Management')
SET IDENTITY_INSERT dbo.Categories OFF
GO

SET IDENTITY_INSERT dbo.Posts ON
INSERT INTO Posts (PostId, Title, AuthorId) VALUES
    (1, 'Composer tutorial', 1),
    (2, 'Going agile', 2),
    (3, 'XSL your XML', 1)
SET IDENTITY_INSERT dbo.Posts OFF
GO

INSERT INTO PostCategories (PostId, CategoryId) VALUES (1, 1), (2, 1), (2, 2), (3, 1)
GO
```

[yuml.me](http://yuml.me/) definition:

```
[Authors|AuthorId:int;FullName:string;BirthDate:date]
[Posts|PostId:int;Title:string;AuthorId:int]
[Categories|CategoryId:int;Name:string]
[PostCategories|PostId:int;CategoryId:int]
[Authors]1->*[Posts]
[Posts]1->*[PostCategories]
[Categories]1->*[PostCategories]
```

Map SQL query XML result to C# object
-------------------------------------

Suppose we have following class:

```csharp
public class Post
{
    public int PostId { get; set; }
    public string Title { get; set; }
    public int AuthorId { get; set; }
}
```

SQL Query: `SELECT P.* FROM Posts AS P FOR XML PATH('Post'), ROOT('ArrayOfPost')` will return XML:

```xml
<ArrayOfPost>
    <Post>
    <PostId>1</PostId>
    <Title>Composer tutorial</Title>
    <AuthorId>1</AuthorId>
    </Post>
    <Post>
    <PostId>2</PostId>
    <Title>Going agile</Title>
    <AuthorId>2</AuthorId>
    </Post>
    <Post>
    <PostId>3</PostId>
    <Title>XSL your XML</Title>
    <AuthorId>1</AuthorId>
    </Post>
</ArrayOfPost>
```

**Notice** how we named root node of xml as `ArrayOfPost` it is used by C# xml serializer while deserializing XML into objects to determine that we wish to get `List<Post>`

Here is how to deserialize that query into `List<Post>`:

```csharp
XmlSerializer xmlSerializer = new XmlSerializer(typeof(List<Post>));
using (var connection = new SqlConnection((new SqlConnectionStringBuilder { InitialCatalog = "Play", IntegratedSecurity = true }).ConnectionString))
{
    using (var command = new SqlCommand("SELECT P.* FROM Posts AS P FOR XML PATH('Post'), ROOT('ArrayOfPost')", connection))
    {
        connection.Open();
        using (var reader = command.ExecuteXmlReader())
        {
            foreach (var post in (List<Post>)xmlSerializer.Deserialize(reader))
            {
                Console.WriteLine("Id: {0:N0}, Title: {1}", post.PostId, post.Title);
            }
        }
    }
}
```

Joining objects together
------------------------

Lets modify our object to be like this:

```csharp
public class Post
{
    public int PostId { get; set; }
    public string Title { get; set; }
    public Author Author { get; set; }
}
```

Id did replace `public int AuthorId { get; set; }` with `public Author Author { get; set; }`

Now we are going to ask our database for more information with this query:

```sql
SELECT
P.PostId,
P.Title,
(SELECT * FROM Authors AS A WHERE A.AuthorId = P.AuthorId FOR XML PATH(''), TYPE) AS Author
FROM Posts AS P
FOR XML PATH('Post'), ROOT('ArrayOfPost')
```

which will return:

```xml
<ArrayOfPost>
    <Post>
    <PostId>1</PostId>
    <Title>Composer tutorial</Title>
    <Author>
        <AuthorId>1</AuthorId>
        <FullName>Alexandr Marchenko</FullName>
        <BirthDate>1985-03-11</BirthDate>
    </Author>
    </Post>
    <Post>
    <PostId>2</PostId>
    <Title>Going agile</Title>
    <Author>
        <AuthorId>2</AuthorId>
        <FullName>Maria Marchenko</FullName>
        <BirthDate>1988-06-11</BirthDate>
    </Author>
    </Post>
    <Post>
    <PostId>3</PostId>
    <Title>XSL your XML</Title>
    <Author>
        <AuthorId>1</AuthorId>
        <FullName>Alexandr Marchenko</FullName>
        <BirthDate>1985-03-11</BirthDate>
    </Author>
    </Post>
</ArrayOfPost>
```

And mapping from previous example should still work (just make sure to chage query):

```csharp
XmlSerializer xmlSerializer = new XmlSerializer(typeof(List<Post>));
using (var connection = new SqlConnection((new SqlConnectionStringBuilder { InitialCatalog = "Play", IntegratedSecurity = true }).ConnectionString))
{
    using (var command = new SqlCommand("SELECT P.PostId, P.Title, (SELECT * FROM Authors AS A WHERE A.AuthorId = P.AuthorId FOR XML PATH(''), TYPE) AS Author FROM Posts AS P FOR XML PATH('Post'), ROOT('ArrayOfPost')", connection))
    {
        connection.Open();
        using (var reader = command.ExecuteXmlReader())
        {
            foreach (var post in (List<Post>)xmlSerializer.Deserialize(reader))
            {
                Console.WriteLine("Id: {0:N0}, Title: {1}, Author: {2} ({3:yyyy\\-MM\\-dd})", post.PostId, post.Title, post.Author.FullName, post.Author.BirthDate);
            }
        }
    }
}
```

will output:

```
Id: 1, Title: Composer tutorial, Author: Alexandr Marchenko (1985-03-11)
Id: 2, Title: Going agile, Author: Maria Marchenko (1988-06-11)
Id: 3, Title: XSL your XML, Author: Alexandr Marchenko (1985-03-11)
```

**IT IS AWESOME!**

Joining multiple objects
------------------------

Lets modify our class again:

```csharp
public class Post
{
    public int PostId { get; set; }
    public string Title { get; set; }
    public Author Author { get; set; }
    public string[] Categories { get; set; }
}
```

Now I have added `public string[] Categories { get; set; }` notice that it is just array of strings, in following example we will join categories itself to post object.

For C# xml serializer to be able to deserialize strings array xml should look something like: `<acme><string>foo</string><string>bar</string></acme>`.

So here is modified SQL:

```sql
SELECT
P.*,
(SELECT * FROM Authors AS A WHERE A.AuthorId = P.AuthorId FOR XML PATH(''), TYPE) AS Author,
(
    SELECT C.Name AS string
    FROM PostCategories AS PC
    INNER JOIN Categories AS C ON PC.CategoryId = C.CategoryId
    WHERE PC.PostId = P.PostId
    FOR XML PATH(''), TYPE
) AS Categories
FROM Posts AS P FOR XML PATH('Post'), ROOT('ArrayOfPost')
```

And its ouput:

```xml
<ArrayOfPost>
    <Post>
    <PostId>1</PostId>
    <Title>Composer tutorial</Title>
    <AuthorId>1</AuthorId>
    <Author>
        <AuthorId>1</AuthorId>
        <FullName>Alexandr Marchenko</FullName>
        <BirthDate>1985-03-11</BirthDate>
    </Author>
    <Categories>
        <string>IT</string>
    </Categories>
    </Post>
    <Post>
    <PostId>2</PostId>
    <Title>Going agile</Title>
    <AuthorId>2</AuthorId>
    <Author>
        <AuthorId>2</AuthorId>
        <FullName>Maria Marchenko</FullName>
        <BirthDate>1988-06-11</BirthDate>
    </Author>
    <Categories>
        <string>IT</string>
        <string>Management</string>
    </Categories>
    </Post>
    <Post>
    <PostId>3</PostId>
    <Title>XSL your XML</Title>
    <AuthorId>1</AuthorId>
    <Author>
        <AuthorId>1</AuthorId>
        <FullName>Alexandr Marchenko</FullName>
        <BirthDate>1985-03-11</BirthDate>
    </Author>
    <Categories>
        <string>IT</string>
    </Categories>
    </Post>
</ArrayOfPost>
```

Our C# code will be able to deserialize this without changes again.

```csharp
XmlSerializer xmlSerializer = new XmlSerializer(typeof(List<Post>));
using (var connection = new SqlConnection((new SqlConnectionStringBuilder { InitialCatalog = "Play", IntegratedSecurity = true }).ConnectionString))
{
    using (var command = new SqlCommand("SELECT P.*, (SELECT * FROM Authors AS A WHERE A.AuthorId = P.AuthorId FOR XML PATH(''), TYPE) AS Author, (SELECT C.Name AS string FROM PostCategories AS PC INNER JOIN Categories AS C ON PC.CategoryId = C.CategoryId WHERE PC.PostId = P.PostId FOR XML PATH(''), TYPE) AS Categories FROM Posts AS P FOR XML PATH('Post'), ROOT('ArrayOfPost')", connection))
    {
        connection.Open();
        using (var reader = command.ExecuteXmlReader())
        {
            foreach (var post in (List<Post>)xmlSerializer.Deserialize(reader))
            {
                Console.WriteLine(
                    "[{0:N0}] {1} by {2} ({3:yyyy\\-MM\\-dd}) in {4}",
                    post.PostId,
                    post.Title,
                    post.Author.FullName,
                    post.Author.BirthDate,
                    string.Join(", ", post.Categories)
                );
            }
        }
    }
}
```

Will output:

```
[1] Composer tutorial by Alexandr Marchenko (1985-03-11) in IT
[2] Going agile by Maria Marchenko (1988-06-11) in IT, Management
[3] XSL your XML by Alexandr Marchenko (1985-03-11) in IT
```

Joining List of objects
-----------------------

Here is last example:

```csharp
public class Post
{
    public int PostId { get; set; }
    public string Title { get; set; }
    public Author Author { get; set; }
    public Category[] Categories { get; set; }
}
```

I did replace `public string[] Categories { get; set; }` with `public Category[] Categories { get; set; }` so now we wanna get list of categories rather than list of category names.

Here is SQL query:

```sql
SELECT
P.*,
(SELECT * FROM Authors AS A WHERE A.AuthorId = P.AuthorId FOR XML PATH(''), TYPE) AS Author,
(
    SELECT *
    FROM PostCategories AS PC
    INNER JOIN Categories AS C ON PC.CategoryId = C.CategoryId
    WHERE PC.PostId = P.PostId
    FOR XML PATH('Category'), TYPE
) AS Categories
FROM Posts AS P FOR XML PATH('Post'), ROOT('ArrayOfPost')
```

And its output:

```xml
<ArrayOfPost>
    <Post>
    <PostId>1</PostId>
    <Title>Composer tutorial</Title>
    <AuthorId>1</AuthorId>
    <Author>
        <AuthorId>1</AuthorId>
        <FullName>Alexandr Marchenko</FullName>
        <BirthDate>1985-03-11</BirthDate>
    </Author>
    <Categories>
        <Category>
        <PostId>1</PostId>
        <CategoryId>11</CategoryId>
        <Name>IT</Name>
        </Category>
    </Categories>
    </Post>
    <Post>
    <PostId>2</PostId>
    <Title>Going agile</Title>
    <AuthorId>2</AuthorId>
    <Author>
        <AuthorId>2</AuthorId>
        <FullName>Maria Marchenko</FullName>
        <BirthDate>1988-06-11</BirthDate>
    </Author>
    <Categories>
        <Category>
        <PostId>2</PostId>
        <CategoryId>11</CategoryId>
        <Name>IT</Name>
        </Category>
        <Category>
        <PostId>2</PostId>
        <CategoryId>22</CategoryId>
        <Name>Management</Name>
        </Category>
    </Categories>
    </Post>
    <Post>
    <PostId>3</PostId>
    <Title>XSL your XML</Title>
    <AuthorId>1</AuthorId>
    <Author>
        <AuthorId>1</AuthorId>
        <FullName>Alexandr Marchenko</FullName>
        <BirthDate>1985-03-11</BirthDate>
    </Author>
    <Categories>
        <Category>
        <PostId>3</PostId>
        <CategoryId>11</CategoryId>
        <Name>IT</Name>
        </Category>
    </Categories>
    </Post>
</ArrayOfPost>
```

**Notice** that we are named our node `Categories` and not `ArrayOfCategories` it will work like this.

C# code:

```csharp
XmlSerializer xmlSerializer = new XmlSerializer(typeof(List<Post>));
using (var connection = new SqlConnection((new SqlConnectionStringBuilder { InitialCatalog = "Play", IntegratedSecurity = true }).ConnectionString))
{
    using (var command = new SqlCommand("SELECT P.*, (SELECT * FROM Authors AS A WHERE A.AuthorId = P.AuthorId FOR XML PATH(''), TYPE) AS Author, (SELECT * FROM PostCategories AS PC INNER JOIN Categories AS C ON PC.CategoryId = C.CategoryId WHERE PC.PostId = P.PostId FOR XML PATH('Category'), TYPE ) AS Categories FROM Posts AS P FOR XML PATH('Post'), ROOT('ArrayOfPost')", connection))
    {
        connection.Open();
        using (var reader = command.ExecuteXmlReader())
        {
            foreach (var post in (List<Post>)xmlSerializer.Deserialize(reader))
            {
                Console.WriteLine(
                    "[{0:N0}] {1} by {2} ({3:yyyy\\-MM\\-dd}) in {4}",
                    post.PostId,
                    post.Title,
                    post.Author.FullName,
                    post.Author.BirthDate,
                    string.Join(", ", post.Categories.Select(c => c.Name))
                );
            }
        }
    }
}
```

output will be exactly the same as in previous example.

Workflow
--------

If you ever got into troubles all you need to do is to try serialize/deserialize objects by hand, look at produced XML and try to reproduce it in SQL query

**Serialize C# object to XML**

```csharp
XmlSerializer xmlSerializer = new XmlSerializer(typeof(Post));
StringBuilder stringBuilder = new StringBuilder();
using (var xmlWriter = XmlWriter.Create(stringBuilder))
{
    xmlSerializer.Serialize(xmlWriter, new Post { PostId = 1, Title = "Composer tutorial", AuthorId = 1 });
    Console.WriteLine(stringBuilder.ToString());
}
```

will output:

```xml
<?xml version="1.0" encoding="utf-16"?>
<Post xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema">
        <PostId>1</PostId>
        <Title>Composer tutorial</Title>
        <AuthorId>1</AuthorId>
</Post>
```

**Deserialize C# object from XML**

```csharp
using (TextReader reader = new StringReader("<?xml version=\"1.0\" encoding=\"utf-16\"?><Post><PostId>1</PostId><Title>Composer tutorial</Title><AuthorId>1</AuthorId></Post>"))
{
    var post = (Post)xmlSerializer.Deserialize(reader);
    Console.WriteLine("Id: {0:N0}, Title: {1}, Author: {2:N0}", post.PostId, post.Title, post.AuthorId);
}
```

Will output:

```
Id: 1, Title: Composer tutorial, Author: 1
```

**Notice** that we are ommited XML namespaces as they seems to be not required

one more addition:

Saving C# object to SQL via XML
-------------------------------

Stored procedure template:

```sql
CREATE PROCEDURE dbo.SavePost @post XML
AS
BEGIN
    MERGE dbo.Posts AS target
    USING (
        SELECT
            Post.value('(PostId/text())[1]','INT') AS PostId,
            Post.value('(Title/text())[1]','NVARCHAR(100)') AS Title,
            Post.value('(AuthorId/text())[1]','INT') AS AuthorId
        FROM
            @post.nodes('/Post')AS TEMPTABLE(Post)
    ) AS source (PostId, Title, AuthorId)
    ON (target.PostId = source.PostId)
    WHEN MATCHED THEN
        UPDATE SET Title = source.Title, AuthorId = source.AuthorId
    WHEN NOT MATCHED THEN
        INSERT (Title, AuthorId)
        VALUES (source.Title, source.AuthorId);
END
GO
```

C# code sample:

```csharp
var xmlSerializer = new XmlSerializer(typeof(Post));
var stringBuilder = new StringBuilder();
using (var xmlWriter = XmlWriter.Create(stringBuilder))
{
    xmlSerializer.Serialize(xmlWriter, new Post { PostId = 2, Title = "Stored Procedure XML", AuthorId = 1 });
}
Console.WriteLine(stringBuilder.ToString());

using (var connection = new SqlConnection((new SqlConnectionStringBuilder { InitialCatalog = "Play", IntegratedSecurity = true }).ConnectionString))
{
    using (var command = new SqlCommand("SavePost", connection) { CommandType = System.Data.CommandType.StoredProcedure })
    {
        command.Parameters.Add(new SqlParameter("@post", stringBuilder.ToString()));
        connection.Open();
        var numberOfAffectedRows = command.ExecuteNonQuery();
        Console.WriteLine("Number of affected rows: {0:N0}", numberOfAffectedRows);
    }
}
```

Example from: http://www.aspsnippets.com/Articles/Pass-XML-parameter-to-Stored-Procedure-in-C-and-VBNet.aspx

```sql
CREATE PROCEDURE [dbo].[InsertXML]
@xml XML
AS
BEGIN
        SET NOCOUNT ON;

        INSERT INTO CustomerDetails
        SELECT
        Customer.value('@Id','INT') AS Id, --ATTRIBUTE
        Customer.value('(Name/text())[1]','VARCHAR(100)') AS Name, --TAG
        Customer.value('(Country/text())[1]','VARCHAR(100)') AS Country --TAG
        FROM
        @xml.nodes('/Customers/Customer')AS TEMPTABLE(Customer)
END
```

demonstrates how to retrieve atributes from xml

**Notice** that you can return whole saved object from stored procedure and deserialize it in C#, also you can pass as complex object as you wish.

In this [article](https://www.simple-talk.com/blogs/2012/01/05/using-xml-to-pass-lists-as-parameters-in-sql-server/) there is sample of passing lists into stored procedure.

In our case it can be something like this:

```sql
DECLARE @post XML
SET @post = '<Post xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema">
<PostId>4</PostId>
<Title>1111Stored Procedure XML Parameter</Title>
<AuthorId>1</AuthorId>
<Categories>
    <string>IT</string>
    <string>Accounting</string>
</Categories>
</Post>'

SELECT Category.value('.', 'NVARCHAR(100)') AS Category
FROM @post.nodes('/Post/Categories/string/text()') AS TEMPTABLE(Category)
```

Which will return:

```
Category
--------
IT
Accounting
```

This can be used to update multiple entites in database in one call.

SQL validate XML with XSD
-------------------------

Found at http://blog.sqlauthority.com/2009/12/02/sql-server-validate-an-xml-document-in-tsql-using-xsd-by-jacob-sebastian/

[xsd example](http://mac-blog.org.ua/xsd-example/) describes what it is and provides sample of xsd

Here is SQL to drop create XML schema collection:

```sql
IF EXISTS (SELECT * FROM sys.xml_schema_collections WHERE name = 'ItemsSchema') DROP XML SCHEMA COLLECTION ItemsSchema
GO

CREATE XML SCHEMA COLLECTION ItemsSchema
    AS'<?xml version="1.0" encoding="utf-8"?>
<!-- describing schema for sample xml -->
<xs:schema attributeFormDefault="unqualified" elementFormDefault="qualified" xmlns:xs="http://www.w3.org/2001/XMLSchema">

    <xs:element name="items">
        <!-- root element will be <items> -->
        <xs:complexType>
            <!-- it will be complex (there are complex and simple) -->
            <xs:sequence>
                <!-- it will consists from sequence of other elements -->
                <xs:element name="item" minOccurs="0" maxOccurs="unbounded">
                    <!-- there will be zero or more <item> elements in sequence -->
                    <xs:complexType>
                        <!-- they all have complex type -->
                        <xs:sequence>
                            <!-- each of them will have next sequence of elements -->
                            <xs:element name="name" type="NonEmptyString" minOccurs="1"/>
                            <!-- required (minOccurs="1") non empty (look for NonEmptyString at bottom) name field -->
                            <xs:element name="photo" minOccurs="1">
                                <!-- required photo with given pattern to validate urls for images -->
                                <xs:simpleType>
                                    <!-- this is example how to use additional restrictions for elements -->
                                    <xs:restriction base="xs:anyURI">
                                        <xs:minLength value="1" />
                                        <xs:pattern value="http://.*(png|jpg|jpeg|gif)" />
                                    </xs:restriction>
                                </xs:simpleType>
                            </xs:element>
                            <xs:element name="tags" type="NonEmptyString" minOccurs="1"/>
                            <xs:element name="diameter" type="xs:positiveInteger" minOccurs="1"/>
                            <xs:element name="weight" type="xs:positiveInteger" minOccurs="1"/>
                            <xs:element name="price" type="positiveDecimal" minOccurs="1"/>
                            <xs:element name="size" minOccurs="0" default="">
                                <!-- example of enum field -->
                                <xs:simpleType>
                                    <xs:restriction base="xs:string">
                                        <xs:enumeration value=""/>
                                        <xs:enumeration value="Big"/>
                                        <xs:enumeration value="Small"/>
                                    </xs:restriction>
                                </xs:simpleType>
                            </xs:element>
                        </xs:sequence>
                    </xs:complexType>
                </xs:element>
            </xs:sequence>
        </xs:complexType>
    </xs:element>
    <xs:simpleType name="positiveDecimal">
        <xs:restriction base="xs:decimal">
            <xs:minExclusive value="0"/>
            <xs:fractionDigits value="2"/>
        </xs:restriction>
    </xs:simpleType>
    <xs:simpleType name="NonEmptyString">
        <!-- we can describe our types separately to reuse them later -->
        <xs:restriction base="xs:string">
            <xs:minLength value="1" />
            <xs:pattern value=".*[^\s].*" />
        </xs:restriction>
    </xs:simpleType>
</xs:schema>'
GO
```

Now if you will run:

```sql
DECLARE @xml XML(ItemsSchema)
SELECT @xml = '<?xml version="1.0" encoding="utf-8"?>
<items>
    <item>
        <name>Item 1</name>
        <photo>http://example.com/photo1.png</photo>
        <tags>Tag1, Tag2</tags>
        <diameter>32</diameter>
        <weight>540</weight>
        <price>60</price>
        <size>Big</size>
    </item>
    <item>
        <name>Item 1</name>
        <photo>http://example.com/photo2.png</photo>
        <tags>Tag1</tags>
        <diameter>23</diameter>
        <weight>340</weight>
        <price>50</price>
    </item>
</items>'
```

All will be ok but if you git empty name, or remove name at all or give it frong photo url SQL will warn you!

For example here I trying to paste wrong photo url:

```sql
DECLARE @xml2 XML(ItemsSchema)
SELECT @xml2 = '<?xml version="1.0" encoding="utf-8"?>
<items>
    <item>
        <name>Item 1</name>
        <photo>http://example.com/photo1</photo>
        <tags>Tag1, Tag2</tags>
        <diameter>32</diameter>
        <weight>540</weight>
        <price>60</price>
        <size>Big</size>
    </item>
</items>'
```

Here is error:

```
Msg 6926, Level 16, State 1, Line 100
XML Validation: Invalid simple type value: 'http://example.com/photo1'. Location: /*:items[1]/*:item[1]/*:photo[1]
```


XSLT your XML right inside SQL
------------------------------

Here is awesomeness

```sql
-- NOTICE that class is not wrapped in namespace
/*
using System.Data.SqlTypes;
using System.IO;
using System.Text;
using System.Xml;
using System.Xml.Xsl;

public class XSLTTransform
{
    public static SqlXml Transform(SqlXml inputDataXML, SqlXml inputTransformXML)
    {
        MemoryStream memoryXml = new MemoryStream();
        XslCompiledTransform xslt = new XslCompiledTransform();
        XmlReader output = null;

        xslt.Load(inputTransformXML.CreateReader());

        // Output the newly constructed XML
        XmlTextWriter outputWriter = new XmlTextWriter(memoryXml, Encoding.Default);
        xslt.Transform(inputDataXML.CreateReader(), null, outputWriter, null);
        memoryXml.Seek(0, SeekOrigin.Begin);
        output = new XmlTextReader(memoryXml);

        return new SqlXml(output);
    }
}
*/


-- STEP 1. Enable CLR
sp_configure 'clr enabled', 1
GO
RECONFIGURE
GO

-- STEP 2. reCREATE assembly and function
IF EXISTS (SELECT * FROM sys.objects WHERE name = 'ApplyXsltTransform') DROP FUNCTION ApplyXsltTransform
GO

IF EXISTS (SELECT * FROM sys.assemblies WHERE name = 'XSLTTransform') DROP ASSEMBLY XSLTTransform
GO

CREATE ASSEMBLY XSLTTransform FROM 'C:\Users\Alexandr\Documents\visual studio 2015\Projects\XSLTTransform\XSLTTransform\bin\Release\XSLTTransform.dll'
GO

CREATE FUNCTION ApplyXsltTransform(@inputXML xml, @inputTransform xml)
RETURNS XML
AS EXTERNAL NAME XSLTTransform.XSLTTransform.Transform
GO


-- STEP 3. Demo
DECLARE @xml XML = (
    SELECT TOP 10 Date, AVG(TimeTaken) AS Value
    FROM MAC.Play.dbo.IISLog
    GROUP BY Date
    ORDER BY Date
    FOR XML PATH('Item'), ROOT('ArrayOfItem')
)
SELECT @xml

--DECLARE @xslt XML = (
--  SELECT BulkColumn FROM OPENROWSET(Bulk 'C:\Users\Alexandr\Documents\visual studio 2015\Projects\XSLTTransform\XSLTTransform\Chart.xslt', SINGLE_BLOB) AS x
--)
DECLARE @xslt XML = '<?xml version="1.0" encoding="UTF-8"?>
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
    <xsl:output method="html" indent="yes"/>

    <xsl:template match="/">
        <html>
            <body>
                <xsl:apply-templates/>
            </body>
        </html>
    </xsl:template>
    <xsl:template match="ArrayOfItem">
        <table>
            <thead>
                <tr>
                    <th>Date</th>
                    <th>Value</th>
                </tr>
            </thead>
            <tbody>
                <xsl:for-each select="Item">
                    <tr>
                        <td>
                            <xsl:value-of select="Date"/>
                        </td>
                        <td>
                            <xsl:value-of select="Value"/>
                        </td>
                    </tr>
                </xsl:for-each>
            </tbody>
        </table>
    </xsl:template>
</xsl:stylesheet>';
SELECT @xslt

SELECT dbo.ApplyXsltTransform(@xml, @xslt)
```

And its output:

```html
<html>
    <body>
    <table>
        <thead>
        <tr><th>Date</th><th>Value</th></tr>
        </thead>
        <tbody>
        <tr><td>2015-06-11T03:00:00</td><td>14</td></tr>
        <tr><td>2015-06-11T03:00:02</td><td>182</td></tr>
        <tr><td>2015-06-11T03:00:03</td><td>149</td></tr>
        <tr><td>2015-06-11T03:00:04</td><td>43</td></tr>
        <tr><td>2015-06-11T03:00:05</td><td>12</td></tr>
        <tr><td>2015-06-11T03:00:07</td><td>181</td></tr>
        <tr><td>2015-06-11T03:00:08</td><td>144</td></tr>
        <tr><td>2015-06-11T03:00:09</td><td>141</td></tr>
        <tr><td>2015-06-11T03:00:10</td><td>524</td></tr>
        <tr><td>2015-06-11T03:00:11</td><td>2106</td></tr>
        </tbody>
    </table>
    </body>
</html>
```

For this to work you need to compile and connect assembly.