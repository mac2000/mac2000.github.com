---
layout: post
title: VML example

tags: [border-radius, corners, radius, roundcorder, vector, vml]
---

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>Corners</title>
<style type="text/css">
html, body {background:#ccc;font-family:Tahoma;}
.container {background:#fff;padding:10px;}

.container {
    -moz-border-radius: 10px;
    -webkit-border-radius: 10px;
    border-radius: 10px;
}
</style>

<!--[if lte IE 8]>
<style type="text/css">
v\:oval { behavior: url(#default#VML);
display:inline-block;
position:absolute;
left:0;
top:0;
width:20px;
height:20px;
}
v\:oval.tl {left:0;top:0;right:auto;bottom:auto;}
v\:oval.tr {left:-10px;top:0;right:auto;bottom:auto;}
v\:oval.bl {left:0;top:auto;right:auto;bottom:0;}
v\:oval.br {left:-10px;top:auto;right:auto;bottom:0;}

table.crn {margin:0;width:100%;border:none;background:none;}
table.crn tbody,
table.crn tr,
table.crn td {border:none;background:none;}
table.crn td {padding:0;font-size:1px;line-height:1px;height:10px;}
table.crn td div {width:10px;height:10px;overflow:hidden;position:relative;}

.container {padding-top:0;padding-bottom:0;}
</style>
<xml:namespace ns="urn:schemas-microsoft-com:vml" prefix="v"/>
<![endif]-->
</head>
<body>

<div style="width:500px;margin:5em auto;">

<!--[if lte IE 8]>
<table class="crn" width="100%" cellpadding="0" cellspacing="0" border="0">
<tbody>
<tr >
<td width="10">
<div><v:oval class="tl" fillcolor="white" stroked="false"/></div>
</td>
<td style="background:#fff;">&nbsp;</td>
<td width="10">
<div><v:oval class="tr" fillcolor="white" stroked="false"/></div>
</td>
</tr>
</tbody>
</table>
<![endif]-->
<div class="container">
        Hello world<br />
        Hello world<br />
        Hello world<br />
        Hello world<br />
        Hello world<br />
</div>

<!--[if lte IE 8]>
<table class="crn" width="100%" height="10" cellpadding="0" cellspacing="0" border="0">
<tbody>
<tr >
<td width="10" height="10">
<div><v:oval class="bl" fillcolor="white" stroked="false"/></div>
</td>
<td height="10" style="background:#fff;">&nbsp;</td>
<td width="10" height="10">
<div><v:oval class="br" fillcolor="white" stroked="false"/></div>
</td>
</tr>
</tbody>
</table>
<![endif]-->

</div>

</body>
</html>
```

Work in IE6,7,8.

http://msdn.microsoft.com/en-us/library/bb264307%28v=VS.85%29.aspx

To add namespace via js:

```js
<script type="text/javascript">
document.namespaces.add("v","urn:schemas-microsoft-com:vml","#default#VML");
</script>
```
