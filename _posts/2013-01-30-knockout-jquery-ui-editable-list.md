---
layout: post
title: Knockout.js, jQuery UI, editable observableArray list
tags: [jquery, ui, dialog, knockout. observableArray]
---

Short note about implementing most usefull knockout use case - editable list.

So we have list of items, and want to do some CRUD on it.

Here is sample code:

```html
<!DOCTYPE html>
<html>
<head>
	<title>list</title>
	<meta charset="utf-8">
	<link rel="stylesheet" href="components/jquery-ui/themes/base/jquery.ui.all.css">
	<link rel="stylesheet" href="components/bootstrap/docs/assets/css/bootstrap.css">
</head>
<body>
<div class="container">
	<table class="table">
		<thead>
		<tr>
			<th>id</th>
			<th>name</th>
			<th>price</th>
			<th></th>
		</tr>
		</thead>
		<tbody data-bind="foreach: items">
		<tr>
			<td data-bind="text: id"></td>
			<td data-bind="text: name"></td>
			<td data-bind="text: price"></td>
			<td>
				<!-- call to editItem must be exactly the same, otherwise there will be event as second argument, look to addItem and editItem methods -->
				<a href="#" class="btn btn-mini" data-bind="click: function(){$parent.editItem($data);}">Edit</a>
				<a href="#" class="btn btn-mini" data-bind="click: function(){$parent.removeItem($data);}">Delete</a>
			</td>
		</tr>
		</tbody>
	</table>

	<a href="#" class="btn btn-block" data-bind="click: addItem">Add Item</a>

	<div id="editDisplay" class="form-horizontal" style="display:none">
		<div class="control-group">
			<label class="control-label">id</label>

			<div class="controls">
				<input class="input-block-level" type="number" data-bind="value: editor.id" disabled>
			</div>
		</div>

		<div class="control-group">
			<label class="control-label">name</label>

			<div class="controls">
				<input class="input-block-level" type="text" data-bind="value: editor.name">
			</div>
		</div>

		<div class="control-group">
			<label class="control-label">price</label>

			<div class="controls">
				<input class="input-block-level" type="number" data-bind="value: editor.price">
			</div>
		</div>
	</div>
</div>


<script type="text/javascript" src="components/jquery/jquery.js"></script>
<script type="text/javascript" src="components/jquery-ui/ui/jquery-ui.custom.js"></script>
<script type="text/javascript" src="components/knockout/build/output/knockout-latest.debug.js"></script>


<script type="text/javascript">
	// Our KISS item modell
	function Item(id, name, price) {
		this.id = ko.observable(id);
		this.name = ko.observable(name);
		this.price = ko.observable(price);
	}

	function List() {
		var self = this;

		self.items = ko.observableArray([
			new Item(1, "one", 5.50),
			new Item(2, "two", 7.25),
			new Item(3, "three", 4.75)
		]);

		self.addItem = function () {
			// Here we create new item with default parameters
			// and call editItem for it
			// notice second argument wich is boolean
			// indicating that this is new item
			// IMPORTANT: in template you MUST use folow:
			// data-bind="click: function(){$parent.editItem($data);}"
			// otherwise it will pass event as second argument
			var item = new Item(0, 'default', 0);
			self.editItem(item, true);
		};

		self.removeItem = function (item) {
			self.items.remove(item);
		};

		// dummy item model for editor
		self.editor = new Item();

		self.editItem = function (item, isNew) {
			// populate editor values
			self.editor.id(item.id());
			self.editor.name(item.name());
			self.editor.price(item.price());

			// create dialog
			$('#editDisplay').dialog({
				modal:true,
				resizable:false,
				width:400,
				buttons:{
					Accept:function () {
						//TODO: validate here

						$(this).dialog("close");

						// save values from editor to item
						item.name(self.editor.name());
						item.price(self.editor.price());

						// and add it if it is new one
						if (isNew) {
							self.items.push(item);
						}
					},
					Cancel:function () {
						$(this).dialog("close");
					}
				}
			});
		};
	}

	ko.applyBindings(new List());
</script>
</body>
</html>
```

And here is live demo: http://jsfiddle.net/mac2000/N2zNk/embedded/result/
