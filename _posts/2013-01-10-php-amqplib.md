---
layout: post
title: php-amqplib example
tags: [rabbitmq, php-amqplib]
---

This example shows two important things:

Job expirations - in my case job must be done today or newer.
Max retries - in my case maximum 3 retries allowed.

Notice that expiration - is milliseconds before job expires and that it is a string not integer.

Also notice how retries accomplished via priority. The problem is that there is cases when the same job may be sended over and over to worker that can not do it, with new priority we are moving job to end of queue and there is much more chances that rabbit will send it to another worker.

composer.json
-------------

```json
{
	"require": {
		"videlalvaro/php-amqplib": "dev-master"
	}
}
```

echo_send_mass.php
------------------

```php
<?php
require 'vendor/autoload.php';
$queue = 'echo';
$connection = new \PhpAmqpLib\Connection\AMQPConnection('localhost', 5672, 'guest', 'guest');
$channel = $connection->channel();
$channel->queue_declare($queue, false, true, false, false);

$msgs = array(
	'Hello',
	'World',
	'mac',
	'was',
	'here',
	'lorem',
	'ipsum'
);
shuffle($msgs);

foreach($msgs as $data) {
	$msg = new \PhpAmqpLib\Message\AMQPMessage($data, array(
		'delivery_mode' => 2,
		'priority' => 1,
		'timestamp' => time(),
		'expiration' => strval(1000 * (strtotime('+1 day midnight') - time() - 1))
	));
	$channel->basic_publish($msg, '', $queue);
	echo ' [>] "' . $data . '" sent' . PHP_EOL;
}
$channel->close();
$connection->close();
```

echo_worker.php
---------------

```php
<?php
require 'vendor/autoload.php';
$queue = 'echo';
$connection = new \PhpAmqpLib\Connection\AMQPConnection('localhost', 5672, 'guest', 'guest');
$channel = $connection->channel();
$channel->queue_declare($queue, false, true, false, false);

global $stop_words;
$stop_words = array_slice($argv, 1);

echo ' [*] Waiting for messages. To exit press CTRL+C' . PHP_EOL;

function callback($msg)
{
	global $stop_words;

	try {
		echo ' [x] Received ' . $msg->body . ' (try: ' . $msg->get('priority') . ')' . PHP_EOL;
		if ($msg->get('priority') > 3) {
			$msg->delivery_info['channel']->basic_ack($msg->delivery_info['delivery_tag']);
			echo ' [!] Maximum retries reached at ' . $msg->get('priority') . ' retries' . PHP_EOL;
		} else {
			if (in_array($msg->body, $stop_words)) throw new Exception('Stop word detected');
			sleep(strlen($msg->body));
			$msg->delivery_info['channel']->basic_ack($msg->delivery_info['delivery_tag']);
			echo ' [+] Done' . PHP_EOL . PHP_EOL;

		}
	} catch (Exception $ex) {
		$channel = $msg->get('channel');
		$queue = $msg->delivery_info['routing_key'];
		$new_msg = new \PhpAmqpLib\Message\AMQPMessage($msg->body, array(
			'delivery_mode' => 2,
			'priority' => 1 + $msg->get('priority'),
			'timestamp' => time(),
			'expiration' => strval(1000 * (strtotime('+1 day midnight') - time() - 1))
		));
		$channel->basic_publish($new_msg, '', $queue);

		$msg->delivery_info['channel']->basic_ack($msg->delivery_info['delivery_tag']);
		echo ' [!] ERROR: ' . $ex->getMessage() . PHP_EOL . PHP_EOL;
	}
};

$channel->basic_qos(null, 1, null);
$channel->basic_consume($queue, '', false, false, false, false, 'callback');

function shutdown($channel, $connection)
{
	$channel->close();
	$connection->close();
}

register_shutdown_function('shutdown', $channel, $connection);

while (count($channel->callbacks)) {
	$channel->wait();
}
```

Usage example
-------------

```sh
$ php echo_send_mass.php
	[>] "World" sent
	[>] "was" sent
	[>] "mac" sent
	[>] "Hello" sent
	[>] "ipsum" sent
	[>] "here" sent
	[>] "lorem" sent
```

```sh
$ php echo_worker.php mac Hello
	[*] Waiting for messages. To exit press CTRL+C
	[x] Received World (try: 1)
	[+] Done

	[x] Received was (try: 1)
	[+] Done

	[x] Received mac (try: 1)
	[!] ERROR: Stop word detected

	[x] Received Hello (try: 1)
	[!] ERROR: Stop word detected

	[x] Received ipsum (try: 1)
	[+] Done

	[x] Received here (try: 1)
	[+] Done

	[x] Received lorem (try: 1)
	[+] Done

	[x] Received mac (try: 2)
	[!] ERROR: Stop word detected

	[x] Received Hello (try: 2)
	[!] ERROR: Stop word detected

	[x] Received mac (try: 3)
	[!] ERROR: Stop word detected

	[x] Received Hello (try: 3)
	[!] ERROR: Stop word detected

	[x] Received mac (try: 4)
	[!] Maximum retries reached at 4 retries
	[x] Received Hello (try: 4)
	[!] Maximum retries reached at 4 retries
```