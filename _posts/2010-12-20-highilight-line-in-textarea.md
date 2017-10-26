---
layout: post
title: Highilight line in textarea

tags: [javascript, textarea, editor]
---

Пример подсветки текущей строки в textarea

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>Untitled Document</title>
<script type="text/javascript" src="http://code.jquery.com/jquery-1.4.2.min.js"></script>
<!-- http://code.google.com/p/jcaret/ -->
<!--<script type="text/javascript" src="/images/wp/jquery.caret.1.02.min.js"></script>-->
<script type="text/javascript">
$(function(){
    $('#data').keyup(highlightTextArea).mouseup(highlightTextArea).scroll(highlightTextArea);
});

function highlightTextArea() {
    var m = ($.browser.msie || $.browser.opera) ? 2 : 1;
    var count = $('#data').caret().start;
    var lines = $('#data').val().split('\n');
    for(var i = 0; i < lines.length; i++) {
        count = count - lines[i].length - m;
        if(count < 0) break;
    }

    var lineHeight = parseInt($('#data').css('line-height'));
    if(isNaN(lineHeight)) lineHeight = 16;
    var top = i * lineHeight - $('#data').scrollTop();
    top = '0 ' + parseInt(top) + 'px';
    $('#data').css('background-position', top);
}
</script>
<style type="text/css">
#data {padding:0;border:1px solid #999;font:14px/16px Arial;background:#fff url(z.png) repeat-x 0 0}
</style>
</head>

<body>
<textarea name="data" id="data" rows="10" columns="50" wrap="off" >1. line
2. line
3. line
4. line
5. line
6. line
7. line
8. line
9. line
10. line
11. line
12. line
13. line
14. line
15. line</textarea>
</body>
</html>
```
