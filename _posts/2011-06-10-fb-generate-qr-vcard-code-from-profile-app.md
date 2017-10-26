---
layout: post
title: FB generate qr vcard code from profile app

tags: [facebook, fb]
---

Primary idea is to create qr code vcard from facebook profile.

http://ru.wikipedia.org/wiki/VCard

http://phpqrcode.sourceforge.net/

```php
<?php
require 'facebook-php-sdk/src/facebook.php';

$facebook = new Facebook(array(
    'appId'  => '192375534146916',
    'secret' => '6d13d68647565031ddee3dd2e21c47a7',
));

$user = $facebook->getUser();
?>
<!doctype html>
<html xmlns="http://www.w3.org/1999/xhtml" xmlns:fb="https://www.facebook.com/2008/fbml">
<head>
<title>qrcode</title>
<style>
body {font-family: 'Lucida Grande', Verdana, Arial, sans-serif;}
h1 a {text-decoration: none;color: #3b5998;}
h1 a:hover {text-decoration: underline;}

/* http://devtacular.com/articles/bkonrad/how-to-style-an-application-like-facebook */

/* Default Facebook CSS */
.fbbody
{
    font-family: "lucida grande" ,tahoma,verdana,arial,sans-serif;
    font-size: 11px;
    color: #333333;
}
/* Default Anchor Style */
.fbbody a
{
    color: #3b5998;
    outline-style: none;
    text-decoration: none;
    font-size: 11px;
    font-weight: bold;
}
.fbbody a:hover
{
    text-decoration: underline;
}
/* Facebook Box Styles */
.fbgreybox
{
    background-color: #f7f7f7;
    border: 1px solid #cccccc;
    color: #333333;
    padding: 10px;
    font-size: 13px;
    font-weight: bold;
}
.fbbluebox
{
    background-color: #eceff6;
    border: 1px solid #d4dae8;
    color: #333333;
    padding: 10px;
    font-size: 13px;
    font-weight: bold;
}
.fbinfobox
{
    background-color: #fff9d7;
    border: 1px solid #e2c822;
    color: #333333;
    padding: 10px;
    font-size: 13px;
    font-weight: bold;
}
.fberrorbox
{
    background-color: #ffebe8;
    border: 1px solid #dd3c10;
    color: #333333;
    padding: 10px;
    font-size: 13px;
    font-weight: bold;
}
/* Content Divider on White Background */
.fbcontentdivider
{
    margin-top: 15px;
    margin-bottom: 15px;
    width: 520px;
    height: 1px;
    background-color: #d8dfea;
}
/* Facebook Tab Style */
.fbtab
{
    padding: 8px;
    background-color: #d8dfea;
    color: #3b5998;
    font-weight: bold;
    float: left;
    margin-right: 4px;
    text-decoration: none;
}
.fbtab:hover
{
    background-color: #3b5998;
    color: #ffffff;
    cursor: hand;
}
table.form th {white-space:nowrap;}
table.form input.text {border:1px solid #bdc7d8;}
</style>
</head>
<body>
<div id="fb-root"></div>
<script src="http://connect.facebook.net/en_US/all.js"></script>
<script>
    FB.init({appId  : '192375534146916', status : true, cookie : true, xfbml  : true});
</script>

<script>
function show_intro() {
    document.getElementById('intro').style.display = 'block';
    document.getElementById('qr').style.display = 'none';
}
function show_qr() {
    document.getElementById('intro').style.display = 'none';
    document.getElementById('qr').style.display = 'block';
}
</script>

<script>
function getinfo() {
    FB.api('/me', function(response) {
        console.log(response);

        document.getElementById('birthday').value = response.birthday;
        document.getElementById('email').value = response.email;
        document.getElementById('first_name').value = response.first_name;
        document.getElementById('gender').value = response.gender;
        document.getElementById('last_name').value = response.last_name;
        document.getElementById('link').value = response.link;
        document.getElementById('name').value = response.name;
        document.getElementById('username').value = response.username;
        document.getElementById('website').value = response.website;
        document.getElementById('work').value = response.work[response.work.length - 1].employer.name;

        show_qr();
    });
}
</script>

<!-- LOGIN AND REQUEST PERMISSIONS -->
<script>
function getperms() {
    FB.login(function(response) {
        if (response.session) {
            if (response.perms) {
                getinfo();
            }
        }
    }, {perms:'user_about_me,user_birthday,user_hometown,user_location,user_website,user_work_history,email'});
}

FB.getLoginStatus(function(response) {
    if (response.status != 'connected') {
        show_intro();
        //getperms();
    } else {
        getinfo();
    }
});
</script>

<div id="intro" style="display:none">
<h3>QR CODE IS:</h3>
<p>Lorem ipsum dolor sit amet, consectetur adipiscing elit. Duis vel ornare nisi. In et nisi libero. Sed nec eros libero. Integer ut tellus arcu, vitae viverra nibh. Etiam eget risus eu purus tempor sed.</p>
<a href="javascript:void(0)" onclick="getperms()">give permissions</a>
<br />
<fb:login-button perms="user_about_me,user_birthday,user_hometown,user_location,user_website,user_work_history,email"></fb:login-button>
</div>

<div id="qr" style="display:none">

<div class="fbgreybox">
Generate QR code from your profile
</div>

<div class="fbbluebox">

<table class="form" cellpadding="0" cellspacing="0" border="0" width="100%">
<tr>
<th valign="top" align="right" width="10">
<label for="birthday">birthday:</label>
</th>
<td valign="top" align="left">
<input id="birthday" name="birthday" type="text" class="text" />
</td>
</tr>
</table>

<label for="email">email:</label><br />
<input id="email" name="email" type="text" /><br /><br />

<label for="first_name">first_name:</label><br />
<input id="first_name" name="first_name" type="text" /><br /><br />

<label for="gender">gender:</label><br />
<input id="gender" name="gender" type="text" /><br /><br />

<label for="last_name">last_name:</label><br />
<input id="last_name" name="last_name" type="text" /><br /><br />

<label for="link">link:</label><br />
<input id="link" name="link" type="text" /><br /><br />

<label for="name">name:</label><br />
<input id="name" name="name" type="text" /><br /><br />

<label for="username">username:</label><br />
<input id="username" name="username" type="text" /><br /><br />

<label for="website">website:</label><br />
<input id="website" name="website" type="text" /><br /><br />

<label for="work">work:</label><br />
<input id="work" name="work" type="text" /><br /><br />
</div>

</div>

</body>
</html>
```

After app starts it checks getLoginStatus to see if user is connected (means that user allowed app to access profile info) to app.

If no, app calling FB.login to get permissions to retrive profile info, and after successfull call fill form with user data.

User then may change data and submit it to qr code generator, that will form qr code vcard from submited data - not implemented.

Development was stopped after this:

http://imageshack.us/photo/my-images/13/77970240.png/

Facebook no longer allow getting user phone - so app is unusefull.

Are facebook and his users - paranoics?

Example of qr code generate:

```php
<?php
include "phpqrcode/qrlib.php";
QRcode::png('BEGIN:VCARD
VERSION:3.0
N:Александр;Марченко
FN:Марченко Александр
PHOTO;VALUE=uri:http://www.gravatar.com/avatar/5f3da2561611ab7c88eb919b3345d00c.png
BDAY:1985-03-11
NICKNAME:mac
EMAIL;TYPE=INTERNET:mac@example.com
X-ICQ:159342338
END:VCARD', 'test.png');
?>

<img src="test.png" />
```
