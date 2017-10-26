---
layout: post
title: jQuery UI dialog callback
tags: [jquery, ui, dialog, callback]
---

DRY. If you have many dialogs with same settings, but different content and save button handler - create own widget. Here is simple starter kit example:

```js
(function ($) {
	$.widget("mac.callbackdialog", $.ui.dialog, {
		options:{
			// put any other default options here
			// modal: true,
			// autoOpen: false,
			// ...

			// Here is our callback function.
			// The main idea is that you will run validation here
			// and will try to save data
			// and if all ok retrun true
			callback:function () {
				return true;
			},
			saveButtonText: 'Save',
			cancelButtonText: 'Cancel'
		},
		_create:function () {
			if(this.options.buttons.length === 0) {
				// Here we are adding our buttons
				if(this.options.saveButtonText) {
					this.options.buttons.push({
						text: this.options.saveButtonText,
						click:function () {
							// And here it is!
							// We are calling our callback
							// and if all ok - closing dialog
							if ($(this).callbackdialog('option', 'callback')()) {
								$(this).callbackdialog('close');
							}
						}
					});
				}

				if(this.options.cancelButtonText) {
					this.options.buttons.push({
						text: this.options.cancelButtonText,
						click:function () {
							$(this).callbackdialog('close');
						}
					});
				}
			}

			$.ui.dialog.prototype._create.call(this);
		},
		destroy:function () {
			$.ui.dialog.prototype.destroy.call(this);
		},
		_setOption:function () {
			$.ui.dialog.prototype._setOption.apply(this, arguments);
		}
	});
}(jQuery));
```

Usage example:

```html
<!DOCTYPE html>
<html>
<head>
	<title>callback</title>
	<link rel="stylesheet" href="components/jquery-ui/themes/base/jquery.ui.all.css">
</head>
<body>
<div id="dlg">
	<p>Hello World</p>
</div>
<script type="text/javascript" src="components/jquery/jquery.js"></script>
<script type="text/javascript" src="components/jquery-ui/ui/jquery-ui.custom.js"></script>
<script type="text/javascript" src="js/jquery.ui.callbackdialog.js"></script>
<script type="text/javascript">
	$('#dlg').callbackdialog({
		callback: function() {
			alert('TODO: return true if valid and saved');
			return true;
		}
	});
</script>
</body>
</html>
```

