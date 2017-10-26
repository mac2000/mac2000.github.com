---
layout: post
title: Zend gData retrive picasa albums and their photos

tags: [gdata, picasa, zend]
---

![screenshot](/images/wp/zend_gdata_picasa.png)

Here is sample code:

```php
<?php
$email = "LOGIN@gmail.com";
$pass = "PASSWORD";

set_include_path(implode(PATH_SEPARATOR, array(
    realpath('ZendGdata-1.11.7/library'),
    get_include_path(),
)));

require_once 'ZendGdata-1.11.7/library/Zend/Loader.php';

Zend_Loader::loadClass('Zend_Gdata');
Zend_Loader::loadClass('Zend_Gdata_Query');
Zend_Loader::loadClass('Zend_Gdata_ClientLogin');
Zend_Loader::loadClass('Zend_Gdata_Photos');
Zend_Loader::loadClass('Zend_Gdata_Photos_UserQuery');
Zend_Loader::loadClass('Zend_Gdata_Photos_AlbumQuery');
Zend_Loader::loadClass('Zend_Gdata_Photos_PhotoQuery');

$client = Zend_Gdata_ClientLogin::getHttpClient($email, $pass, Zend_Gdata_Photos::AUTH_SERVICE_NAME);
$service = new Zend_Gdata_Photos($client);

$albums = retrive_albums($service);

echo '<h3>ALBUMS LIST</h3><table border="1"><tr><th>id</th><th>name</th></tr>';
foreach($albums as $album) {
    echo '<tr><td>' . $album['id'] . '</td><td>' . $album['name'] . '</td></tr>';
}
echo '</table>';

$album_index = 5;
echo '<h3>PHOTOS IN : ' . $albums[$album_index]['name'] . '</h3><table border="1"><tr><th>thumb</th><th>full</th></tr>';
$photos = retrive_photos($service, $albums[$album_index]['id']);
foreach($photos as $photo) {
    echo '<tr><td><img src="' . $photo['thumbnail'] . '" /></td><td><img src="' . $photo['fullsize'] . '" /></td></tr>';
}

/////////////////////////////////////////////

function retrive_albums($service) {
    $albums = array();

    $results = $service->getUserFeed();
    while($results != null) {
        foreach($results as $entry) {
            $album_id = $entry->gphotoId->text;//$entry->getId();//$entry->getGphotoId()->getText();
            $album_name = $entry->title->text;

            $albums[] = array(
                'id' => $album_id,
                'name' => $album_name
            );
        }

        try {
            $results = $results->getNextFeed();
        }
        catch(Exception $e) {$results = null;}
    }

    return $albums;
}

function retrive_photos($service, $album_id) {
    $photos = array();
    $query = new Zend_Gdata_Photos_AlbumQuery();
    $query->setAlbumId($album_id);
    // http://code.google.com/intl/ru/apis/picasaweb/docs/2.0/reference.html
    $query->setThumbsize(144);
    $query->setImgMax(800);
    $results = $service->getAlbumFeed($query);
    while($results != null) {
        foreach($results as $entry) {
            foreach($results as $photo) {
                //thumbnail(s): $photo->mediaGroup->thumbnail[0]->url
                //fullsize: $photo->mediaGroup->content[0]->url
                $photos[] = array(
                    'thumbnail' => $photo->mediaGroup->thumbnail[0]->url,
                    'fullsize' => $photo->mediaGroup->content[0]->url
                );
            }
        }
        try {
            $results = $results->getNextFeed();
        }
        catch(Exception $e) {$results = null;}
    }

    return $photos;
}
```

For this example work u need download zend gdata library.
