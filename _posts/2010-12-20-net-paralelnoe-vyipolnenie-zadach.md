---
layout: post
title: .Net паралельное выполнение задач

tags: [.net, c#, parralel, threading, tasks, async]
---

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace ConsoleApplication1
{
    class Program
    {

        static void Main(string[] args)
        {
            StringBuilder sb = new StringBuilder();

            DateTime s = DateTime.Now;
            for (double i = 1; i < 9999999; i++)
            {
                double z;
                for(int x = 0; x < 50; x++)
                z = Math.Sqrt(i);
                //sb.Append(i);
            }
            DateTime e = DateTime.Now;

            Console.WriteLine(s);
            Console.WriteLine(e);

            //Console.ReadKey();

            StringBuilder sb2 = new StringBuilder();
            s = DateTime.Now;
            Parallel.For(1, 9999999, delegate(int i)
            {
                double z;
                for(int x = 0; x < 50; x++)
                z = Math.Sqrt(i);
                //sb2.Append(i);
            });
            e = DateTime.Now;

            Console.WriteLine(s);
            Console.WriteLine(e);

            Console.ReadKey();

        }
    }
}
```
