---
layout: post
title: PHP SQLite
tags: [php, sqlite, exec, prepare, stmt, pdo, dbal, memory]
---

Short note to memorize how to access sqlite databases from php


DBAL
----

```php
<?php
use Doctrine\DBAL\Configuration;
use Doctrine\DBAL\DriverManager;

require_once 'vendor/autoload.php';

$connection = DriverManager::getConnection([
    'driver' => 'pdo_sqlite',
    //'path' => 'database.sqlite3',
    'memory' => true
], new Configuration());

$connection->exec('DROP TABLE IF EXISTS users');

$connection->exec("CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    age INTEGER DEFAULT 0
)");

$connection->insert('users', ['first_name' => 'Alexandr', 'last_name' => 'Marchenko', 'age' => 29]);

$stmt = $connection->prepare("SELECT first_name || ' ' || last_name AS full_name, age FROM users WHERE age > :age");

$stmt->execute(['age' => 10]);

while($user = $stmt->fetch(PDO::FETCH_ASSOC)) {
    echo $user['full_name'] . ' (' . $user['age'] . ')' . PHP_EOL;
}
```


PDO
---

```php
<?php
$connection = new PDO('sqlite::memory:', null, null, [PDO::ERRMODE_EXCEPTION => true]);
//$connection = new PDO('sqlite:database.sqlite3', null, null, [PDO::ERRMODE_EXCEPTION => true]);

$connection->exec("DROP TABLE IF EXISTS users");

$connection->exec("CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    age INTEGER DEFAULT 0
)");

$stmt = $connection->prepare("INSERT INTO users VALUES(NULL, :first_name, :last_name, :age)");
$stmt->execute(['first_name' => 'Alexandr', 'last_name' => 'Marchenko', 'age' => 29]);

$stmt = $connection->prepare("SELECT first_name || ' ' || last_name AS full_name, age FROM users WHERE age > :age");

$stmt->execute(['age' => 10]);

while($user = $stmt->fetch(PDO::FETCH_ASSOC)) {
    echo $user['full_name'] . ' (' . $user['age'] . ')' . PHP_EOL;
}
```


SQLite3
-------

```php
<?php
$connection = new SQLite3(':memory:');
//$connection = new SQLite3('database.sqlite3');

$connection->exec("DROP TABLE IF EXISTS users");

$connection->exec("CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    age INTEGER DEFAULT 0
)");

$stmt = $connection->prepare("INSERT INTO users VALUES(NULL, :first_name, :last_name, :age)");
$stmt->bindValue(':first_name', 'Alexandr');
$stmt->bindValue(':last_name', 'Marchenko');
$stmt->bindValue(':age', 29);
$stmt->execute();

$stmt = $connection->prepare("SELECT first_name || ' ' || last_name AS full_name, age FROM users WHERE age > :age");
$stmt->bindValue(':age', 10);
$result = $stmt->execute();
while($user = $result->fetchArray(SQLITE3_ASSOC)) {
    echo $user['full_name'] . ' (' . $user['age'] . ')' . PHP_EOL;
}
```
