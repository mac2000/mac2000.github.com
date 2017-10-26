---
layout: post
title: Drupal theming

tags: [drupal, php]
---

[Guides and videos on Drupal 6 theming](http://drupal.org/node/587366)

As example, we have 2 content types: _page_ and _story_

Create new theme and define regions in its info file:

```
regions[left] = Left sidebar
regions[right] = Right sidebar
regions[content] = Content
regions[header] = Header
regions[footer] = Footer
regions[pageright] = Page right
regions[pagetop] = Page top
regions[pagebottom] = Page bottom
regions[pageleft] = Page left
regions[storyright] = Story right
regions[storytop] = Story top
regions[storybottom] = Story bottom
regions[storyleft] = Story left
```

Then in `template.php` add

```php
function THEMENAME_preprocess_node(&$variables) {
    if($variables['type'] == 'page') {
        $variables['pagetop'] = theme('blocks', 'pagetop');
    }
}
```

Then create files `node-page.tpl.php` and `node-story.tpl.php` and insert in them somethins like this:

```php
<?php if ($pagetop): ?>
<div style="background:#999;padding:1em;">
    <?php print $pagetop ?>
</div>
<?php endif; ?>
```

[Imagecache: dynamic image manipulation](http://drupal.org/node/163561)

[Theming CCK fields within a content type](http://drupal.org/node/807330)
