---
layout: post
title: C# Abstract Factory

tags: [.net, c#, designpatterns, oop]
---

```csharp
using System;
using System.Collections.Generic;
using System.Text;
using System.Reflection;

namespace ConsoleApplication1
{
    class Program
    {
        static void Main(string[] args)
        {

            Console.WriteLine(Factory.Generate("Number"));
            Console.WriteLine(Factory.Generate("Number", 1000));
            Console.WriteLine(Factory.Generate("Number", 5000,5100));

            Console.ReadKey();
        }
    }

    interface IGenerator
    {
        string Generate();
    }

    class Number : IGenerator
    {
        public Random rnd = new Random();

        public string Generate()
        {
            return Generate(0, 9);
        }

        public string Generate(int max)
        {
            return Generate(0, max);
        }

        public string Generate(int min, int max)
        {
            return rnd.Next(min, max).ToString();
        }
    }

    class Factory
    {
        public static string Generate(string type, params object[] args)
        {
            Assembly assembly = Assembly.GetAssembly(typeof(Number));
            AssemblyName assemblyName = assembly.GetName();

            Type generatorType = assembly.GetType(assemblyName.Name + "." + type);

            Type[] argTypes = new Type[args.Length];
            for (int i = 0; i < args.Length; i++)
            {
                argTypes[i] = args[i].GetType();
            }

            MethodInfo mi = generatorType.GetMethod("Generate", argTypes);
            IGenerator Generator = (IGenerator)Activator.CreateInstance(generatorType);

            return mi.Invoke(Generator, args).ToString();
        }
    }
}
```
