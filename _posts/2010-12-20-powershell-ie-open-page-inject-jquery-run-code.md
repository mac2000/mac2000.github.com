---
layout: post
title: PowerShell IE open page, inject jQuery, run code

tags: [cmd, powershell, script, shell, ie, internetexplorer, inject, jquery, com, navigate2]
---

```powershell
# Create instance of InternetExplorer
$ie = new-object -com internetexplorer.application
$ie.visible = $true

# Go to page
$ie.navigate2("http://example.com/")
# Wait for page load
while($ie.busy) {start-sleep 1}

# Inject jQuery
$ie.Document.body.innerHTML = $ie.Document.body.innerHTML + "<div id=`"a`" onclick=`"var th = document.getElementsByTagName('body')[0];var s = document.createElement('script');s.setAttribute('type','text/javascript');s.setAttribute('src','http://code.jquery.com/jquery-1.4.2.min.js');th.appendChild(s);`">hello</div>"
$ie.Document.getElementById("a").click()

# Run our javascript
$ie.Document.body.innerHTML = $ie.Document.body.innerHTML + "<div id=`"x`" onclick=`"`$('a').hide()`">hello</div>"
$ie.Document.getElementById("x").click()
```
