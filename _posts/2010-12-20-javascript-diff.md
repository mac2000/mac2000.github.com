---
layout: post
title: Javascript Diff

tags: [diff, javascript]
---

http://ejohn.org/projects/javascript-diff-algorithm/

Пример использования:

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml" >
<head>
<script type="text/javascript" src="jsdiff.js"></script>
</head>
<body>
<div id="res"></div>
<script type="text/javascript">
    document.getElementById('res').innerHTML = diffString(
        "The red brown\nfox jumped over the rolling log.",
        "The brown spotted fox leaped over the rolling log"
    );
</script>
</body>
</html>
```

Файлы: [jsdiff](/images/wp/jsdiff.zip)
