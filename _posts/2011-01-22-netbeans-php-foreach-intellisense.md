---
layout: post
title: Netbeans php foreach intellisense

tags: [autosuggest, foreach, intellisense, netbeans, php, zend]
---

For intellisense auto suggestion helper work in php projects in netbeans code must be like this one:

```php
<?php /* @var $this Zend_View */?>
<h3>List</h3>

<ol>
<?php foreach($this->entries as $entry): /* @var $entry Application_Model_Rtest */?>
    <li>
        <?php echo $this->escape($entry->get_url())?>
    </li>
<?php endforeach?>
</ol>
```
