---
layout: post
title: jQuery UI override selectable to work with shift key
tags: [jquery, ui, selectable]
---

jQuery UI selectable plugin by default wont work with shift key as you expected, so lets try to fix this.

Here is few ways to accomplish this:

http://stackoverflow.com/questions/9374743/enable-shift-multiselect-in-jquery-ui-selectable

And here is how to do it without any plugins:

```js
var prev = -1; // here we will store index of previous selection
$('tbody').selectable({
	selecting: function(e, ui) { // on select
		var curr = $(ui.selecting.tagName, e.target).index(ui.selecting); // get selecting item index
		if(e.shiftKey && prev > -1) { // if shift key was pressed and there is previous - select them all
			$(ui.selecting.tagName, e.target).slice(Math.min(prev, curr), 1 + Math.max(prev, curr)).addClass('ui-selected');
			prev = -1; // and reset prev
		} else {
			prev = curr; // othervise just save prev
		}
	}
});
```

http://jsfiddle.net/mac2000/DJFaL/1/embedded/result/

But this is not our way, lets override widget itself, here is basic way to override any widget:

```js
$.widget("my.selectable", $.ui.selectable, {
	options: {},
	_create: function() {
		$.ui.selectable.prototype._create.call(this);
	},
	destroy: function() {
		$.ui.selectable.prototype.destroy.call(this);
	},
	_setOption: function() {
		$.ui.selectable.prototype._setOption.apply(this, arguments);
	}
});
```

Now all we need is to combine this with previous solution, and we get:

```js
$.widget("shift.selectable", $.ui.selectable, {
	options: {},
	previousIndex: -1,
	currentIndex: -1,
	_create: function() {
		var self = this;

		$.ui.selectable.prototype._create.call(this);

		$(this.element).on('selectableselecting', function(event, ui){
			self.currentIndex = $(ui.selecting.tagName, event.target).index(ui.selecting);
			if(event.shiftKey && self.previousIndex > -1) {
				$(ui.selecting.tagName, event.target).slice(Math.min(self.previousIndex, self.currentIndex), 1 + Math.max(self.previousIndex, self.currentIndex)).addClass('ui-selected');
				self.previousIndex = -1;
			} else {
				self.previousIndex = self.currentIndex;
			}
		});
	},
	destroy: function() {
		$.ui.selectable.prototype.destroy.call(this);
	},
	_setOption: function() {
		$.ui.selectable.prototype._setOption.apply(this, arguments);
	}
});
```

http://mac-blog.org.ua/examples/jquery/ui/jquery.ui.shiftselectable.html