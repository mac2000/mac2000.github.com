---
layout: post
title: php normal gaussian distribution min max

tags: [algorithm, gaus, gauss, generator, php, rand, random, rnd]
---

![screenshot](/images/wp/rnd_vb_gauss.png)

```php
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>Gauss test</title>
<script type="text/javascript" src="https://www.google.com/jsapi"></script>
<script type="text/javascript">
google.load("visualization", "1", {packages:["corechart"]});
google.setOnLoadCallback(drawChart);
function drawChart() {
    var data = new google.visualization.DataTable();
    data.addColumn('string', 'Iteration');
    data.addColumn('number', 'Random');
    data.addColumn('number', 'Gauss');
    data.addRows([
        <?php genrows()?>
    ]);

    var chart = new google.visualization.AreaChart(
        document.getElementById('chart')
    );

    chart.draw(
        data,
        {
            width: 600,
            height: 400,
            title: 'Rand vs Gauss'
    });
}
</script>
</head>

<body>
    <div id="chart"></div>
</body>
</html>
<?php

function gauss($min = 0, $max = 1) {
    $mean = ($max - $min) / 2;
    $std_dev = 1;

    $x=(float)rand()/(float)getrandmax();
    $y=(float)rand()/(float)getrandmax();

    $u=sqrt(-2*log($x))*cos(2*pi()*$y);

    return $u*$std_dev+$mean;
}

function genrows() {
    $rows = array();
    for($i = 1; $i <= 10; $i++) {
        $r = rand(1,10);
        $g = round(gauss(1,10));
        $rows[] = "['$i', $r, $g]";
    }
    echo implode(',', $rows);
}
```
