---
layout: post
title: Automapper example
tags: [c#, automapper, map]
---

Everyday task - retrieve data from SQL and map it to object can be achived in nice and elegant way with [Automapper](http://automapper.org/)

Automapper usage example
------------------------

```csharp
using System;
using System.Globalization;
using System.Linq;
using AutoMapper;

namespace AutomapperExample
{
    // Source class retrieved from somewhere e.g. sql query
    class PersonDto
    {
        public string FullName { get; set; } // should be splitted into first name and last name
        public string Skills { get; set; } // comma separated list
        public string BirthDate { get; set; } // should be casted to date time, inputs will be something like: 1985-03-11
        public int Sex { get; set; } // 1 or 2 should be resolved to proper enum, can be null
        public double Latitude { get; set; } // both Latitude and Longitude should be mapped to GeoPoint object
        public double Longitude { get; set; }
    }

    // Destination class to which we want map our source class
    class Person
    {
        public string FirstName { get; set; }
        public string LastName { get; set; }

        public string[] Skills { get; set; }

        public DateTime? BirthDate { get; set; }

        public Sex Sex { get; set; }

        public GeoPoint Location { get; set; }

        public string FullName
        {
            get { return string.Format("{0} {1}", FirstName, LastName); }
        }

        public int Age
        {
            get { return BirthDate.HasValue ? DateTime.Now.Year - BirthDate.Value.Year : 0; }
        }
    }

    class GeoPoint
    {
        public double Latitude { get; set; }
        public double Longitude { get; set; }

        public override string ToString()
        {
            return string.Format("{0}, {1}", Latitude, Longitude);
        }
    }

    enum Sex
    {
        Undefined = 0,
        Male = 1,
        Female = 2
    }

    #region Resolvers
    class CommaSeparatedListResolver : ValueResolver<string, string[]>
    {
        protected override string[] ResolveCore(string source)
        {
            return (source ?? "").Split(',').Select(item => item.Trim()).Where(item => !string.IsNullOrEmpty(item)).OrderBy(item => item).Distinct().ToArray();
        }
    }

    class SexResolver : ValueResolver<int, Sex>
    {
        protected override Sex ResolveCore(int source)
        {
            switch (source)
            {
                case 1: return Sex.Male;
                case 2: return Sex.Female;
                default: return Sex.Undefined;
            }
        }
    }

    class DateResolver : ValueResolver<string, DateTime?>
    {
        protected override DateTime? ResolveCore(string source)
        {
            try
            {
                return DateTime.ParseExact(source, "yyyy-MM-dd", CultureInfo.InvariantCulture);
            }
            catch (ArgumentNullException ex)
            {
                return null;
            }
            catch (FormatException ex)
            {
                return null;
            }
        }
    }
    #endregion

    class Program
    {
        static void Main(string[] args)
        {
            Mapper.CreateMap<PersonDto, Person>()
                // Inline mappings
                .ForMember(destination => destination.FirstName, member => member.MapFrom(source => source.FullName.Split(' ').FirstOrDefault()))
                .ForMember(destination => destination.LastName, member => member.MapFrom(source => source.FullName.Split(' ').LastOrDefault()))
                .ForMember(destination => destination.Location, member => member.MapFrom(source => new GeoPoint { Latitude = source.Latitude, Longitude = source.Longitude }))
                // Custom resolvers
                .ForMember(destination => destination.Skills, member => member.ResolveUsing<CommaSeparatedListResolver>().FromMember(source => source.Skills))
                .ForMember(destination => destination.BirthDate, member => member.ResolveUsing<DateResolver>().FromMember(source => source.BirthDate))
                .ForMember(destination => destination.Sex, member => member.ResolveUsing<SexResolver>().FromMember(source => source.Sex));

            var row = new PersonDto
            {
                FullName = "Alexandr Marchenko",
                Skills = " php, js,c#,,js ",
                Latitude = 50.450097,
                Longitude = 30.523397
                //Sex = 1
            };

            var person = Mapper.Map<Person>(row);

            Console.WriteLine("FirstName: {0}", person.FirstName); // Alexandr
            Console.WriteLine("FirstName: {0}", person.LastName); // Marchenko
            Console.WriteLine("FullName: {0}", person.FullName); // Alexandr Marchenko
            Console.WriteLine("Skills: {0}", string.Join(", ", person.Skills)); // "c#, js, php"
            Console.WriteLine("BirthDate: {0}", person.BirthDate.HasValue ? person.BirthDate.Value.ToShortDateString() : "NULL"); // NULL
            Console.WriteLine("Age: {0}", person.Age); // 0
            Console.WriteLine("Sex: {0}", person.Sex); // Undefined
            Console.WriteLine("Location: {0}", person.Location); // 50,450097, 30,523397
            Console.ReadKey();
        }
    }
}
```

And here is yet another more full example mapping sql query results to objects in C#:

```csharp
using System;
using System.Collections.Generic;
using System.Data.Entity;
using System.Linq;
using AutoMapper;

namespace AutomapperExample
{
    class ResumeDto
    {
        public int Id { get; set; }
        public string Rubrics { get; set; }
        public string SubRubrics { get; set; }
        public string FullName { get; set; }
        public DateTime BirthDate { get; set; }
        public int Age { get; set; }
        public string Sex { get; set; }
        public string Speciality { get; set; }
        public string Keywords { get; set; }
        public string City { get; set; }
        public string Schedule { get; set; }
        public string Education { get; set; }
        public string ProfLevel { get; set; }
        public DateTime Created { get; set; }
        public DateTime Updated { get; set; }
        public string Photo { get; set; }
        public string Link { get; set; }
        public int Salary { get; set; }
        public string Description { get; set; }
        public string Country { get; set; }
        public string Province { get; set; }
        public decimal Latitude { get; set; }
        public decimal Longitude { get; set; }
    }

    class Storage : DbContext
    {
        public Storage()
            : base("Default")
        {
            Database.SetInitializer<Storage>(null);
        }

        public DbSet<ResumeDto> Resumes { get; set; }

        public IEnumerable<ResumeDto> GetResumes()
        {
            return Resumes.SqlQuery(@"
SELECT
TOP 10
R.Id AS Id,
STUFF((
            SELECT DISTINCT ',' + R1.Name
            FROM TESTSRV13.RabotaUA2.dbo.ResumeRubricNEW AS RR
            INNER JOIN TESTSRV13.RabotaUA2.dbo.Rubric2Level R2 ON RR.RubricID2 = R2.ID
            INNER JOIN TESTSRV13.RabotaUA2.dbo.Rubric1To2 R12 ON R12.RubricID2 = R2.ID
            INNER JOIN TESTSRV13.RabotaUA2.dbo.Rubric1Level R1 ON R1.ID = R12.RubricID1
            WHERE RR.ResumeID = R.Id
            FOR XML PATH(''), TYPE).value('.', 'NVARCHAR(MAX)'), 1, 1, '') AS Rubrics,

STUFF((
            SELECT DISTINCT ',' + R2.Name
            FROM TESTSRV13.RabotaUA2.dbo.ResumeRubricNEW AS RR
            INNER JOIN TESTSRV13.RabotaUA2.dbo.Rubric2Level R2 ON RR.RubricID2 = R2.ID
            INNER JOIN TESTSRV13.RabotaUA2.dbo.Rubric1To2 R12 ON R12.RubricID2 = R2.ID
            INNER JOIN TESTSRV13.RabotaUA2.dbo.Rubric1Level R1 ON R1.ID = R12.RubricID1
            WHERE RR.ResumeID = R.Id
            FOR XML PATH(''), TYPE).value('.', 'NVARCHAR(MAX)'), 1, 1, '') AS SubRubrics,

RTRIM(R.Surname + ' '  + R.Name + ' ' + R.FatherName) AS FullName,
R.BirthDate as BirthDate,
DATEDIFF(year, R.BirthDate, GETDATE()) AS Age,
CASE WHEN R.Sex = 1 THEN 'Мужчина'
        WHEN R.Sex = 0 THEN 'Женщина'
        ELSE NULL
        END AS Sex,
R.Speciality AS Speciality,
R.AlternativeName as Keywords,
C.Name AS City,
S.Name AS Schedule,
E.Name AS Education,
P.Name AS ProfLevel,
R.AddDate AS Created,
R.UpdateDate AS Updated,
CASE WHEN R.Photo = '' THEN 'http://img1.rabota.com.ua/static/2013/11/img/nophoto.png' ELSE 'http://rabota.ua/cvphotos/' + R.Photo END AS Photo,
'http://rabota.ua/cv/' + R.Link AS Link,
CASE WHEN R.CurrencyId = 1 THEN R.Salary
        WHEN R.CurrencyId = 2 THEN 25 * R.Salary
        ELSE NULL
        END AS Salary,
R.Description AS Description,

DC.CountryName AS Country,
DC.AdministrativeAreaName AS Province,
DC.Latitude AS Latitude,
DC.Longitude AS Longitude

FROM TESTSRV13.RabotaUA2.dbo.Resume AS R
INNER JOIN TESTSRV13.RabotaUA2.dbo.ResumeExtra AS RE ON R.Id = RE.ResumeId AND IsModerated = 1 AND IsModeratedRubric = 1 /* только отмодерированные резюме */
INNER JOIN TESTSRV13.RabotaUA2.dbo.Notebook AS N ON R.NotebookId = N.Id AND NotebookStateId NOT IN (4 /* черный список */)
LEFT JOIN TESTSRV13.RabotaUA2.dbo.City AS C ON R.CityId = C.Id
LEFT JOIN TESTSRV13.RabotaUA2.dbo.Schedule AS S ON R.ScheduleId = S.Id
LEFT JOIN TESTSRV13.RabotaUA2.dbo.Education AS E ON R.EducationId = E.Id
LEFT JOIN TESTSRV13.RabotaUA2.dbo.Proflevel AS P ON R.ProfLevelID = P.Id
LEFT JOIN Dictionaries.Cities AS DC ON R.CityId = DC.CityId
WHERE R.State = 1 AND R.IsProfessional = 1 AND R.Speciality LIKE '%PHP%'
ORDER BY R.UpdateDate DESC");
        }
    }


    class Resume
    {
        public int Id { get; set; }
        public string[] Rubrics { get; set; }
        public string[] SubRubrics { get; set; }
        public string FullName { get; set; }
        public DateTime BirthDate { get; set; }
        public int Age { get; set; }
        public string Sex { get; set; }
        public string Speciality { get; set; }
        public string[] Keywords { get; set; }
        public string City { get; set; }
        public string Schedule { get; set; }
        public string Education { get; set; }
        public string ProfLevel { get; set; }
        public DateTime Created { get; set; }
        public DateTime Updated { get; set; }
        public string Photo { get; set; }
        public string Link { get; set; }
        public int Salary { get; set; }
        public string Description { get; set; }
        public string Country { get; set; }
        public string Province { get; set; }
        public decimal Latitude { get; set; }
        public decimal Longitude { get; set; }
    }

    class CommaSeparatedListResolver : ValueResolver<string, string[]>
    {
        protected override string[] ResolveCore(string source)
        {
            return (source ?? "").Split(',').Select(item => item.Trim()).Where(item => !string.IsNullOrEmpty(item)).OrderBy(item => item).Distinct().ToArray();
        }
    }

    class Repository
    {
        private readonly Storage _storage = new Storage();

        public Repository()
        {
            Mapper.CreateMap<ResumeDto, Resume>()
                .ForMember(destination => destination.Rubrics, member => member.ResolveUsing<CommaSeparatedListResolver>().FromMember(source => source.Rubrics))
                .ForMember(destination => destination.SubRubrics, member => member.ResolveUsing<CommaSeparatedListResolver>().FromMember(source => source.SubRubrics))
                .ForMember(destination => destination.Keywords, member => member.ResolveUsing<CommaSeparatedListResolver>().FromMember(source => source.Keywords))
                ;
        }

        public IEnumerable<Resume> GetResumes()
        {
            return Mapper.Map<IEnumerable<Resume>>(_storage.GetResumes());
        }
    }

    class Program
    {
        static readonly Repository Repository = new Repository();

        static void Main(string[] args)
        {
            foreach (var resume in Repository.GetResumes())
            {
                Console.WriteLine("{0}, {1} ({2})", resume.Speciality, resume.FullName, resume.Age);
            }

            Console.ReadKey();
        }
    }
}
```

BTW: [How do you embed sql queries in c#](http://stackoverflow.com/questions/7064214/how-do-you-embed-sql-queries-in-c-sharp) gives nice recommendation to store SQL queries in embedded sql files.
