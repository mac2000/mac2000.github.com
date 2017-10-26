---
layout: post
title: .Net SortedSet

tags: [.net, c#, sortedset, collections]
---

[Новая коллекция SortedSet в .Net 4.0](http://habrahabr.ru/blogs/net/102697/#habracut)

http://www.codeproject.com/KB/cs/SortedSet_T__Collection.aspx

User type example
-----------------

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;

namespace SortedSetTest
{
    public class User : IComparable<User>
    {
        public int id { get; set; }
        public string name { get; set; }

        public User(int Id, string Name)
        {
            id = Id;
            name = Name;
        }

        public int CompareTo(User other)
        {
            if (id < other.id) return -1;
            if (id > other.id) return 1;
            return 0;
        }
    }

    class Program
    {
        static void Main(string[] args)
        {
            var u1 = new User(1, "user1");
            var u2 = new User(2, "user2");
            var u3 = new User(3, "user3");
            var u4 = new User(4, "user4");
            var u5 = new User(5, "user5");
            var u6 = new User(6, "user6");

            var elements1 = new SortedSet<User>() {u1,u2,u1,u3};

            var elements2 = new SortedSet<User>() {u1,u3,u4,u1,u3};

            var union = elements1.Union(elements2);

            foreach (User u in union)
            {
                Console.WriteLine(string.Format("{0,-4} {1}", u.id, u.name));
            }

            Console.ReadKey();
        }
    }
}
```
