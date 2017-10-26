---
layout: post
title: PHP ldap Active Directory auth
tags: [php, ldap, ad, auth]
---

Here is simplest ever way to auth with your local AD.

```php
<?php
function auth($username, $password, $domain = 'RABOTA', $endpoint = 'ldap://rabota.local', $dc = 'dc=rabota,dc=local') {
	$ldap = @ldap_connect($endpoint);
	if(!$ldap) return false;

	ldap_set_option($ldap, LDAP_OPT_PROTOCOL_VERSION, 3);
	ldap_set_option($ldap, LDAP_OPT_REFERRALS, 0);

	$bind = @ldap_bind($ldap, "$domain\\$username", $password);
	if(!$bind) return false;

	$result = @ldap_search($ldap, $dc, "(sAMAccountName=$username)");
	if(!$result) return false;

	@ldap_sort($ldap, $result, 'sn');
	$info = @ldap_get_entries($ldap, $result);
	if(!$info) return false;
	if(!isset($info['count']) || $info['count'] !== 1) return false;

	$data = [];

	foreach($info[0] as $key => $value) {
		if(is_numeric($key)) continue;
		if($key === 'count') continue;

		$data[$key] = (array)$value;
		unset($data[$key]['count']);
	}

	return [
		'mail' => $data['mail'][0],
		'displayname' => $data['displayname'][0]
	];
}
?><!DOCTYPE html>
<html>
<head>
	<meta charset="UTF-8">
	<meta name="viewport" content="width=device-width,initial-scale=1,user-scalable=no">
	<title>AD</title>
	<link rel="stylesheet" href="http://maxcdn.bootstrapcdn.com/bootstrap/latest/css/bootstrap.css">
	<style>
		form {max-width: 300px;margin:auto}
		input {margin-bottom:10px}
	</style>
</head>
<body>
	<div class="container">
		<h1 class="text-center">Active Directory</h1>
		<?php if(empty($_POST['username']) || empty($_POST['password'])) { ?>
		<form method="POST">
			<input type="text" name="username" placeholder="username" class="form-control" required>
			<input type="password" name="password" placeholder="password" class="form-control" required>
			<input type="submit" class="btn btn-default btn-block" value="Login">
		</form>
		<?php } else {
			$info = auth($_POST['username'], $_POST['password']);


			if(!$info) echo '<div class="alert alert-danger text-center">Login failed</div>';
			else echo '<div class="alert alert-success text-center">Login success</div><h1 class="text-center"><a href="mailto:' . $info['mail'] . '">' . $info['displayname'] . '</a></h1>';
		}
		?>
	</div>
</body>
</html>
```

Take a note to make secure requests you need:

1. Ensure that php_openssl.dll extension is enabled
2. Put `TLS_REQCERT never` into `C:\openldap\sysconf\ldap.conf`
3. Change endpoint schema from `ldap://` to `ldaps://`

Thanks to: https://www.novell.com/coolsolutions/tip/5838.html and many other tutorials found on the net.

And here is main part - integrate you local intranet Wordpress site with Active Directory:

```php
<?php
/*
Plugin Name: Active Directory
Plugin URI: http://mac-blog.org.ua/php-ldap-active-directory-auth-sample/
Description: This is sample plugin demonstrating how could you integrate with your corporate AD. Main idea is taken from: Active Directory Integration Plugin which already does the job for you.
Author: Marchenko Alexandr <marchenko.alexandr@gmail.com>
Version: 1.0
Author URI: http://mac-blog.org.ua/
*/

function ad_authenticate($user = NULL, $username = '', $password = '') {
	$user = get_user_by('login', $username);

	if (is_object($user) && ($user->ID == 1)) return false; // Do nothing for admin user


	$domain = 'RABOTA';
	$endpoint = 'ldap://rabota.local';
	$dc = 'dc=rabota,dc=local';


	$ldap = @ldap_connect($endpoint);
	if(!$ldap) return false;

	ldap_set_option($ldap, LDAP_OPT_PROTOCOL_VERSION, 3);
	ldap_set_option($ldap, LDAP_OPT_REFERRALS, 0);

	$bind = @ldap_bind($ldap, "$domain\\$username", $password);
	if(!$bind) return false;

	$result = @ldap_search($ldap, $dc, "(sAMAccountName=$username)");
	if(!$result) return false;

	@ldap_sort($ldap, $result, 'sn');
	$info = @ldap_get_entries($ldap, $result);
	if(!$info) return false;
	if(!isset($info['count']) || $info['count'] !== 1) return false;

	$data = [];

	foreach($info[0] as $key => $value) {
		if(is_numeric($key)) continue;
		if($key === 'count') continue;

		$data[$key] = (array)$value;
		unset($data[$key]['count']);
	}

	$data = [
		'mail' => $data['mail'][0],
		'displayname' => $data['displayname'][0]
	];

	// If we got here - then user can be logged in

	$user = get_user_by('login', $username);
	if(!$user) {
		$user_id = wp_create_user($username, $password, $data['mail']);

		wp_update_user([
			'ID' => $user_id,
			'nickname' => $data['displayname']
		]);

		$user = new WP_User($user_id);
		$user->set_role('editor');
	}
}
add_filter('authenticate', 'ad_authenticate', 10, 3);
```
