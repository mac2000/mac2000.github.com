---
layout: post
title: Knockout.js multiple models editable list
tags: [knockout, inherit]
---

Lets assume that we want to create editable list of something, this is really easy with knockout, but what if we need to be able to host multiple different models in our list?

Here is demo: http://mac-blog.org.ua/examples/knockout/editablelist.html

And here is commented code:

```html
<!DOCTYPE html>
<html>
<head>
	<title>editable list</title>
	<meta charset="utf-8">
	<link href="http://ajax.googleapis.com/ajax/libs/jqueryui/1/themes/base/jquery-ui.css" rel="stylesheet">
	<link href="http://netdna.bootstrapcdn.com/twitter-bootstrap/latest/css/bootstrap-combined.min.css" rel="stylesheet">
</head>
<body>
<div class="container-fluid">
	<table class="table">
		<thead>
		<tr>
			<th>id</th>
			<th>name</th>
			<th>answer</th>
			<th>importance</th>
			<th></th>
		</tr>
		</thead>
		<tbody data-bind="foreach: items">
		<tr>
			<td data-bind="text: id"></td>
			<td data-bind="text: name"></td>
			<td data-bind="text: answer"></td>
			<td data-bind="text: importance"></td>
			<td>
				<a href="#" class="btn btn-mini" data-bind="click: function(){$parent.editItem($data);}">Edit</a>
				<a href="#" class="btn btn-mini" data-bind="click: function(){$parent.items.remove($data);}">Delete</a>
			</td>
		</tr>
		</tbody>
	</table>

	<div class="row-fluid">
		<div class="span6">
			<a href="#" class="btn btn-block" data-bind="click: addLanguage">Add Language Question</a>
		</div>
		<div class="span6">
			<a href="#" class="btn btn-block" data-bind="click: addExperience">Add Experience Question</a>
		</div>
	</div>

	<div id="experience" class="form-horizontal" style="display:none" title="Experience question">
		<div class="control-group">
			<label class="control-label">id</label>
			<div class="controls">
				<input class="input-block-level" type="number" data-bind="value: experience.id" disabled>
			</div>
		</div>

		<div class="control-group">
			<label class="control-label">name</label>
			<div class="controls">
				<input class="input-block-level" type="text" data-bind="value: experience.name">
			</div>
		</div>

		<div class="control-group">
			<label class="control-label">answer</label>
			<div class="controls">
				<select class="input-block-level" data-bind="options: experienceAnswerOptions, value: experience.answer"></select>
			</div>
		</div>

		<div class="control-group">
			<label class="control-label">importance</label>
			<div class="controls">
				<select class="input-block-level" data-bind="options: importanceOptions, value: experience.importance"></select>
			</div>
		</div>
	</div>

	<div id="language" class="form-horizontal" style="display:none" title="Language question">
		<div class="control-group">
			<label class="control-label">id</label>
			<div class="controls">
				<input class="input-block-level" type="number" data-bind="value: language.id" disabled>
			</div>
		</div>

		<div class="control-group">
			<label class="control-label">name</label>
			<div class="controls">
				<input class="input-block-level" type="text" data-bind="value: language.name" disabled>
			</div>
		</div>

		<div class="control-group">
			<label class="control-label">language</label>
			<div class="controls">
				<select class="input-block-level" data-bind="options: languageOptions, value: language.language"></select>
			</div>
		</div>

		<div class="control-group">
			<label class="control-label">answer</label>
			<div class="controls">
				<select class="input-block-level" data-bind="options: languageAnswerOptions, value: language.answer"></select>
			</div>
		</div>

		<div class="control-group">
			<label class="control-label">importance</label>
			<div class="controls">
				<select class="input-block-level" data-bind="options: importanceOptions, value: language.importance"></select>
			</div>
		</div>
	</div>

</div>

<script src="http://ajax.googleapis.com/ajax/libs/jquery/1/jquery.min.js"></script>
<script src="http://ajax.googleapis.com/ajax/libs/jqueryui/1/jquery-ui.min.js"></script>
<script type="text/javascript" src="http://knockoutjs.com/downloads/knockout-2.2.1.js"></script>

<script type="text/javascript">
	(function ($) {
		// we are creating own dialog widget to get rid of duplicate code in our model definition (see later)
		$.widget("rua.callbackdialog", $.ui.dialog, {
			options:{
				modal:true,
				resizable:false,
				width:400,
				callback:function () {
					return true;
				},
				saveButtonText: 'Save',
				cancelButtonText: 'Cancel'
			},
			_create:function () {
				// before calling parents constructor we are adding dialog buttons with our code
				if (!this.options.buttons.length) {
					this.options.buttons = [];
					if(this.options.saveButtonText) {
						this.options.buttons.push({
							text: this.options.saveButtonText,
							click:function () {
								// here is main part, "Save" button will call callback and if it will return true - will close the dialog
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
</script>

<script type="text/javascript">
	// Base model, that will be inherited by other models
	// it need for setting values via options argument
	function QuestionBaseModel(options) {
		this.init = function (options) {
			var val;
			if (options) {
				for(key in options) {
					// here we are checking if concrete options attribute is function (observable)
					val = (typeof options[key] === 'function') ? options[key]() : options[key];
					// try catch need here because there are cases when this code will try to change computed value, knockout will throw an exception in that case
					try {
						if(typeof this[key] === 'function') this[key](val);
						else this[key] = val;
					} catch(err) {}
				}
			}
		};
		this.init.apply(this, arguments);
	}

	// out first model
	function QuestionExperienceModel(options) {
		this.id = ko.observable();
		this.name = ko.observable();
		this.importance = ko.observable();
		this.answer = ko.observable();

		this.init.apply(this, arguments); // apply given options
	}
	QuestionExperienceModel.prototype = new QuestionBaseModel(); // inherit it from base model

	// our second model
	function QuestionLanguageModel(options) {
		this.id = ko.observable();
		this.language = ko.observable();
		this.name = ko.computed(function () {
			return 'Your knowledge of ' + this.language();
		}, this);
		this.importance = ko.observable();
		this.answer = ko.observable();

		this.init.apply(this, arguments); // apply given options
	}
	QuestionLanguageModel.prototype = new QuestionBaseModel(); // inherit it from base model

	// main model which will contain observable array of QuestionExperienceModel and QuestionLanguageModel models
	function List() {
		var self = this;

		self.importanceOptions = ['Non important', 'Semi important', 'Important'];
		self.languageOptions = ['English', 'Ukrainian'];
		self.experienceAnswerOptions = ['1+ year', '2+ years', '5+ years'];
		self.languageAnswerOptions = ['little', 'good', 'carrier'];

		self.items = ko.observableArray([
			new QuestionExperienceModel({id:1, name:'Your experience in JS', importance:'Important', answer:'2+ years'}),
			new QuestionLanguageModel({id:2, importance:'Semi important', answer:'good', language:'Ukrainian'})
		]);

		// dummy items that is used by editor dialogs
		self.experience = new QuestionExperienceModel();
		self.language = new QuestionLanguageModel();

		// on adding we are just creating new instance of desired model with default arguments and editing it
		// notice second boolean argument indicating that this is new item that must be added to items array later
		self.addLanguage = function () {
			var item = new QuestionLanguageModel({id:0, importance:'Important', answer:'good', language:'English'});
			self.editItem(item, true);
		};

		// on adding we are just creating new instance of desired model with default arguments and editing it
		// notice second boolean argument indicating that this is new item that must be added to items array later
		self.addExperience = function () {
			var item = new QuestionExperienceModel({id:0, name:'Your experience in ...', importance:'Important', answer:'2+ years'});
			self.editItem(item, true);
		};

		// we are calling this method in both cases on creating and editing items
		// second boolean argument indicating that this is new item that must be added to items array later
		self.editItem = function(item, isNew){
			if(item instanceof QuestionLanguageModel) self.editLanguage(item, isNew);
			else self.editExperience(item, isNew);
		};

		self.editExperience = function (item, isNew) {
			// fill dummy experience with given item
			self.experience.init(item);
			// show edit dialog
			$('#experience').callbackdialog({
				callback: function () {
					//TODO: validate here and return false if need
					// if all ok save data back to item from dummy experience
					item.init(self.experience);
					// if this is a new item - add it to items array
					if (isNew) self.items.push(item);
					return true;
				}
			});
		};

		self.editLanguage = function (item, isNew) {
			// fill dummy language with given item
			self.language.init(item);
			// show edit dialog
			$('#language').callbackdialog({
				callback: function () {
					//TODO: validate here and return false if need
					// if all ok save data back to item from dummy language
					item.init(self.language);
					// if this is a new item - add it to items array
					if (isNew) self.items.push(item);
					return true;
				}
			});
		};
	}

	ko.applyBindings(new List());
</script>
</body>
</html>
```
