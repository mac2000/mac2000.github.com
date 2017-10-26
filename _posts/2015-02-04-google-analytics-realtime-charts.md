---
layout: post
title: Google Analytics RealTime Charts
tags: [ga, analytics, realtime, chart]
---

Here is way you can retrieve Google Analytics Realtime report charts:

```js
var casper = require('casper').create({
	viewportSize: {
		width: 1440,
		height: 800
	}
});

casper.start('http://google.com.ua/analytics');

// Click on Sig In button
casper.then(function(){
	this.click('a.secondary-button');
});

// Fill & submit login form
casper.thenEvaluate(function() {
	document.querySelector('input[name="Email"]').setAttribute('value', 'YOUR_USER_NAME_HERE@gmail.com');
	document.querySelector('input[name="Passwd"]').setAttribute('value', '****************');
	document.querySelector('form').submit();
});

// Got second screen with question to confirm second email
//casper.thenEvaluate(function() {
//    if(document.querySelector('input[name="emailAnswer"]')) {
//        document.querySelector('input[name="emailAnswer"]').setAttribute('value', 'SECOND_EMAIL@gmail.com');
//        document.querySelector('form').submit();
//    }
//});

// Navigate to Realtime report
casper.thenOpen('https://www.google.com/analytics/web/?hl=ru&pli=1#realtime/rt-overview/YOUR_ID_HERE/');

// Wait for data & save screenshots
casper.waitFor(function check() {
	return this.evaluate(function() {
		return document.querySelectorAll('#ID-overviewPanelRequestUriTable tr').length > 3 && document.querySelectorAll('#ID-overviewPanelTrafficSourceValueOrganicTable tr').length > 3;
	});
}, function then() {
	this.wait(1000, function() {
		this.captureSelector('C:/inetpub/img1/anal/minute.png', '#ID-minuteChart');
		this.captureSelector('C:/inetpub/img1/anal/second.png', '#ID-secondChart');
	});
}, function timeout() {
	this.echo("Something wrong").exit();
});

casper.run();
```

Script uses headless webkit to navigate to relatime report and takes screenshots from charts, and saves them to given location

Required software: [phantomjs](http://phantomjs.org/), [casperjs](http://casperjs.org/)

Both are portables so you do not need to install anything.

In our case we are running this script each minute to save fresh chart images, which then used in alert mails if something is wrong to determine is alert false or not.

Here is how it looks:

![Example](/images/posts/anal.png)