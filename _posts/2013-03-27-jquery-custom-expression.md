---
layout: post
title: jQuery Custom Expression
tags: [jquery, expression, custom]
---

Here is simple example how you can provide custom expressions for jquery like `:visible`, `:first` etc.

```html
<!doctype html>
<title>jQuery Custom Expression</title>
<meta charset="UTF-8">

<span>Hello</span>
<span style="display:block">World</span>

<script src="http://ajax.googleapis.com/ajax/libs/jquery/1/jquery.min.js"></script>
<script>
	(function($){
		$.extend($.expr[':'], {
			'block': function(a, i, m) {
				return $(a).css('display') == 'block';
			}
		});
	})(jQuery);
</script>
<script>
	jQuery('span:block').hide();
</script>
```