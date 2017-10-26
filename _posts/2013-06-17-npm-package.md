---
layout: post
title: NPM package
tags: [npm]
---

https://npmjs.org/

**Search for package**

```sh
npm search express
```

**Create package.json file**

```sh
npm init
```

**Example of package.json file**

```json
{
	"name": "example",
	"version": "0.0.0",
	"description": "example",
	"author": "Marchenko Alexandr <marchenko.alexandr@gmail.com>",
	"license": "BSD",
	"dependencies": {
		"express": "x"
	},
	"devDependencies": {
		"grunt": "x"
	}
}
```

**package.sublime-snippet**

```
<snippet>
	<content><![CDATA[{
	"name": "${1:example}",
	"version": "${2:0.0.0}",
	"description": "${3:example}",
	"author": "Marchenko Alexandr <marchenko.alexandr@gmail.com>",
	"license": "BSD",
	"dependencies": {
		${4:"express": "x"}
	},
	"devDependencies": {
		${5:"grunt": "x"}
	}
}]]></content>
	<tabTrigger>package</tabTrigger>
	<scope>source.json, text.plain</scope>
</snippet>
```
