---
layout: post
title: Human readable time ago

tags: [ago, date, datetime, format, plural, singular, time]
---

```php
<?php

echo ago(time() - 3452); // will return: 58 минут назад

function ago($timestamp) {
    $difference = time() - $timestamp;
    $periods = array(
        array('секунду', 'секурнды', 'секунд'),
        array('минуту', 'минуты', 'минут'),
        array('час', 'часа', 'часов'),
        array('день', 'дня', 'дней'),
        array('неделю', 'недели', 'недель'),
        array('месяц', 'месяца', 'месяцев'),
        array('год', 'года', 'лет'),
        array('десятилетие', 'десятилетий', 'десятилетий'),
    );

    $lengths = array('60','60','24','7','4.35','12','10');

    for($j = 0; $difference >= $lengths[$j]; $j++)
        $difference /= $lengths[$j];

    $difference = round($difference);

    $cases = array (2, 0, 1, 1, 1, 2);
    $text = $periods[$j][ ($difference%100>4 && $difference%100<20)? 2: $cases[min($difference%10, 5)] ];
    return $difference.' '.$text . ' назад';
}
```

Реализация на .net

```csharp
public static string timeAgo(DateTime date)
{
    if (date == DateTime.Now) return "только что";

    double difference = Math.Abs( (DateTime.Now - date).TotalSeconds );

    List<string[]> periods = new List<string[]>();
    periods.Add(new string[] {"секунду", "секурнды", "секунд" });
    periods.Add(new string[] {"минуту", "минуты", "минут" });
    periods.Add(new string[] {"час", "часа", "часов" });
    periods.Add(new string[] {"день", "дня", "дней" });
    periods.Add(new string[] {"неделю", "недели", "недель" });
    periods.Add(new string[] {"месяц", "месяца", "месяцев" });
    periods.Add(new string[] {"год", "года", "лет" });
    periods.Add(new string[] { "десятилетие", "десятилетий", "десятилетий" });

    double[] lengths = new double[] { 60, 60, 24, 7, 4.35, 12, 10 };
    int[] cases = new int[] { 2, 0, 1, 1, 1, 2 };

    int j = 0;
    for (j = 0; difference >= lengths[j]; j++)
    {
        difference = difference / lengths[j];
    }

    difference = Math.Round(difference);

    string text = periods[j][ ( difference%100 > 4 && difference%100 < 20 ) ? 2 : cases[(int)Math.Min(difference%10, 5)] ];

    string prefix = date > DateTime.Now ? "через " : string.Empty;
    string suffix = date < DateTime.Now ? " назад" : string.Empty;

    return string.Format("{0}{1} {2}{3}", prefix, difference, text, suffix);
}
```

Реализация на javascript

```js
function ago(date) {
    var difference = ((new Date()).getTime() - date.getTime()) / 1000;
    var periods = [
        ['секунду', 'секурнды', 'секунд'],
        ['минуту', 'минуты', 'минут'],
        ['час', 'часа', 'часов'],
        ['день', 'дня', 'дней'],
        ['неделю', 'недели', 'недель'],
        ['месяц', 'месяца', 'месяцев'],
        ['год', 'года', 'лет'],
        ['десятилетие', 'десятилетий', 'десятилетий']
    ];

    var lengths = [60, 60, 24, 7, 4.35, 12, 10];

    for(var i = 0; difference >= lengths[i]; i++) {
        difference = difference / lengths[i];
    }

    difference = Math.round(difference);

    var cases = [2, 0, 1, 1, 1, 2];
    var text = periods[i][(difference%100>4 && difference%100<20)? 2: cases[Math.min(difference%10, 5)]];
    return difference + ' ' + text + ' назад';
}
```
