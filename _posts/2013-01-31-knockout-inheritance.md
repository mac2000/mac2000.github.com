---
layout: post
title: Knockout.js model inheritance
tags: [knockout. inherit]
---

DRY. If you are copy pasting your model methods/properties etc - you are on wrong way. Just inherit them from base model. Here is example:

```html
<!DOCTYPE html>
<html>
<head>
	<title>inheritance simple</title>
	<meta charset="utf-8">
</head>
<body>
<div class="container">
	<div data-bind="foreach: items">
		<p data-bind="text: toString()"></p>
	</div>
</div>


<script type="text/javascript" src="components/jquery/jquery.js"></script>
<script type="text/javascript" src="components/knockout/build/output/knockout-latest.debug.js"></script>


<script type="text/javascript">
	// Base model, that will be inherited
	// notice that options are provided as object, rather than simple arguments
	function QuestionBaseModel(options) {
		this.id = ko.observable();
		this.name = ko.observable();
		this.importance = ko.observable();
		this.importanceOptions = ['Non important', 'Semi important', 'Important'];
		this.answer = ko.observable();

		// found at: http://stackoverflow.com/questions/10520400/knockout-issue-with-prototypical-inheritance
		this.init = function (options) {
			if (options) {
				for(key in options) {
					if(typeof this[key] === 'function') {
						this[key](options[key]);
					} else {
						this[key] = options[key];
					}
				}
				//if(options.id) this.id(options.id);
				//if(options.name) this.name(options.name);
				//if(options.importance) this.importance(options.importance);
				//if(options.answer) this.answer(options.answer);
			}
		};
		this.init.apply(this, arguments); // apply given options
	}


	function QuestionExperienceModel(options) {
		this.answerOptions = ['little', '2+ years', 'huge']; // provide available answers

		this.toString = function() {
			return ko.toJSON(this);
		};

		this.init.apply(this, arguments); // apply given options
	}
	QuestionExperienceModel.prototype = new QuestionBaseModel(); // inherit from base model

	function QuestionLanguageModel(options) {
		this.language = ko.observable(options.language); // additional field
		this.languageOptions = ['English', 'Ukrainian']; // and its options
		this.answerOptions = ['little', 'good', 'carrier']; // provide available answers

		// override name, so it is no longer writable
		this.name = ko.computed(function () {
			return 'Your knowledge of ' + this.language();
		}, this);

		this.toString = function() {
			return ko.toJSON(this);
		};

		this.init.apply(this, arguments); // apply given options
	}
	QuestionLanguageModel.prototype = new QuestionBaseModel(); // inherit from base model

	function List() {
		var self = this;

		self.items = ko.observableArray([
			new QuestionExperienceModel({id:1, name:'Your experience in JS', importance:'Important', answer:'2+ years'}),
			new QuestionLanguageModel({id:2, importance:'Semi important', answer:'good', language:'Ukrainian'})
		]);
	}

	ko.applyBindings(new List());
</script>
</body>
</html>
```
