---
layout: post
title: jQuery toggle visibility plugin
tags: [jquery, visibility, toggle]
---

```js
(function($){
	$.fn.toggleVisibility = function(state) {
		var bool = typeof state === 'boolean';

		return this.each(function() {
			var isHidden = $(this).css('visibility') === 'hidden';

			if (bool ? state : isHidden) {
				jQuery(this).css('visibility', 'visible');
			} else {
				jQuery(this).css('visibility', 'hidden');
			}
		});
	};
})(jQuery);
```