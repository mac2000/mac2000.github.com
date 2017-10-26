---
layout: post
title: Drupal h1, title, meta

tags: [drupal, h1, meta, preprocess_page, seo, title]
---

To add custom automaticaly generated titles, h1 and metatags u need modify `phptemplate_preprocess_page` in t`emplate.php` like this:

```php
function phptemplate_preprocess_page(&$vars) {
    $path = drupal_get_path_alias($_GET['q']);
    if(strpos($path, 'category/') === 0) {
        $path = explode('/', $path);
        if(isset($path[1]) && !empty($path[1])) {
            $taxonomy_name = $path[1];
            $taxonomy_name = strtolower(preg_replace('/[^a-zA-Z0-9_-]+/', '-', $taxonomy_name));
            $taxonomy_name = 'taxonomy-' . $taxonomy_name;
            $vars['body_classes'] .= ' ' . $taxonomy_name;
        }
    }

    set_title_tag($vars);
    set_h1_tag($vars);
    set_meta_keywords($vars);
    set_meta_description($vars);

    set_add_link($vars);
}

function set_add_link(&$vars) {
    global $user;

    if($user->uid == 0) {
        $vars['add_link'] = '/user/register';
    } else {

        $vars['add_link'] = '/node/add';

        if (!empty($vars['node']))  $vars['add_link'] = '/node/add/' . $vars['node']->type;

        //for pages
        $path = drupal_get_path_alias($_GET['q']);
        if($path == 'node')     $vars['add_link'] = '/node/add';
        if($path == 'games')    $vars['add_link'] = '/node/add/game';
        if($path == 'books')    $vars['add_link'] = '/node/add/book';
        if($path == 'services') $vars['add_link'] = '/node/add/service';

        if(strpos($path, 'category/kategorii-igr') === 0)       $vars['add_link'] = '/node/add/game';
        if(strpos($path, 'category/igrovye-platformy') === 0)   $vars['add_link'] = '/node/add/game';
        if(strpos($path, 'category/kategorii-knig') === 0)      $vars['add_link'] = '/node/add/book';
        if(strpos($path, 'category/kategorii-servisov') === 0)  $vars['add_link'] = '/node/add/service';

    }
}

function set_title_tag(&$vars) {
    $path = drupal_get_path_alias($_GET['q']);

    if (!empty($vars['node'])) {
        if($vars['node']->type == 'game')       $vars['head_title'] = $vars['title'] . ' описание, отзывы, скачать бесплатно. Оставь свое мнение.';
        if($vars['node']->type == 'book')       $vars['head_title'] = $vars['title'] . ' описание, отзывы, скачать бесплатно. Оставь свое мнение.';
        if($vars['node']->type == 'service')    $vars['head_title'] = $vars['title'] . ' описание, отзывы. Оставь свое мнение.';
    }

    //for pages
    if($path == 'node')     $vars['head_title'] = "Игры, Книги и Web-сервисы. Узнай об интересном на in-opinion.ru. | in-opinion";
    if($path == 'games')    $vars['head_title'] = "Игры – отзывы, описание, скачать бесплатно популярные игры.";
    if($path == 'books')    $vars['head_title'] = "Книги – отзывы, описание, скачать бесплатно популярные книги.";
    if($path == 'services') $vars['head_title'] = "Сервисы – отзывы, описание, популярные сервисы.";

    if(strpos($path, 'category/kategorii-igr') === 0) $vars['head_title'] = 'Игры '.$vars['title'].'. Популярные игры жанра '.$vars['title'];
    if(strpos($path, 'category/igrovye-platformy') === 0) $vars['head_title'] = 'Игры '.$vars['title'].'. Популярные игры жанра '.$vars['title'];
    if(strpos($path, 'category/kategorii-knig') === 0) $vars['head_title'] = 'Книги '.$vars['title'].'. Популярные книги жанра '.$vars['title'];
    if(strpos($path, 'category/kategorii-servisov') === 0) $vars['head_title'] = 'Сервисы '.$vars['title'].'. Популярные сервисы рубрики '.$vars['title'];
}

function set_meta_keywords(&$vars) {
    $path = drupal_get_path_alias($_GET['q']);

    if (!empty($vars['node'])) $vars['meta_keywords'] = $vars['title'];

    //for taxonomy pages
    if ($vars['template_files'][0] == 'page-taxonomy') $vars['meta_keywords'] = $vars['title'];

    //for pages
    if($path == 'node')     $vars['meta_keywords'] = "Игры, книги, web-сервисы";
    if($path == 'games')    $vars['meta_keywords'] = "Игры, игры скачать, игры отзывы, популярные игры, скачать игры бесплатно, описание игр";
    if($path == 'books')    $vars['meta_keywords'] = "Книги, книги скачать, книги отзывы, популярные книги, скачать книги бесплатно, описание книг";
    if($path == 'services') $vars['meta_keywords'] = "Сервисы, сервисы отзывы, популярные сервисы, описание сервисов";

    $vars['meta_keywords'] = empty($vars['meta_keywords']) ? '' : '<meta name="Keywords" content="' . $vars['meta_keywords'] . '">';
}

function set_meta_description(&$vars) {
    $path = drupal_get_path_alias($_GET['q']);

    if (!empty($vars['node'])) {
        $vars['meta_description'] = mb_substr(strip_tags($vars['node']->field_opinion[0]['value']), 0, 200, 'UTF-8');
    }

    //for taxonomy pages
    if ($vars['template_files'][0] == 'page-taxonomy') $vars['meta_description'] = '';

    //for pages
    if($path == 'node')     $vars['meta_description'] = "in-opinion это коллективный блог, где собраны все интересные игры, книги и web-сервисы с описанием и мнениями других пользователей. Вы можете добавить интересные вам книги, игры, сервисы, или оставить свое мнение об уже опубликованных.";
    if($path == 'games')    $vars['meta_description'] = "Описание и отзывы о популярных играх всех жаров. Оставь свое мнение. Ссылки на бесплатное скачивание игр.";
    if($path == 'books')    $vars['meta_description'] = "Описание и отзывы о популярных книгах всех жаров. Оставь свое мнение. Ссылки на бесплатное скачивание книг.";
    if($path == 'services') $vars['meta_description'] = "Описание и отзывы о популярных сервисах всех типов. Оставь свое мнение. Ссылки на сервисы.";

    $vars['meta_description'] = empty($vars['meta_description']) ? '' : '<meta name="Description" content="' . $vars['meta_description'] . '">';
}

function set_h1_tag(&$vars) {
    $path = drupal_get_path_alias($_GET['q']);
    $title = $vars['title'];

    if (!empty($vars['node'])) {
        if($vars['node']->type == 'game')       $vars['title'] = "$title описание и мнение об игре $title.";
        if($vars['node']->type == 'book')       $vars['title'] = "$title описание и мнение об книге $title.";
        if($vars['node']->type == 'service')    $vars['title'] = "$title описание и мнение об сервисе $title.";
    }

    //H1 for taxonomy pages
    if ($vars['template_files'][0] == 'page-taxonomy') $vars['title'] = $vars['title'];

    //H1 for pages
    if($path == 'node')     $vars['title'] = "Игры, Книги и Web-сервисы. Узнай об интересном на in-opinion.ru.";
    if($path == 'games')    $vars['title'] = "Описание и отзывы о популярных играх.";
    if($path == 'books')    $vars['title'] = "Описание и отзывы о популярных книгах.";
    if($path == 'services') $vars['title'] = "Описание и отзывы о популярных сервисах.";
}
```

Also there is example adding custom links to theme. So now anywhere in theme u can call it like:

```php
<div style="font-size:11px;">Книги, игры и веб-сервисы — <br /><a href="<?php print $add_link?>">расскажите</a> об интересном</div>
```
