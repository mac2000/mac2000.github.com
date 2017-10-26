---
layout: post
title: Xunit custom traits for categories
tags: [xunit, trait, category, attribute]
---

In xunit you able to use `[Trait("Category", "Sample")]` for your tests, and here is how you can simplify things a little bit:

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using Xunit;
using Xunit.Abstractions;
using Xunit.Sdk;
using XunitCategoriesSample.Traits;

namespace XunitCategoriesSample.Traits
{
	public class CategoryDiscoverer : ITraitDiscoverer
	{
		public const string KEY = "Category";

		public IEnumerable<KeyValuePair<string, string>> GetTraits(IAttributeInfo traitAttribute)
		{
			var ctorArgs = traitAttribute.GetConstructorArguments().ToList();
			yield return new KeyValuePair<string, string>(KEY, ctorArgs[0].ToString());
		}
	}

	//NOTICE: Take a note that you must provide appropriate namespace here
	[TraitDiscoverer("XunitCategoriesSample.Traits.CategoryDiscoverer", "XunitCategoriesSample")]
	[AttributeUsage(AttributeTargets.Method, AllowMultiple = true)]
	public class CategoryAttribute : Attribute, ITraitAttribute
	{
		public CategoryAttribute(string category) { }
	}
}

namespace XunitCategoriesSample
{
	public class Class1
	{
		[Fact]
		[Category("Jobsearcher")]
		public void PassingTest()
		{
			Assert.Equal(4, Add(2, 2));
		}

		[Fact]
		[Category("Employer")]
		public void FailingTest()
		{
			Assert.Equal(5, Add(2, 2));
		}

		int Add(int x, int y)
		{
			return x + y;
		}
	}
}
```


**NOTICE** you must provide right namespaces in `TraitDiscoverer` attribute.

But here is more, lets make even more specialized attributes:

```csharp
public class JobsearcherTraitDiscoverer : ITraitDiscoverer
{
	public const string VALUE = "Jobsearcher";

	public IEnumerable<KeyValuePair<string, string>> GetTraits(IAttributeInfo traitAttribute)
	{
		yield return new KeyValuePair<string, string>(CategoryDiscoverer.KEY, VALUE);
	}
}

[TraitDiscoverer("XunitCategoriesSample.Traits.JobsearcherTraitDiscoverer", "XunitCategoriesSample")]
[AttributeUsage(AttributeTargets.Method, AllowMultiple = true)]
public class JobsearcherTraitAttribute : Attribute, ITraitAttribute
{
	public JobsearcherTraitAttribute()
	{
	}
}
```


So from now on you will be able to just type `[JobsearcherTrait]`

**Links**:


https://github.com/xunit/xunit/issues/394 - discussion about why TraitAttribute was marked as sealed

https://github.com/xunit/samples.xunit/tree/master/TraitExtensibility - sample by xunit how to make custom attributes

https://github.com/wespday/CategoryTraits.Xunit2 - one more sample

https://github.com/xunit/xunit/blob/47fdc2669ae6aa28f6d642e202840193dfc7dbd7/test/test.xunit.execution/Common/TraitHelperTests.cs - xunit test sample of implementing custom attributes

