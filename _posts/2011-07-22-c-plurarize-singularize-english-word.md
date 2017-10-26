---
layout: post
title: C# plurarize/singularize english word

tags: [.net, c#, form, plural, plurarize, seo, singular, singularize, text, word]
---

Found at:

http://lotsacode.wordpress.com/2010/03/05/singularization-pluralization-in-c/

http://mattgrande.wordpress.com/2009/10/28/pluralization-helper-for-c/

**Code:**

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Text.RegularExpressions;

namespace DFP
{
    public class Utils
    {
        private static readonly IList<string> Unpluralizables = new List<string> { "equipment", "information", "rice", "money", "species", "series", "fish", "sheep", "deer" };
        private static readonly IDictionary<string, string> Pluralizations = new Dictionary<string, string>
        {
            // Start with the rarest cases, and move to the most common
            { "person", "people" },
            { "ox", "oxen" },
            { "child", "children" },
            { "foot", "feet" },
            { "tooth", "teeth" },
            { "goose", "geese" },
            // And now the more standard rules.
            { "(.*)fe?", "$1ves" },         // ie, wolf, wife
            { "(.*)man$", "$1men" },
            { "(.+[aeiou]y)$", "$1s" },
            { "(.+[^aeiou])y$", "$1ies" },
            { "(.+z)$", "$1zes" },
            { "([m|l])ouse$", "$1ice" },
            { "(.+)(e|i)x$", @"$1ices"},    // ie, Matrix, Index
            { "(octop|vir)us$", "$1i"},
            { "(.+(s|x|sh|ch))$", @"$1es"},
            { "(.+)", @"$1s" }
        };

        private static readonly IDictionary<string, string> Singularizations = new Dictionary<string, string>
        {
            // Start with the rarest cases, and move to the most common
            {"people", "person"},
            {"oxen", "ox"},
            {"children", "child"},
            {"feet", "foot"},
            {"teeth", "tooth"},
            {"geese", "goose"},
            // And now the more standard rules.
            {"(.*)ives?", "$1ife"},
            {"(.*)ves?", "$1f"},
            // ie, wolf, wife
            {"(.*)men$", "$1man"},
            {"(.+[aeiou])ys$", "$1y"},
            {"(.+[^aeiou])ies$", "$1y"},
            {"(.+)zes$", "$1"},
            {"([m|l])ice$", "$1ouse"},
            {"matrices", @"matrix"},
            {"indices", @"index"},
            {"(.+[^aeiou])ices$","$1ice"},
            {"(.*)ices", @"$1ex"},
            // ie, Matrix, Index
            {"(octop|vir)i$", "$1us"},
            {"(.+(s|x|sh|ch))es$", @"$1"},
            {"(.+)s", @"$1"}
        };

        public static string Pluralize(int count, string singular)
        {
            if (count == 1)
                return singular;

            if (Unpluralizables.Contains(singular))
                return singular;

            var plural = "";

            foreach (var pluralization in Pluralizations)
            {
                if (Regex.IsMatch(singular, pluralization.Key))
                {
                    plural = Regex.Replace(singular, pluralization.Key, pluralization.Value);
                    break;
                }
            }

            return plural;
        }

        public static string Singularize(string word)
        {
            if (Unpluralizables.Contains(word.ToLowerInvariant()))
            {
                return word;
            }

            foreach (var singularization in Singularizations)
            {
                if (Regex.IsMatch(word, singularization.Key))
                {
                    return Regex.Replace(word, singularization.Key, singularization.Value);
                }
            }

            return word;
        }

        public static bool IsPlural(string word)
        {
            if (Unpluralizables.Contains(word.ToLowerInvariant()))
            {
                return true;
            }

            foreach (var singularization in Singularizations)
            {
                if (Regex.IsMatch(word, singularization.Key))
                {
                    return true;
                }
            }

            return false;
        }
    }
}
```
