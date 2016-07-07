---
layout: post
title: SqlDataReader Mapper
tags: [sqldatareader, mapper, xml]
---

SqlDataReader simple mapper extension with XML support

In my case I have following setup:

	USE master
	GO
	IF EXISTS(SELECT * FROM sys.databases WHERE name='DataMapperTests')
	BEGIN
		ALTER DATABASE DataMapperTests SET SINGLE_USER WITH ROLLBACK IMMEDIATE
		DROP DATABASE DataMapperTests
	END
	CREATE DATABASE DataMapperTests
	GO
	USE DataMapperTests
	GO

	PRINT 'Creating tables'
	GO

	CREATE TABLE Tag (
		Id INT IDENTITY(1, 1) PRIMARY KEY,
		Name NVARCHAR(50) NOT NULL
	)

	CREATE TABLE Post (
		Id INT IDENTITY(1, 1) PRIMARY KEY,
		Title NVARCHAR(50) NOT NULL,
		Published BIT NOT NULL DEFAULT 0,
		CreatedAt DATETIME2 NOT NULL DEFAULT CONVERT(DATE, GETDATE()),
		PublishedAt DATETIME2
	)

	CREATE TABLE PostTags (
		PostId INT NOT NULL,
		TagId INT NOT NULL,
		PRIMARY KEY (PostId, TagId),
		CONSTRAINT FK_Post FOREIGN KEY (PostId) REFERENCES Post(Id) ON DELETE CASCADE,
		CONSTRAINT FK_Tag FOREIGN KEY (TagId) REFERENCES Tag(Id) ON DELETE CASCADE
	)
	GO

	PRINT 'Seed tables'
	GO

	SET NOCOUNT ON

	SET IDENTITY_INSERT Tag ON
	INSERT INTO Tag (Id, Name) VALUES
		(1, 'Tag 1'),
		(2, 'Tag 2');
	SET IDENTITY_INSERT Tag OFF
	GO

	SET IDENTITY_INSERT Post ON
	INSERT INTO Post (Id, Title, Published, PublishedAt) VALUES
		(1, 'Post 1', 1, CONVERT(DATE, GETDATE())),
		(2, 'Post 2', 0, NULL),
		(3, 'Post 3', 1, CONVERT(DATE, GETDATE()));
	SET IDENTITY_INSERT Post OFF
	GO

	INSERT INTO PostTags (PostId, TagId) VALUES
		(1, 1),
		(2, 1), (2, 2);
	GO

	PRINT 'Create view'
	GO

	CREATE VIEW SampleView AS
	SELECT
		Id,
		Title,
		CreatedAt,
		Published,
		PublishedAt,

		ISNULL((SELECT DISTINCT
			LTRIM(RTRIM(T.Name))
			FROM PostTags AS PT
			JOIN Tag AS T ON PT.TagId = T.Id AND PT.PostId = P.Id
			FOR XML PATH ('string'), ROOT('ArrayOfString'), TYPE), '<ArrayOfString></ArrayOfString>') AS ListString,

		ISNULL((SELECT DISTINCT
			LTRIM(RTRIM(T.Id))
			FROM PostTags AS PT
			JOIN Tag AS T ON PT.TagId = T.Id AND PT.PostId = P.Id
			FOR XML PATH ('int'), ROOT('ArrayOfInt'), TYPE), '<ArrayOfInt></ArrayOfInt>') AS ListInt,

		ISNULL((SELECT DISTINCT
			T.Id AS Id,
			LTRIM(RTRIM(T.Name)) AS Name
			FROM PostTags AS PT
			JOIN Tag AS T ON PT.TagId = T.Id AND PT.PostId = P.Id
			FOR XML PATH ('Tag'), ROOT('ArrayOfTag'), TYPE), '<ArrayOfTag></ArrayOfTag>') AS ListTag

	FROM Post AS P
	GO

Sample view is returning following data:

| Name        | Type               |
| ----------- | ------------------ |
| Id          | int                |
| Title       | string             |
| CreatedAt   | DateTime           |
| Published   | bool               |
| PublishedAt | DateTime           |
| ListString  | List&lt;string&gt; |
| ListInt     | List&lt;int&gt;    |
| ListTag     | List&lt;Tag&gt;    |

Our models are:

	public class Tag
	{
		public int Id { get; set; }
		public string Name { get; set; }
	}

	public class Post
	{
		public int Id { get; set; }
		public string Title { get; set; }
		public bool Published { get; set; }
		public DateTime PublishedAt { get; set; }
		public DateTime CreatedAt { get; set; }

		public List<int> ListInt { get; set; }
		public List<string> ListString { get; set; }
		public List<Tag> ListTag { get; set; }
	}

Here is SqlDataReaderMapperExtension:

	public static class SqlDataReaderExtensions
	{
		private static readonly MemoryCache Cache = MemoryCache.Default;

		public static T Map<T>(this SqlDataReader reader) where T : new()
		{
			var properties = GetProperties(typeof(T));

			var item = new T();

			for (var i = 0; i < reader.FieldCount; i++)
			{
				if (reader.IsDBNull(i)) continue;

				var property = properties[NormalizeKey(reader.GetName(i))].FirstOrDefault();

				if (property == null) continue;

				if (reader.GetFieldType(i) == property.PropertyType)
					property.SetValue(item, reader[i]);
				else if (reader.GetProviderSpecificFieldType(i) == typeof(SqlXml))
					property.SetValue(item, new XmlSerializer(property.PropertyType).Deserialize(reader.GetXmlReader(i)));
			}

			return item;
		}

		private static ILookup<string, PropertyInfo> GetProperties(Type type)
		{
			var cacheKey = $"{nameof(SqlDataReaderExtensions)}.{type.FullName}";
			var result = Cache.Get(cacheKey) as ILookup<string, PropertyInfo>;

			if (result != null) return result;

			result = type.GetProperties(BindingFlags.Public | BindingFlags.Instance).Where(p => p.CanWrite).ToLookup(p => NormalizeKey(p.Name));
			Cache.Add(cacheKey, result, null);

			return result;
		}

		private static string NormalizeKey(string name)
		{
			return name.Replace("_", "").ToLower().Trim();
		}
	}

which will allow you to get objects like this:

	using (var command = new SqlCommand(Query, GetSqlConnection()))
	{
		command.Connection.Open();
		using (var reader = command.ExecuteReader())
		{
			if (reader.HasRows)
			{
				while (reader.Read())
				{
					var post = reader.Map<Post>();
				}
			}
		}
	}

and here is event shorten way with command extension

	public static class SqlCommandExtensions
	{
		public static IEnumerable<T> ExecuteReader<T>(this SqlCommand command) where T : new()
		{
			command.Connection.Open();
			using (var reader = command.ExecuteReader())
			{
				if (!reader.HasRows) yield break;

				while (reader.Read())
				{
					yield return reader.Map<T>();
				}
			}
			command.Connection.Close();
		}
	}

and now you can do something like this:

	using (var command = new SqlCommand(Query, GetSqlConnection()))
	{
		var posts = command.ExecuteReader<Post>();
	}

Extension were written to handle xml in first place and for demo purposes
