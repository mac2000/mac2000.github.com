---
layout: post
title: RabbitMQ python example with expire and reject
tags: [rabbitmq, reject, expire]
---

Based on: http://www.rabbitmq.com/tutorials/tutorial-two-python.html

This example demonstrates how to reject jobs that can not be done at moment on current worker, handling retries count and job expiration

send.py
-------

```py
#!/usr/bin/env python
import pika
import time
import datetime
import json
import sys

count = int(sys.argv[1]) # read from command line arguments count of jobs to create
queue = 'retries' # queue name

''' example of more robust pika.ConnectionParameters
host='localhost',
port=5672,
virtual_host='/',
credentials=pika.credentials.PlainCredentials(
	username='guest',
	password='guest'
)
'''
connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()
channel.queue_declare(queue=queue, durable=True) # durable=True - makes queue persistent

for i in range(1, count + 1):
	message = "item %d" % i
	timestamp = time.time()
	now = datetime.datetime.now()
	expire = 1000 * int((now.replace(hour=23, minute=59, second=59, microsecond=999999) - now).total_seconds())
	headers = { # example how headers can be used
		'hello': 'world',
		'created': int(timestamp)
	}
	data = { # example hot to transfer objects rather than string using json.dumps and json.loads
		'keyword': message,
		'domain': message,
		'created': int(timestamp),
		'expire': expire
	}
	channel.basic_publish(
		exchange='',
		routing_key=queue,
		body=json.dumps(data), # must be string
		properties=pika.BasicProperties(
			delivery_mode=2, # makes persistent job
			priority=0, # default priority
			timestamp=timestamp, # timestamp of job creation
			expiration=str(expire), # job expiration (milliseconds from now), must be string, handled by rabbitmq
			headers=headers
		))
	print "[>] Sent %r" % message

connection.close()
```

worker.py
---------

```py
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import pika
import time
import datetime
import json
import sys

stop_word = sys.argv[1]
max_retries = 3
queue = 'retries'

connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()
channel.queue_declare(queue=queue, durable=True)

print '[*] Waiting for messages. To exit press CTRL+C'

def callback(ch, method, properties, body):
	#print properties.headers.get('hello')
	data = json.loads(body)
	print "[>] Received '%s' (try: %d)" % (data.get('keyword'), 1 + int(properties.priority))

	if properties.priority >= max_retries - 1: # example handling retries
		ch.basic_ack(delivery_tag=method.delivery_tag)
		print "[!] '%s' rejected after %d retries" % (data.get('keyword'), 1 + int(properties.priority))
	else:
		try:
			if data.get('keyword') == stop_word: # example - rejeceting job
				raise Exception('Stop word detected')

			time.sleep(len(data.get('keyword')))
			ch.basic_ack(delivery_tag=method.delivery_tag)
			print "[+] Done"

		except:
			timestamp = time.time()
			now = datetime.datetime.now()
			expire = 1000 * int((now.replace(hour=23, minute=59, second=59, microsecond=999999) - now).total_seconds())

			# to reject job we create new one with other priority and expiration
			channel.basic_publish(exchange='', routing_key=queue, body=json.dumps(data),
				properties=pika.BasicProperties(delivery_mode=2, priority=int(properties.priority) + 1, timestamp=timestamp, expiration=str(expire), headers=properties.headers))
			# also do not forget to send back acknowledge about job
			ch.basic_ack(delivery_tag=method.delivery_tag)
			print "[!] Rejected, going to sleep for a while"
			time.sleep(10)

	print

channel.basic_qos(prefetch_count=1)
channel.basic_consume(callback, queue=queue)

try:
	channel.start_consuming()
except KeyboardInterrupt:
	channel.stop_consuming();

connection.close()
```


How to run
----------

```sh
python send.py 6
[>] Sent 'item 1'
[>] Sent 'item 2'
[>] Sent 'item 3'
[>] Sent 'item 4'
[>] Sent 'item 5'
[>] Sent 'item 6'

python worker.py "item 3"
[*] Waiting for messages. To exit press CTRL+C
[>] Received 'item 1' (try: 1)
[+] Done

[>] Received 'item 3' (try: 1)
[!] Rejected, going to sleep for a while

[>] Received 'item 5' (try: 1)
[+] Done

python worker.py "item 5"
[*] Waiting for messages. To exit press CTRL+C
[>] Received 'item 2' (try: 1)
[+] Done

[>] Received 'item 4' (try: 1)
[+] Done

[>] Received 'item 6' (try: 1)
[+] Done

[>] Received 'item 3' (try: 2)
[+] Done
```
