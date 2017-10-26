---
layout: post
title: YUI CSS Framework

tags: [css, framework, YUI]
---

CSS
---

```css
/*
Copyright (c) 2010, Yahoo! Inc. All rights reserved.
Code licensed under the BSD License:
http://developer.yahoo.com/yui/license.html
version: 3.1.0
build: 2026
*/
/*
    TODO will need to remove settings on HTML since we can't namespace it.
    TODO with the prefix, should I group by selector or property for weight savings?
*/
html{
    color:#000;
    background:#FFF;
}
/*
    TODO remove settings on BODY since we can't namespace it.
*/
/*
    TODO test putting a class on HEAD.
        - Fails on FF.
*/
body,
div,
dl,
dt,
dd,
ul,
ol,
li,
h1,
h2,
h3,
h4,
h5,
h6,
pre,
code,
form,
fieldset,
legend,
input,
textarea,
p,
blockquote,
th,
td {
    margin:0;
    padding:0;
}
table {
    border-collapse:collapse;
    border-spacing:0;
}
fieldset,
img {
    border:0;
}
/*
    TODO think about hanlding inheritence differently, maybe letting IE6 fail a bit...
*/
address,
caption,
cite,
code,
dfn,
em,
strong,
th,
var {
    font-style:normal;
    font-weight:normal;
}
/*
    TODO Figure out where this list-style rule is best set. Hedger has a request to investigate.
*/
li {
    list-style:none;
}

caption,
th {
    text-align:left;
}
h1,
h2,
h3,
h4,
h5,
h6 {
    font-size:100%;
    font-weight:normal;
}
q:before,
q:after {
    content:'';
}
abbr,
acronym {
    border:0;
    font-variant:normal;
}
/* to preserve line-height and selector appearance */
sup {
    vertical-align:text-top;
}
sub {
    vertical-align:text-bottom;
}
input,
textarea,
select {
    font-family:inherit;
    font-size:inherit;
    font-weight:inherit;
}
/*to enable resizing for IE*/
input,
textarea,
select {
    *font-size:100%;
}
/*because legend doesn't inherit in IE */
legend {
    color:#000;
}

/*
Copyright (c) 2010, Yahoo! Inc. All rights reserved.
Code licensed under the BSD License:
http://developer.yahoo.com/yui/license.html
version: 3.1.0
build: 2026
*/
/**
    * Percents could work for IE, but for backCompat purposes, we are using keywords.
    * x-small is for IE6/7 quirks mode.
    */
body {
    font:13px/1.231 arial,helvetica,clean,sans-serif;
    *font-size:small; /* for IE */
    *font:x-small; /* for IE in quirks mode */
}

/**
    * Nudge down to get to 13px equivalent for these form elements
    */
select,
input,
button,
textarea {
    font:99% arial,helvetica,clean,sans-serif;
}

/**
    * To help tables remember to inherit
    */
table {
    font-size:inherit;
    font:100%;
}

/**
    * Bump up IE to get to 13px equivalent for these fixed-width elements
    */
pre,
code,
kbd,
samp,
tt {
    font-family:monospace;
    *font-size:108%;
    line-height:100%;
}

/*
Copyright (c) 2010, Yahoo! Inc. All rights reserved.
Code licensed under the BSD License:
http://developer.yahoo.com/yui/license.html
version: 3.1.0
build: 2026
*/
/* base.css, part of YUI's CSS Foundation */
h1 {
    /*18px via YUI Fonts CSS foundation*/
    font-size:138.5%;
}
h2 {
    /*16px via YUI Fonts CSS foundation*/
    font-size:123.1%;
}
h3 {
    /*14px via YUI Fonts CSS foundation*/
    font-size:108%;
}
h1,h2,h3 {
    /* top & bottom margin based on font size */
    margin:1em 0;
}
h1,h2,h3,h4,h5,h6,strong {
    /*bringing boldness back to headers and the strong element*/
    font-weight:bold;
}
abbr,acronym {
    /*indicating to users that more info is available */
    border-bottom:1px dotted #000;
    cursor:help;
}
em {
    /*bringing italics back to the em element*/
    font-style:italic;
}
blockquote,ul,ol,dl {
    /*giving blockquotes and lists room to breath*/
    margin:1em;
}
ol,ul,dl {
    /*bringing lists on to the page with breathing room */
    margin-left:2em;
}
ol li {
    /*giving OL's LIs generated numbers*/
    list-style: decimal outside;
}
ul li {
    /*giving UL's LIs generated disc markers*/
    list-style: disc outside;
}
dl dd {
    /*providing spacing for definition terms*/
    margin-left:1em;
}
th,td {
    /*borders and padding to make the table readable*/
    border:1px solid #000;
    padding:.5em;
}
th {
    /*distinguishing table headers from data cells*/
    font-weight:bold;
    text-align:center;
}
caption {
    /*coordinated margin to match cell's padding*/
    margin-bottom:.5em;
    /*centered so it doesn't blend in to other content*/
    text-align:center;
}
p,fieldset,table,pre {
    /*so things don't run into each other*/
    margin-bottom:1em;
}
/* setting a consistent width, 160px;
    control of type=file still not possible */
input[type=text],input[type=password],textarea{width:12.25em;*width:11.9em;}
```

HTML
----

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>Untitled Document</title>
<link rel="stylesheet" href="reset.css" type="text/css">
<link rel="stylesheet" href="fonts.css" type="text/css">
<link rel="stylesheet" href="base.css" type="text/css">
<style type="text/css">
body {font-family:verdana;font-size:14px;color:#333;}
h1,h2,h3,h4,h5,h6 {color:#000}
</style>
</head>

<body>
    <h1>This is a H1 element.</h1>
    <h2>This is a H2 element.</h2>
    <h3>This is a H3 element.</h3>
    <h4>This is a H4 element.</h4>
    <h5>This is a H5 element.</h5>
    <h6>This is a H6 element.</h6>

    <ul>
        <li>This is a LI in a UL</li>
    </ul>

    <ol>
        <li>This is a LI in a UL</li>
    </ol>

    <dl>
        <dt>This is a DT in a DL</dt>
        <dd>This is a DD in a DL</dd>
    </dl>

    <form>
        <input value="This is an INPUT type TEXT in a FORM" type="text">

        <select>
            <option selected="selected">This is an OPTION in a SELECT</option>
            <option>This is an OPTION in a SELECT</option>
            <optgroup>
                <option>This is an OPTION in a OPTGROUP in a SELECT</option>
                <option>This is an OPTION in a OPTGROUP in a SELECT</option>
            </optgroup>
            <option>This is an OPTION in a SELECT</option>
        </select>

        <textarea name="ta1">This is text in a TEXTAREA in a FORM</textarea>

        <fieldset>
            <textarea name="ta2">This is text in a TEXTAREA in a FIELDSET in a
FORM</textarea>
        </fieldset>

        <button>This is a BUTTON</button>

    </form>

    <p>This paragraph contains a bunch of phrase elements. Up first in an <a href="http://developer.yahoo.com/yui/3/examples/cssbase/test">[A]nchor</a>, followed by an <abbr title="test">ABBR with a title value</abbr>, followed by an <acronym title="test">ACRONYM with a title value</acronym>, followed by an </p><address>ADDRESS</address>, followed by a <cite>CITE</cite> element, followed by a <code>CODE element, followed by a <del>DEL</del> element, followed by a <em>EM</em> element, followed by a <ins>INS</ins> element, followed by a <kbd>KBD</kbd> element, followed by a <q>Q</q> element, followed by a <samp>SAMP</samp> element, followed by a <span>SPAN</span> element, followed by a <strong>STRONG</strong> element, followed by a <tt>TT</tt> element, followed by a <var>VAR</var> element, all within a containing P.</p>

    <blockquote>This is a BLOCKQUOTE element.</blockquote>

    <table>
        <caption>This is a CAPTION in a TABLE</caption>
        <thead>
            <tr>
                <th>This is a TH in a TR in a THEAD in a TABLE</th>
                <td>This is a TD in a TR in a THEAD in a TABLE</td>
            </tr>
        </thead>
        <tfoot>
            <tr>
                <th>This is a TH in a TR in a TFOOT in a TABLE</th>
                <td>This is a TD in a TR in a TFOOT in a TABLE</td>
            </tr>
        </tfoot>
        <tbody>
            <tr>
                <th>This is a TH in a TR in a TBODY in a TABLE</th>
                <td>This is a TD in a TR in a TBODY in a TABLE</td>
            </tr>
        </tbody>
    </table>
</body>
</html>
```

Файлы:

http://developer.yahoo.com/yui/examples/base/base-simple_source.html

http://developer.yahoo.com/yui/build/reset/reset.css

http://developer.yahoo.com/yui/build/fonts/fonts.css

http://developer.yahoo.com/yui/build/base/base.css
