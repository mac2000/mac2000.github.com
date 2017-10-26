---
layout: post
title: Textarea tab

tags: [GUI, javascript, textarea, UI, tab]
---

Here is example of code that will allow neter tabs into textarea

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>Untitled Document</title>
<script type="text/javascript">
function setSelectionRange(input, selectionStart, selectionEnd) {
    if (input.setSelectionRange) {
    input.focus();
    input.setSelectionRange(selectionStart, selectionEnd);
    }
    else if (input.createTextRange) {
    var range = input.createTextRange();
    range.collapse(true);
    range.moveEnd('character', selectionEnd);
    range.moveStart('character', selectionStart);
    range.select();
    }
}

function replaceSelection (input, replaceString) {
    if (input.setSelectionRange) {
        var selectionStart = input.selectionStart;
        var selectionEnd = input.selectionEnd;
        input.value = input.value.substring(0, selectionStart)+ replaceString + input.value.substring(selectionEnd);

        if (selectionStart != selectionEnd){
            setSelectionRange(input, selectionStart, selectionStart +   replaceString.length);
        }else{
            setSelectionRange(input, selectionStart + replaceString.length, selectionStart + replaceString.length);
        }

    }else if (document.selection) {
        var range = document.selection.createRange();

        if (range.parentElement() == input) {
            var isCollapsed = range.text == '';
            range.text = replaceString;

                if (!isCollapsed)  {
                range.moveStart('character', -replaceString.length);
                range.select();
            }
        }
    }
}

// We are going to catch the TAB key so that we can use it, Hooray!
function catchTab(item,e){
    if(navigator.userAgent.match("Gecko")){
        c=e.which;
    }else{
        c=e.keyCode;
    }
    if(c==9){
        replaceSelection(item,String.fromCharCode(9));
        setTimeout("document.getElementById('"+item.id+"').focus();",0);
        return false;
    }

}
</script>
</head>
<body>
    <textarea name="data" id="data" rows="20" columns="35" wrap="off" onkeydown="return catchTab(this,event)" ></textarea>
</body>
</html>
```
