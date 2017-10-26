---
layout: post
title: Create Drupal Node from Wordpress via XMLRPC
tags: [drupal, wordpress, xmlrpc, services]
---

Here is sample `test.php` code that should be put into wordpress site root.

```php
<?php
require_once('./wp-load.php');
require_once('./wp-includes/class-IXR.php');

$hostname = 'calls.vcap.me';
$endpoint = 'xmlrpc'; // resources required: node.create, node.retrieve, user.login, user.logout, user.token
$username = 'admin';
$password = 'kG5iyslRF#x4';

#region normalize arguments
$hostname = explode('//', $hostname);
$hostname = array_pop($hostname);
$hostname = explode('/', $hostname);
$hostname = array_shift($hostname);
$hostname = strtolower($hostname);

$endpoint = trim($endpoint, '/ ');
#endregion

$client = new IXR_Client('http://' . $hostname . '/' . $endpoint);

#region Step 1: Login
$result = $client->query('user.login', $username, $password);
$response = $client->getResponse();
$client->headers['Cookie'] = $response['session_name'] . '=' . $response['sessid'];
dump('user.login', $result, $response);
/*
Array (
	[sessid] => xvzEAQwxTeejs0VYtSrsxYs56qXXD5c72AT598UuTwQ
	[session_name] => SESSe44223095fe7255df9836d59ea94ad7a
	[user] => Array (
			[uid] => 1
			[name] => admin
			[mail] => admin.calls@freelancewritingservices.info
			[theme] =>
			[signature] =>
			[signature_format] => plain_text
			[created] => 1381164873
			[access] => 1384681656
			[login] => 1384681783
			[status] => 1
			[timezone] => Europe/Kiev
			[language] =>
			[picture] =>
			[init] => marchenko.alexandr@gmail.com
			[data] => Array (
				[overlay] => 1
			)
			[roles] => Array (
				[2] => authenticated user
			)
	)
)
*/
#endregion

#region Step 2: Get token
$result = $client->query('user.token');
$response = $client->getResponse();
$client->headers['X-CSRF-Token'] = $response['token'];
dump('user.token', $result, $response);
/*
Array (
	[token] => u9xxg5T5pY5vgxP72m3aJBWwl-zedcrBy_LMHaRtX54
)
*/
#endregion

#region Step 4: Create node
$result = $client->query('node.create', array(
	'type' => 'customer_call',
	'status' => 1,

	//field_customer_name[und][0][value]
	'field_customer_name' => array(
		'und' => array(
			array('value' => 'Hello')
		)
	),

	//field_customer_email[und][0][email]
	'field_customer_email' => array(
		'und' => array(
			array('email' => 'xmlrpc@test.com')
		)
	),

	//field_site[und][0][url]
	'field_site' => array(
		'und' => array(
			array('url' => 'http://test.com')
		)
	),

	//field_phone[und][0][country_codes]
	//field_phone[und][0][number]
	'field_phone' => array(
		'und' => array(
			array(
				'country_codes' => 'ua',
				'number' => '443607885'
			)
		)
	),

	//field_skype[und][0][value]
	'field_skype' => array(
		'und' => array(
			array('value' => 'mac2000-skype')
		)
	),

	//field_gender[und]
	'field_gender' => array(
		'und' => array(
			1
		)
	),

	//field_niche[und]
	'field_niche' => array(
		'und' => array(
			6
		)
	),

	//field_status[und]
	'field_status' => array(
		'und' => array(
			9
		)
	),

	//field_outcome[und]
	'field_outcome' => array(
		'und' => array(
			'_none'
		)
	),

	//field_assignee[und]
	'field_assignee' => array(
		'und' => array(
			9
		)
	),

	//field_received_from[und]
	'field_received_from' => array(
		'und' => array(
			24
		)
	),

	//field_received_at[und][0][value][month]
	//field_received_at[und][0][value][day]
	//field_received_at[und][0][value][year]
	//field_received_at[und][0][value][hour]
	//field_received_at[und][0][value][minute]
	'field_received_at' => array(
		'und' => array(
			array(
				'value' => array(
					'month' => '11',
					'day' => '16',
					'year' => '2013',
					'hour' => '15',
					'minute' => '00'
				)
			)
		)
	),

	//field_comment[und][0][value]
	'field_comment' => array(
		'und' => array(
			array('value' => 'Hello World!')
		)
	),

	//field_order_total[und][0][value]
	'field_order_total' => array(
		'und' => array(
			array('value' => '9.99')
		)
	),

	//field_passed_by[und]
	'field_passed_by' => array(
		'und' => array(
			10
		)
	),
));
$response = $client->getResponse();
dump('node.create', $result, $response);
/*
Array (
	[nid] => 122
)
*/
#endregion

#region Step 5: Retrieve node
$result = $client->query('node.retrieve', array('nid' => $response['nid']));
$response = $client->getResponse();
dump('node.retrieve', $result, $response);
/*
Array (
	[vid] => 432
	[uid] => 9
	[title] => Hello - 11/17/2013 - 11:49
	[log] =>
	[status] => 1
	[comment] => 0
	[promote] => 0
	[sticky] => 0
	[nid] => 122
	[type] => customer_call
	[language] => und
	[created] => 1384681784
	[changed] => 1384681784
	[tnid] => 0
	[translate] => 0
	[revision_timestamp] => 1384681784
	[revision_uid] => 1
	[field_customer_name] => Array (
		[und] => Array (
			[0] => Array (
				[value] => Hello
				[format] =>
				[safe_value] => Hello
			)
		)
	)

	[field_customer_email] => Array (
		[und] => Array (
			[0] => Array (
				[email] => xmlrpc@test.com
			)
		)
	)

	[field_phone] => Array (
		[und] => Array (
			[0] => Array (
				[number] => 443607885
				[country_codes] => ua
				[extension] =>
			)
		)
	)

	[field_skype] => Array (
		[und] => Array (
			[0] => Array (
				[value] => mac2000-skype
				[format] =>
				[safe_value] => mac2000-skype
			)
		)
	)

	[field_gender] => Array (
		[und] => Array (
			[0] => Array (
				[tid] => 1
			)
		)
	)

	[field_niche] => Array (
		[und] => Array (
			[0] => Array (
				[tid] => 6
			)
		)
	)

	[field_status] => Array (
		[und] => Array (
			[0] => Array (
				[tid] => 9
			)
		)
	)

	[field_outcome] => Array ()

	[field_assignee] => Array (
		[und] => Array (
			[0] => Array (
				[target_id] => 9
			)
		)
	)

	[field_order_total] => Array (
		[und] => Array (
			[0] => Array (
				[value] => 9.99
			)
		)
	)

	[field_received_from] => Array (
		[und] => Array (
			[0] => Array (
				[tid] => 24
			)
		)
	)

	[field_received_at] => Array (
		[und] => Array (
			[0] => Array (
				[value] => 2013-11-16 13:00:00
				[timezone] => Europe/Kiev
				[timezone_db] => UTC
				[date_type] => datetime
			)
		)
	)

	[field_site] => Array (
		[und] => Array (
			[0] => Array (
				[url] => http://test.com
				[title] =>
				[attributes] => Array ()
			)
		)
	)

	[field_passed_by] => Array (
		[und] => Array (
			[0] => Array (
				[target_id] => 1
			)
		)
	)

	[field_comment] => Array (
		[und] => Array (
			[0] => Array (
				[value] => Hello World!
				[format] =>
				[safe_value] => Hello World!
			)
		)
	)

	[name] => caller
	[picture] => 0
	[data] => b:0;
	[path] => http://calls.vcap.me/node/122
)
*/
#endregion

#region Step 5: Logout (optional)
$result = $client->query('user.logout');
$response = $client->getResponse();
dump('logout', $result, $response);
#endregion

function dump()
{
	$args = func_get_args();
	$title = strtoupper(array_shift($args));
	echo "<h3>$title</h3>";
	foreach ($args as $arg) {
		echo var_dump($arg);
		//echo '<pre>' . print_r($arg, true) . '</pre>';
	}
	echo "<hr/>";
}
```