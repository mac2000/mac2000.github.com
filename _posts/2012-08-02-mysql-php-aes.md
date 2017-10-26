---
layout: post
title: MySQL, PHP, AES encryption and decryption
tags: [mysql, php, aes, encrypt, decrypt, mcrypt, rijndeal, ecb]
---

```php
<?php
global $pdo;
$pdo = new PDO('mysql:host=localhost;dbname=[DATABASE]', '[LOGIN]', '[PASSWORD]');

$text = 'Hello World!';
$key = 'Secret';

$mysql_enc = mysql_aes_encrypt($text, $key);
$mysql_dec = mysql_aes_decrypt($mysql_enc, $key);

$php_enc = php_aes_encrypt($text, $key);
$php_dec = php_aes_decrypt($php_enc, $key);

$php_by_mysql = mysql_aes_decrypt($php_enc, $key);
$mysql_by_php = php_aes_decrypt($mysql_enc, $key);

echo 'Encryption is ';
echo $mysql_enc == $php_enc ? 'equal' : 'not equal';
echo PHP_EOL;
echo 'Decryption is ';
echo ($mysql_dec == $text && $php_dec == $text && $php_by_mysql == $text && $mysql_by_php == $text) ? 'equal' : 'not equal';


function mysql_aes_encrypt($text, $key) {
    global $pdo;
    $stmt = $pdo->prepare("SELECT AES_ENCRYPT(?, ?)");
    $stmt->execute(array($text, $key));
    return $stmt->fetchColumn(0);
}

function mysql_aes_decrypt($text, $key) {
    global $pdo;
    $stmt = $pdo->prepare("SELECT AES_DECRYPT(?, ?)");
    $stmt->execute(array($text, $key));
    return $stmt->fetchColumn(0);
}

function php_aes_encrypt($text, $key) {
    $key = mysql_aes_key($key);
    $pad_value = 16 - (strlen($text) % 16);
    $text = str_pad($text, (16 * (floor(strlen($text) / 16) + 1)), chr($pad_value));
    return mcrypt_encrypt(MCRYPT_RIJNDAEL_128, $key, $text, MCRYPT_MODE_ECB, mcrypt_create_iv(mcrypt_get_iv_size(MCRYPT_RIJNDAEL_128, MCRYPT_MODE_ECB), MCRYPT_DEV_URANDOM));
}

function php_aes_decrypt($text, $key) {
    $key = mysql_aes_key($key);
    $text = mcrypt_decrypt(MCRYPT_RIJNDAEL_128, $key, $text, MCRYPT_MODE_ECB, mcrypt_create_iv(mcrypt_get_iv_size(MCRYPT_RIJNDAEL_128, MCRYPT_MODE_ECB), MCRYPT_DEV_URANDOM));
    return rtrim($text, "\0..\16");
}

function mysql_aes_key($key) {
    $new_key = str_repeat(chr(0), 16);
    for($i=0,$len=strlen($key);$i<$len;$i++)
    {
        $new_key[$i%16] = $new_key[$i%16] ^ $key[$i];
    }
    return $new_key;
}
```

This code will return:

```
Encryption is equal
Decryption is equal
```

Do not forget to change database connection string.

Also notice that if you want to save aes encrypted values to database - use `varbinary(150)` instead of `varchar(100)`, and give it more space (at least half)

Found at: http://coding.smashingmagazine.com/2012/05/20/replicating-mysql-aes-encryption-methods-with-php/
