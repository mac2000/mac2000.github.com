---
layout: post
title: Drupal custom breadcrumbs

tags: [breadcrumbs, drupal, preprocess_page]
---

```php
function mydrive_preprocess_page(&$variables) {

    if($variables['node']->type != "") {
        $variables['template_files'][] = "page-node-" . $variables['node']->type;

        if($variables['node']->type == "autoschool") {

            $distinct = null;
            foreach($variables['node']->taxonomy as $term) {
                if($term->vid == "3") {
                    $distinct = $term;
                    break;
                }
            }

            $regions_terms = get_cached_regions_terms();

            $city = null;
            foreach($regions_terms as $term) {
                if($term->tid == $distinct->tid) {
                    $distinct = $term;
                    foreach($regions_terms as $term) {
                        if($term->tid == $distinct->parents[0]) {
                            $city = $term;
                            break;
                        }
                    }
                    break;
                }
            }

            $breadcrumbs = array();
            $breadcrumbs[] = '<a href="/">Главная</a>';
            $breadcrumbs[] = '<a href="/ca">Автошколы</a>';
            if($city) {
                $breadcrumbs[] = '<a href="/ca/'.$city->tid.'">'.$city->name.'</a>';
            }
            if($city && $distinct) {
                $breadcrumbs[] = '<a href="/ca/'.$city->tid.'/'.$distinct->tid.'">'.$distinct->name.'</a>';
            }
            $breadcrumbs[] = $variables['node']->title;

            $variables['breadcrumb'] = theme('breadcrumb', $breadcrumbs);

        }
    }

}
```
