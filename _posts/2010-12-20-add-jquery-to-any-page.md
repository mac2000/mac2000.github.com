---
layout: post
title: Add jQuery to any page

tags: [firefox, javascript, jquery, plugin]
---

This code will add jQuery to page, wait for it loads, and then run your code

Example:

```js
var done = false;
var head = document.getElementsByTagName('head')[0];
var script=document.createElement('script');
script.src='http://code.jquery.com/jquery-latest.min.js';
script.onload=script.onreadystatechange=function(){
    if(!done && (!this.readyState || this.readyState=='loaded' || this.readyState=='complete')){
        done = true;
        $=jQuery.noConflict();
        inject_and_execute_jquery_code();
        script.onload=script.onreadystatechange=null;
        head.removeChild(script);
    }
};
head.appendChild(script);
function inject_and_execute_jquery_code() {
    $('p').hide();
}
```

Create in some html link like this:

```html
<a href="javascript:%20(function(){var%20el=document.createElement('div'),b=document.getElementsByTagName('body')[0];otherlib=false,msg='';el.style.position='fixed';el.style.height='32px';el.style.width='220px';el.style.marginLeft='-110px';el.style.top='0';el.style.left='50%';el.style.padding='5px%2010px';el.style.zIndex=1001;el.style.fontSize='12px';el.style.color='#222';el.style.backgroundColor='#f99';if(typeof%20jQuery!='undefined'){msg='This%20page%20already%20using%20jQuery%20v'+jQuery.fn.jquery;return%20showMsg();}else%20if(typeof%20$=='function'){otherlib=true;}%20function%20getScript(url,success){var%20script=document.createElement('script');script.src=url;var%20head=document.getElementsByTagName('head')[0],done=false;script.onload=script.onreadystatechange=function(){if(!done&#038;&(!this.readyState||this.readyState=='loaded'||this.readyState=='complete')){done=true;success();script.onload=script.onreadystatechange=null;head.removeChild(script);}};head.appendChild(script);}%20getScript('http://code.jquery.com/jquery-latest.min.js',function(){if(typeof%20jQuery=='undefined'){msg='Sorry,%20but%20jQuery%20wasn\'t%20able%20to%20load';}else{msg='This%20page%20is%20now%20jQuerified%20with%20v'+jQuery.fn.jquery;if(otherlib){msg+='%20and%20noConflict().%20Use%20$jq(),%20not%20$().';}}%20return%20showMsg();});function%20showMsg(){el.innerHTML=msg;b.appendChild(el);window.setTimeout(function(){if(typeof%20jQuery=='undefined'){b.removeChild(el);}else{jQuery(el).fadeOut('slow',function(){jQuery(this).remove();});if(otherlib){$jq=jQuery.noConflict();}}},2500);}})();">jQuerify</a>
```

And drag it to your bookmarks

Then on any page click on bookmark and you will the notification tool tip that jquery successfully added to page
