---
layout: post
title: Asp.net string to Type

tags: [.net, appdomain, assembly, c#, convert, reflection, type]
---

I have string with class name, and want convert it to Type, here how it can be done:

*Do not forget* - class name must be full with namespace, for example `RabotaUA.Entity.Model.SaleCompanyInfo`

```csharp
public class Tmp
{
    public static Type StringToType(string typeName)
    {
        foreach (System.Reflection.Assembly assembly in AppDomain.CurrentDomain.GetAssemblies() /*AssemblyLocator.GetAssemblies()*/)
        {
            Type foundType = assembly.GetType(typeName);

            if (foundType != null)
                return foundType;
        }
        return null;
    }
}
```

In some cases this class will be also very useful

```csharp
public static class AssemblyLocator
{
    private static readonly ReadOnlyCollection<Assembly> AllAssemblies;
    private static readonly ReadOnlyCollection<Assembly> BinAssemblies;

    static AssemblyLocator()
    {
        AllAssemblies = new ReadOnlyCollection<Assembly>(
            BuildManager.GetReferencedAssemblies().Cast<Assembly>().ToList());

        IList<Assembly> binAssemblies = new List<Assembly>();

        string binFolder = HttpRuntime.AppDomainAppPath + "bin\\";
        IList<string> dllFiles = Directory.GetFiles(binFolder, "*.dll", SearchOption.TopDirectoryOnly).ToList();

        foreach (string dllFile in dllFiles)
        {
            AssemblyName assemblyName = AssemblyName.GetAssemblyName(dllFile);

            Assembly locatedAssembly = AllAssemblies.FirstOrDefault(a =>
                AssemblyName.ReferenceMatchesDefinition(
                    a.GetName(), assemblyName));

            if (locatedAssembly != null)
            {
                binAssemblies.Add(locatedAssembly);
            }
        }

        BinAssemblies = new ReadOnlyCollection<Assembly>(binAssemblies);
    }

    public static ReadOnlyCollection<Assembly> GetAssemblies()
    {
        return AllAssemblies;
    }

    public static ReadOnlyCollection<Assembly> GetBinFolderAssemblies()
    {
        return BinAssemblies;
    }
}
```
