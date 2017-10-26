---
layout: post
title: Apache Solr for multiple sites

tags: [apache, apachesolr, java, solr, tomcat, tomcat7, ubuntu, xampp]
---

After some researches get working Apache Solr instance.

http://youtu.be/-LD1aCxYODI

Here is briew commands that will run two solr instances.

```sh
sudo apt-get install tomcat7
wget http://apache.vc.ukrtel.net//lucene/solr/3.4.0/apache-solr-3.4.0.zip
unzip apache-solr-3.4.0.zip
cp -R apache-solr-3.4.0/example/solr/ ~/example1
cp -R apache-solr-3.4.0/example/solr/ ~/example2
sudo chown -R tomcat7:tomcat7 example1
sudo chown -R tomcat7:tomcat7 example2
sudo cp apache-solr-3.4.0/dist/apache-solr-3.4.0.war /usr/share/tomcat7/lib/solr.war
sudo touch /etc/tomcat7/Catalina/localhost/example1.xml
    <Context docBase="/usr/share/tomcat7/lib/solr.war" debug="0" crossContext="true" >
        <Environment name="solr/home" type="java.lang.String" value="/home/mac/example1" override="true" />
    </Context>
sudo touch /etc/tomcat7/Catalina/localhost/example2.xml
    <Context docBase="/usr/share/tomcat7/lib/solr.war" debug="0" crossContext="true" >
        <Environment name="solr/home" type="java.lang.String" value="/home/mac/example2" override="true" />
    </Context>
sudo /etc/init.d/tomcat7 restart
```

now u can open: http://localhost:8080/example1/admin/ and http://localhost:8080/example2/admin/

Do not copy solr.war into webapps, instead copy it to tomcat lib folder. To find it use something like this:

```sh
sudo find / -name lib -type d | grep tomcat
```

Do not forget to paste correct docBase it must point to solr.war. **Actualy all you need to do is change Enviroment.value, and nothing else**, that must point to copied example/solr folder. Also do not forget that tomcat must have access to it.

If you running Windows with xampp:

Copy `example/solr` anywhere you want.

Copy solr.war to `C:\xampp\tomcat\lib`.

XML config files are stored in `C:\xampp\tomcat\conf\Catalina\localhost` folder.

Example of XML file:

```xml
<Context docBase="C:\xampp\tomcat\lib\solr.war" debug="0" crossContext="true" >
    <Environment name="solr/home" type="java.lang.String" value="C:\xampp\htdocs\solr\example1" override="true" />
</Context>
```

To restrict access add valve to context, something like this:

```xml
<Context docBase="/usr/share/tomcat6/lib/solr.war" debug="0" crossContext="true" >
<Environment name="solr/home" type="java.lang.String" value="/var/tomcat/example1" override="true" />
<Valve className="org.apache.catalina.valves.RemoteAddrValve" allow="127\.0\.0\.1"/>
</Context>
```

More info can be found here: http://tomcat.apache.org/tomcat-7.0-doc/config/valve.html

Also this can be done via firewall.

Also there is realms in tomcat, but i can not find solution for them. All examples require changing web.xml, but this file is compressed into solr.war.

Here is some examples of using solr with php, probably u can use it with any language you want, just to show how it can be done:

```php
<?php

$_HOW_MUCH = 100000;

header('Content-Type: text/html; charset=utf-8');
require_once('Apache/Solr/Service.php');
require_once 'Mac/Generator.php';

$items = array();

$solr = new Apache_Solr_Service('localhost', 8080, '/gensolr2/');?><!DOCTYPE HTML>
<html lang="en-US">
<head>
    <meta charset="UTF-8">
    <title>Import</title>
</head>
<body>

<?php
$solr->deleteByQuery('*:*');

$docs = array();
for ($i = 0; $i < $_HOW_MUCH; $i++) {
    $doc = new Apache_Solr_Document();

    srand(make_seed());
    $title = Mac_Generator::get(Mac_Generator::COMPANY_NAME);
    srand(make_seed());
    $contry = Mac_Generator::get(Mac_Generator::COUNTRY);
    srand(make_seed());
    $city = Mac_Generator::get(Mac_Generator::CITY);
    srand(make_seed());
    $auth = Mac_Generator::get(Mac_Generator::LOGIN);
    srand(make_seed());
    $text = Mac_Generator::get();
    srand(make_seed());
    $content_types = array('html', 'xml', 'txt', 'json', 'yml', 'css', 'js');
    $content_type = $content_types[array_rand($content_types)];
    srand(make_seed());
    $weight = rand(1, 100);
    srand(make_seed());
    $price = rand(10, 1000);
    srand(make_seed());
    $popularity = rand(0, 5);
    srand(make_seed());
    $inStock = rand(0, 1000) > 500;
    $categories = categories(array('blog', 'forum', 'post', 'product', 'video', 'photo', 'note', 'draft'));
    $features = features();

    $doc->addField('id', $i + 1);
    $doc->addField('title', $title);
    $doc->addField('country_s', $contry);
    $doc->addField('city_s', $city);
    $doc->addField('author', $auth);
    $doc->addField('text', $text);
    $doc->addField('content_type', $content_type);
    $doc->addField('weight', $weight);
    $doc->addField('price', $price);
    $doc->addField('popularity', $popularity);
    $doc->addField('inStock', $inStock);

    foreach ($categories as $category) $doc->addField('cat', $category);
    foreach ($features as $feature) $doc->addField('features', $feature);

    $docs[] = $doc;
}

/*$doc->addField('id', 1);
$doc->addField('title', 'tit1');
$doc->addField('text', 'text1');
$solr->addDocument($doc);*/

$solr->addDocuments($docs);
$solr->commit();

echo 'done';
?>

</body>
</html>
<?php

function make_seed()
{
    list($usec, $sec) = explode(' ', microtime());
    return (float)$sec + ((float)$usec * 100000);
}

function categories($categories)
{
    $items = array();
    srand(make_seed());
    $categories_count = rand(1, 5);
    for ($x = 0; $x < $categories_count; $x++) {
        srand(make_seed());
        $items[] = $categories[array_rand($categories)];
    }
    return array_unique($items);
}

function features()
{
    $features = array();
    srand(make_seed());
    $features_count = rand(1, 5);
    for ($x = 0; $x < $features_count; $x++) {
        srand(make_seed());
        $features[] = Mac_Generator::get(Mac_Generator::WORD);
    }
    return array_unique($features);
}
```

http://code.google.com/p/solr-php-client/used.

And here is example of simple faceted search:

```php
<?php
header('Content-Type: text/html; charset=utf-8');

$limit = 10;
$query = isset($_REQUEST['q']) ? $_REQUEST['q'] : '*:*';
$results = false;

if ($query) {
    require_once('Apache/Solr/Service.php');
    $solr = new Apache_Solr_Service('localhost', 8080, '/gensolr2/');

    if (get_magic_quotes_gpc() == 1) {
        $query = stripslashes($query);
    }

    $additionalParameters = array(
        //'fq' => 'cat:product',
        //'fq' => array('cat:product', 'weight:[50 TO *]'),
        //'fq' => array('cat:product'),
        'fq' => array(),
        'facet' => 'true',
        // notice I use an array for a muti-valued parameter
        'facet.query' => array(
            'price:[* TO 250]',
            'price:[250 TO 500]',
            'price:[500 TO 750]',
            'price:[750 TO *]',
            'weight:[* TO 50]',
            'weight:[50 TO *]',
        ),
        'facet.field' => array(
            'cat',
            'features',
            'popularity',
            'inStock',
            'content_type',
        )
    );

    $cat = isset($_REQUEST['cat']) ? $_REQUEST['cat'] : array();
    foreach ($cat as $c) $additionalParameters['fq'][] = sprintf('cat:%s', $c);

    $popularity = isset($_REQUEST['popularity']) ? $_REQUEST['popularity'] : array();
    foreach ($popularity as $p) $additionalParameters['fq'][] = sprintf('popularity:%s', $p);

    $inStock = isset($_REQUEST['inStock']) ? $_REQUEST['inStock'] : array();
    foreach ($inStock as $p) $additionalParameters['fq'][] = sprintf('inStock:%s', $p);

    $content_type = isset($_REQUEST['content_type']) ? $_REQUEST['content_type'] : array();
    foreach ($content_type as $p) $additionalParameters['fq'][] = sprintf('content_type:%s', $p);

    $price = isset($_REQUEST['price']) ? $_REQUEST['price'] : null;
    switch ($price) {
        case 1:
            {
            $additionalParameters['fq'][] = 'price:[* TO 250]';
            break;
            }
        case 2:
            {
            $additionalParameters['fq'][] = 'price:[250 TO 500]';
            break;
            }
        case 3:
            {
            $additionalParameters['fq'][] = 'price:[500 TO 750]';
            break;
            }
        case 4:
            {
            $additionalParameters['fq'][] = 'price:[750 TO *]';
            break;
            }
    }

    $weight = isset($_REQUEST['weight']) ? $_REQUEST['weight'] : null;
    switch ($price) {
        case 1:
            {
            $additionalParameters['fq'][] = 'weight:[* TO 50]';
            break;
            }
        case 2:
            {
            $additionalParameters['fq'][] = 'weight:[50 TO *]';
            break;
            }
    }

    try
    {
        $results = $solr->search($query, 0, $limit, $additionalParameters);
    }
    catch (Exception $e)
    {
        die("<html><head><title>SEARCH EXCEPTION</title><body><pre>{$e->__toString()}</pre></body></html>");
    }
}

?>
<html lang="en-US">
<head>
    <meta charset="UTF-8">
    <title>Solr</title>
    <style>
        body {
            font-size: 12px;
            font-family: Arial;
            color: #444;
        }

        table {
            border-collapse: collapse;
        }

        table th, table td {
            padding: 5px;
            border-collapse: collapse;
            font-size: 12px;
        }

        table th {
            color: #999;
        }

        li {
            margin-bottom: 10px;
        }

        #left table th, #left table td {
            border: 1px solid #ccc;
        }
    </style>
</head>
<body>

<table>
    <tr>
        <td valign="top">

            <div id="left">

                <form accept-charset="utf-8" method="get">
                    <label for="q">Search:</label>
                    <input id="q" name="q" type="text"
                            value="<?php echo htmlspecialchars($query, ENT_QUOTES, 'utf-8'); ?>"/>
                    <input type="submit"/>
                </form>
                <?php

                if ($results) {
                    $total = (int)$results->response->numFound;
                    $start = min(1, $total);
                    $end = min($limit, $total);
                    $time = $results->responseHeader->QTime;
                    ?>
                    <div>Results <?php echo $start; ?> - <?php echo $end;?> of <?php echo $total; ?>
                        in <?php echo $time ?>ms:
                    </div>
                    <ol>
                        <?php
                        foreach ($results->response->docs as $doc)
                        {
                            ?>
                            <li>
                                <table cellpadding="0" cellspacing="0" border="0">
                                    <?php
                                    // iterate document fields / values
                                    foreach ($doc as $field => $value)
                                    {
                                        if (in_array($field, array('country_s', 'city_s', 'author', 'title'))) continue;

                                        ?>
                                        <tr>
                                            <th width="100" align="right" valign="top">
                                                <?php echo htmlspecialchars($field, ENT_NOQUOTES, 'utf-8'); ?>
                                            </th>
                                            <td width="400" align="left" valign="top">
                                                <?php
                                                if (is_bool($value)) {
                                                    echo $value ? 'Yes' : 'No';
                                                } else if (is_array($value)) {
                                                    echo implode(', ', $value);
                                                } else {
                                                    echo htmlspecialchars($value, ENT_NOQUOTES, 'utf-8');
                                                }
                                                ?>
                                            </td>
                                        </tr>
                                        <?php
                                    }
                                    ?>
                                </table>
                            </li>
                            <?php
                        }
                        ?>
                    </ol>
                    <?php
                }
                ?>

            </div>

        </td>
        <td valign="top">

            <form metho="get">
                <p><b>Categories</b></p>
                <?php facet_render($results, 'cat');?>

                <p><b>Popularity</b></p>
                <?php facet_render($results, 'popularity');?>

                <p><b>In stock</b></p>
                <?php facet_render($results, 'inStock');?>

                <p><b>Content type</b></p>
                <?php facet_render($results, 'content_type');?>

                <p><b>Price</b></p>
                <label>
                    <input <?php echo isset($_REQUEST['price']) && $_REQUEST['price'] == 1 ? ' checked="checked" ' : ''?>
                        type="checkbox" name="price" value="1"> 0 TO 250
                    (<?php echo $results->facet_counts->facet_queries->{'price:[* TO 250]'};?>)
                </label>
                <br>
                <label>
                    <input <?php echo isset($_REQUEST['price']) && $_REQUEST['price'] == 2 ? ' checked="checked" ' : ''?>
                        type="checkbox" name="price" value="2"> 250 TO 500
                    (<?php echo $results->facet_counts->facet_queries->{'price:[250 TO 500]'};?>)
                </label>
                <br>
                <label>
                    <input <?php echo isset($_REQUEST['price']) && $_REQUEST['price'] == 3 ? ' checked="checked" ' : ''?>
                        type="checkbox" name="price" value="3"> 500 TO 750
                    (<?php echo $results->facet_counts->facet_queries->{'price:[500 TO 750]'};?>)
                </label>
                <br>
                <label>
                    <input <?php echo isset($_REQUEST['price']) && $_REQUEST['price'] == 4 ? ' checked="checked" ' : ''?>
                        type="checkbox" name="price" value="4"> 750 TO 1000
                    (<?php echo $results->facet_counts->facet_queries->{'price:[750 TO *]'};?>)
                </label>
                <br>

                <p><b>Weight</b></p>
                <label>
                    <input <?php echo isset($_REQUEST['weight']) && $_REQUEST['weight'] == 1 ? ' checked="checked" ' : ''?>
                        type="checkbox" name="weight" value="1"> 0 TO 50
                    (<?php echo $results->facet_counts->facet_queries->{'weight:[* TO 50]'};?>)
                </label>
                <br>
                <label>
                    <input <?php echo isset($_REQUEST['weight']) && $_REQUEST['weight'] == 2 ? ' checked="checked" ' : ''?>
                        type="checkbox" name="weight" value="2"> 50 TO 100
                    (<?php echo $results->facet_counts->facet_queries->{'weight:[50 TO *]'};?>)
                </label>
                <br>

                <input type="submit" value="Submit">
            </form>

            <pre><code><?php /*print_r($results->facet_counts)*/?></pre>

        </td>
    </tr>
</table>

</body>
</html>
<?php

function facet_render($results, $name)
{
    foreach ($results->facet_counts->facet_fields->{$name} as $k => $v) {
        if ($v == 0) continue; // skip empty facets
        echo '<label>';

        echo '<input ' . facet_checked($name, $k) . ' type="checkbox" name="' . $name . '[]" value="' . $k . '"/>';
        if (!is_facet_checked($name, $k)) echo sprintf('<a href="%s">%s (%d)</a>', facet_link($name, $k), $k, $v);
        else echo sprintf('%s (%d) <a href="%s">x</a>', $k, $v, facet_link_rem($name, $k));

        echo '</label><br />';
    }
}

function facet_link_rem($name, $key)
{
    $qs = $_SERVER['QUERY_STRING'];
    $qs = str_replace($name . '[]=' . $key, '', $qs);
    $qs = trim($qs, '&');
    $prefix = empty($qs) ? '' : '?';
    return $_SERVER['PHP_SELF'] . $prefix . $qs;
}

function facet_link($name, $key)
{
    $prefix = empty($_SERVER['QUERY_STRING']) ? '' : '?';
    $sep = empty($_SERVER['QUERY_STRING']) ? '?' : '&';
    return $_SERVER['PHP_SELF'] . $prefix . $_SERVER['QUERY_STRING'] . $sep . $name . '[]=' . $key;
}

function is_facet_checked($name, $key)
{
    return isset($_REQUEST[$name]) && in_array($key, $_REQUEST[$name]);
}

function facet_checked($name, $key)
{
    return is_facet_checked($name, $key) ? ' checked="checked" ' : '';
}
```

Notice this is not good code, just for fun.
