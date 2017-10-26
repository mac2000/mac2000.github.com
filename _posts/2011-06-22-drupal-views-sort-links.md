---
layout: post
title: Drupal views sort links

tags: [d6, drupal, hook_views_query_alter, order, sort, views]
---

Create any view page, for example i have one with url /service.

Create simple module wich will implement `hook__views_query_alter`, like this:

```php
function sync_company_prices_views_query_alter(&$view, &$query) {
    if($view->name == 'Services') {
        if (arg(1) == 'pfa') $query->orderby[0]='field_price_from_value ASC';
        if (arg(1) == 'pfd') $query->orderby[0]='field_price_from_value DESC';
        if (arg(1) == 'pta') $query->orderby[0]='field_price_to_value ASC';
        if (arg(1) == 'ptd') $query->orderby[0]='field_price_to_value DESC';
    }
}
```

Now anywhere in theme u can add links like:

```
<a href="/services/pfa">by price from ASC</a> |
<a href="/services/pfd">by price from DESC</a> |
<a href="/services/pta">by price to ASC</a> |
<a href="/services/ptd">by price to DESC</a>
```
