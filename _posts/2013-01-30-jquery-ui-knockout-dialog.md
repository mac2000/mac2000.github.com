---
layout: post
title: jQuery UI knockout dialog
tags: [jquery, ui, dialog, callback, knockout]
---

Extending jQuery UI dialog with knockout model plus callback.

```js
(function ($) {
	$.widget("mac.knockoutdialog", $.ui.dialog, {
		options:{
			model:function(){},
			callback:function (model) {
				return true;
			},
			saveButtonText: 'Save',
			cancelButtonText: 'Cancel'
		},
		_create:function () {
			if(this.options.buttons.length === 0) {
				if(this.options.saveButtonText) {
					this.options.buttons.push({
						text: this.options.saveButtonText,
						click:function () {
							if ($(this).knockoutdialog('option', 'callback')($(this).knockoutdialog('option', 'model'))) {

								$(this).knockoutdialog('close');
							}
						}
					});
				}

				if(this.options.cancelButtonText) {
					this.options.buttons.push({
						text: this.options.cancelButtonText,
						click:function () {
							$(this).knockoutdialog('close');
						}
					});
				}
			}

			$.ui.dialog.prototype._create.call(this);

			this.options.model = new this.options.model();
			this.options.model.element = this.element;
			this.options.model.dialog = this;
			ko.applyBindings(this.options.model, this.element[0]);
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
	<title>ko</title>
	<link rel="stylesheet" href="components/jquery-ui/themes/base/jquery.ui.all.css">
</head>
<body>
<div id="dlg">
	<p><input type="text" data-bind="value: name, valueUpdate: 'afterkeyup'"></p>
	<p data-bind="text: name"></p>
</div>
<script type="text/javascript" src="components/jquery/jquery.js"></script>
<script type="text/javascript" src="components/jquery-ui/ui/jquery-ui.custom.js"></script>
<script type="text/javascript" src="components/knockout/build/output/knockout-latest.debug.js"></script>

<script type="text/javascript" src="js/jquery.ui.callbackdialog.js"></script>
<script type="text/javascript" src="js/jquery.ui.knockoutdialog.js"></script>

<script type="text/javascript">
	$('#dlg').knockoutdialog({
		model: function(){
			this.name = ko.observable();
		},
		callback: function(model){
			return model.name().length > 0;
		}
	});
	// $('#dlg').knockoutdialog('option', 'model').name('Hello'); // access model
	// $('#dlg').knockoutdialog('option', 'model').element.knockoutdialog('close'); // access dialog from model
	// $('#dlg').knockoutdialog('option', 'model').dialog.close(); // access dialog from model
</script>
</body>
</html>
```
