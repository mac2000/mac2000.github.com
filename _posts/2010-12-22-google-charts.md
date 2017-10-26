---
layout: post
title: Google charts

tags: [actionscript, chart, flash, google, javascript, online, service, tool]
---

http://code.google.com/intl/ru-RU/apis/visualization/documentation/gallery.html

Визуализация, данных, очень просто и главное быстро. На странице показаны уже готовые к применению примеры.

Простенький пример:

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>Untitled Document</title>
<style type="text/css">
html, body {padding:0;margin:0;height:100%;overflow:hidden;}
</style>
<script type='text/javascript' src='https://www.google.com/jsapi'></script>
    <script type='text/javascript'>
      google.load('visualization', '1', {'packages':['annotatedtimeline']});
      google.setOnLoadCallback(drawChart);
      function drawChart() {
        var data = new google.visualization.DataTable();
        data.addColumn('date', 'Date');
        data.addColumn('number', 'IE');
        data.addColumn('number', 'Opera');
        data.addColumn('number', 'Chrome');

        data.addRows([
          [new Date(2008, 1 ,1), 20, 23, 23],
          [new Date(2008, 1 ,2), 10, 10, 11],
          [new Date(2008, 1 ,3), 5, 5, 8],
          [new Date(2008, 1 ,4), 30, 33, 32],
          [new Date(2008, 1 ,5), 5, 6, 3],
          [new Date(2008, 1 ,6), 1, 1, 1]
        ]);

        var chart = new google.visualization.AnnotatedTimeLine(document.getElementById('chart_div'));
        chart.draw(data, {displayAnnotations: false, displayRangeSelector: false, displayZoomButtons: false});
      }

    </script>
</head>
<body>
  <div id='chart_div' style='width:100%;height:100%;'></div>
</body>
</html>
```

И как оно выглядит:

![screenshot](/images/wp/1.png)
