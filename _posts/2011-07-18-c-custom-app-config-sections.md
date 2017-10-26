---
layout: post
title: C# custom App.config sections

tags: [.net, app.config, c#, config, configSections, ConfigurationSettings, NameValueSectionHandler, section, SingleTagSectionHandler]
---

**Code:**

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Configuration;
using System.Collections.Specialized;
using System.Collections;

namespace custom_app_config
{
    class Program
    {
        static void Main(string[] args)
        {
            //appSettings
            string appsettings_login = ConfigurationManager.AppSettings.Get("login");
            Console.WriteLine("appSettings/login: " + appsettings_login);
            Console.WriteLine();

            //one tag section
            IDictionary oneTag = (IDictionary)ConfigurationManager.GetSection("auth");
            string auth_login = (string)oneTag["login"];
            string auth_password = (string)oneTag["password"];
            Console.WriteLine("auth/login: " + auth_login);
            Console.WriteLine("auth/password: " + auth_password);
            Console.WriteLine();

            //name value collection section (the same as appSettings)
            NameValueCollection faurls = (NameValueCollection)ConfigurationManager.GetSection("FavoriteUrls");
            foreach (string key in faurls.AllKeys)
            {
                Console.WriteLine("FavoriteUrls/" + key + ": " + faurls[key]);
            }
            Console.WriteLine();

            //custom section
            StartupFoldersConfigSection section = (StartupFoldersConfigSection)ConfigurationManager.GetSection("StartupFolders");
            foreach (FolderElement item in section.FolderItems)
            {
                Console.WriteLine("StartupFolders/Folders/" + item.FolderType + ": " + item.Path);
            }
            Console.WriteLine();

            Console.ReadKey();
        }
    }

    public class StartupFoldersConfigSection : ConfigurationSection
    {
        [ConfigurationProperty("Folders")]
        public FoldersCollection FolderItems
        {
            get { return ((FoldersCollection)(base["Folders"])); }
        }
    }

    [ConfigurationCollection(typeof(FolderElement))]
    public class FoldersCollection : ConfigurationElementCollection
    {
        protected override ConfigurationElement CreateNewElement()
        {
            return new FolderElement();
        }
        protected override object GetElementKey(ConfigurationElement element)
        {
            return ((FolderElement)(element)).FolderType;
        }
        public FolderElement this[int idx]
        {
            get { return (FolderElement)BaseGet(idx); }
        }
    }

    public class FolderElement : ConfigurationElement
    {
        [ConfigurationProperty("folderType", DefaultValue = "", IsKey = true, IsRequired = true)]
        public string FolderType
        {
            get { return ((string)(base["folderType"])); }
            set { base["folderType"] = value; }
        }

        [ConfigurationProperty("path", DefaultValue = "", IsKey = false, IsRequired = false)]
        public string Path
        {
            get { return ((string)(base["path"])); }
            set { base["path"] = value; }
        }
    }
}
```

**App.config:**

```xml
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
    <configSections>
    <!-- one tag config section with attributes -->
    <section name="auth" type="System.Configuration.SingleTagSectionHandler" />
    <!-- the same as appSettings -->
    <section name="FavoriteUrls" type="System.Configuration.NameValueSectionHandler" />
    <!-- custom section -->
    <section name="StartupFolders" type="custom_app_config.StartupFoldersConfigSection, custom_app_config"/>
    </configSections>

    <appSettings>
    <add key="login" value="admin"/>
    </appSettings>

    <auth login="admin" password="123" />

    <FavoriteUrls>
    <add key="Microsoft" value="http://www.microsoft.com/" />
    <add key="DotNetSpider" value="http://www.DotNetSpider.com/" />
    <add key="AsianSpider" value="http://www.AsianSpider.com/" />
    </FavoriteUrls>

    <StartupFolders>
    <Folders>
        <add folderType="A" path="c:\foo" />
        <add folderType="B" path="C:\foo1" />
    </Folders>
    </StartupFolders>

</configuration>
```
