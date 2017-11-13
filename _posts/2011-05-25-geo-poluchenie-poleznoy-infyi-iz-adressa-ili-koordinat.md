---
layout: post
title: GEO получение полезной инфы из адресса или координат

tags: [geo, geocoder, google, map, metro, wikimapia, yandex, ymaps]
---

Полноценное использование современных сервисов для получения гео данных от пользователя.

<amp-img src="/images/wp/map-helper.png" alt="screenshot" width="1199" height="609"></amp-img>

Основная идея в том чтобы избавить пользователя от необходимости вручную вбивать такие данные как: Страна, Городо, Район. Достаточно лишь указать адрес или положение на карте все остальные данные получаются у Яндекса, Google и Wikimapia. Так же как дополнительный бонус, благодаря Яндексу можно получить данные о ближайших станциях метро.

Это упрощает пользователю заполнение информации, все данные разбиты - то есть их можно будет использовать, к примеру, для поиска, скажем - "выведи мне все записи у которых расстояние для ближайшего метро не больше 1000 метров" или "выведи мне все записи которые расположены в неофиц. микрорайоне Троещина".  Плюс ко всему можно будет на отдельных страницах выводить карты с объектами, строить маршруты и т.д. и т.п.

**Как работает:**

Довольно примитивный механизм - за основу взять geocoder Яндекса, в дополнение к нему дергается GoogleGeocoder (для получения координат района адреса) и Wikimapia (для получения микрорайонов адреса).

**index.php**

```php
<?php
// KEYS
$google = 'ABQIAAAALi8oup_hVN3coIirpDRtGBTNj2TnvvsBgOKuWKvD1ZenHSzIvhSH1vUUjJqNcahwBh-WWJjH_yteNg';
$yandex = 'AAyl3E0BAAAAjZLmXAQAfFspVbDVrROagfcue18lh24kJX0AAAAAAAAAAAC2vCPmciciz70C6HfABZKJKsbHZw==';
?><!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>Map Helper</title>
<style type="text/css">
    .map-helper-map,
    .map-helper-overlay {width:100%;height:100%;position:absolute;top:0;left:0;}
    .map-helper-overlay {z-index:2;background:#fff url(http://habrastorage.org/storage/2a669297/3429d7a7/4513bfa8/bcb5be20.gif) no-repeat 50% 50%;filter:progid:DXImageTransform.Microsoft.Alpha(opacity=50);-moz-opacity: 0.5;-khtml-opacity: 0.5;opacity: 0.5;}
</style>
<script type="text/javascript" src="http://code.jquery.com/jquery-1.5.min.js"></script>
<script src="http://maps.google.com/maps?file=api&amp;v=2.133d&amp;key=<?php print $google?>" type="text/javascript"></script>
<script src="http://api-maps.yandex.ru/1.1/index.xml?key=<?php print $yandex?>&modules=regions~metro~pmap" type="text/javascript"></script>
<script type="text/javascript">
    var map_holder_selector = '#map';

    var map, placemark, yandex_geocoder, metro_geocoder;
    var google_geocoder = null;

    var is_search_done = {
        yandex_geocoder: false,
        google_geocoder: false,
        metro_geocoder: false,
        wikimapia_geocoder: false
    };

    var current_point_geodata = {
        Latitude: 0,
        Longtitude: 0,
        YandexAddressDetails: {},
        GoogleAddressDetails: {},
        WikimapiaAddressDetails: [],
        metroStations: []
    };

    YMaps.jQuery(function () {
        google_geocoder = new GClientGeocoder();

        YMaps.jQuery(map_holder_selector).css('position', 'relative').html('<div class="map-helper-map"></div><div class="map-helper-overlay" style="display:none"></div>');

        map = new YMaps.Map(YMaps.jQuery(map_holder_selector).find('div.map-helper-map:first').get(0));
        map.setCenter(new YMaps.GeoPoint(30.522301, 50.451118), 10);
        map.addControl(new YMaps.Zoom());
        var searchControl = new YMaps.SearchControl({noPlacemark: true});
        map.addControl(searchControl);

        placemark = new YMaps.Placemark(map.getCenter(), {draggable: true, style: 'default#hospitalIcon'});
        placemark.name = "Москва"; //TODO: change this
        placemark.description = "Столица Российской Федерации";

        map.addOverlay(placemark);
        getgeodata();

        YMaps.Events.observe(searchControl, searchControl.Events.Select, function (obj, geoResult) {
            placemark.setGeoPoint(geoResult.getGeoPoint());
            getgeodata();
        });

        YMaps.Events.observe(placemark, placemark.Events.DragEnd, function (obj) {
            getgeodata();
        });

        function show_overlay() {
            YMaps.jQuery(map_holder_selector).find('div.map-helper-overlay:first').show();

            YMaps.jQuery.each(is_search_done, function(k, v){
                is_search_done[k] = false;
            });
        }

        function hide_overlay() {
            var res = true;
            YMaps.jQuery.each(is_search_done, function(k, v){
                if(!v && res) res = false;
            });

            if(res) {
                YMaps.jQuery(map_holder_selector).find('div.map-helper-overlay:first').hide();
                YMaps.jQuery(map_holder_selector).trigger('change', proccess_geo_data(current_point_geodata));
            }
        }

        function getgeodata() {
            show_overlay();
            current_point_geodata.Latitude = placemark.getGeoPoint().getLat();
            current_point_geodata.Longtitude = placemark.getGeoPoint().getLng();

            //get yandex address details
            yandex_geocoder = new YMaps.Geocoder(placemark.getGeoPoint(), {results: 1});
            YMaps.Events.observe(yandex_geocoder, yandex_geocoder.Events.Load, function () {
                current_point_geodata.YandexAddressDetails = {};
                if(this.length()) {
                    current_point_geodata.YandexAddressDetails = getFlatAddressDetails(this.get(0).AddressDetails, current_point_geodata.YandexAddressDetails);
                }
                is_search_done.yandex_geocoder = true;
                hide_overlay();
                dump();
            });
            YMaps.Events.observe(yandex_geocoder, yandex_geocoder.Events.Fault, function (yandex_geocoder, error) {
                current_point_geodata.YandexAddressDetails = {};

                is_search_done.yandex_geocoder = true;
                hide_overlay();
                dump();
            });

            //get closes metro stations
            var metro_geocoder = new YMaps.Metro.Closest(placemark.getGeoPoint(), { results : 2 } );
            YMaps.Events.observe(metro_geocoder, metro_geocoder.Events.Load, function (metro_geocoder) {
                current_point_geodata.metroStations = [];

                if (metro_geocoder.length()) {
                    for(var i = 0; i < metro_geocoder.length(); i++) {
                        var metro_station = metro_geocoder.get(i);
                        var metroFlatAddressDetails = {};
                        metroFlatAddressDetails = getFlatAddressDetails(metro_station.AddressDetails, metroFlatAddressDetails);
                        if(typeof metroFlatAddressDetails.PremiseName != 'undefined') {
                            var dist = Distance(placemark.getGeoPoint().getLat(), placemark.getGeoPoint().getLng(), metro_station.getGeoPoint().getLat(), metro_station.getGeoPoint().getLng());
                            var o = {
                                Name: metroFlatAddressDetails.PremiseName,
                                Distance: Math.round(dist)
                            };
                            current_point_geodata.metroStations.push(o);
                        }
                    }
                }
                is_search_done.metro_geocoder = true;
                hide_overlay();
                dump();
            });
            YMaps.Events.observe(metro_geocoder, metro_geocoder.Events.Fault, function (metro_geocoder, error) {
                current_point_geodata.metroStations = [];
                is_search_done.metro_geocoder = true;
                hide_overlay();
                dump();
            });

            //get google address details
            google_geocoder.getLocations(new GLatLng(placemark.getGeoPoint().getLat(), placemark.getGeoPoint().getLng()), function(addresses) {
                if(addresses.Status.code != 200) {
                    current_point_geodata.GoogleAddressDetails = {};
                    is_search_done.google_geocoder = true;
                    hide_overlay();
                    dump();
                }
                else {
                    current_point_geodata.GoogleAddressDetails = {};
                    current_point_geodata.GoogleAddressDetails = getFlatAddressDetails(addresses.Placemark[0].AddressDetails, current_point_geodata.GoogleAddressDetails);

                    var params = {
                        lon : current_point_geodata.Longtitude,
                        lat : current_point_geodata.Latitude,
                        lon_min : addresses.Placemark[1].ExtendedData.LatLonBox.west,
                        lat_min : addresses.Placemark[1].ExtendedData.LatLonBox.south,
                        lon_max : addresses.Placemark[1].ExtendedData.LatLonBox.east,
                        lat_max : addresses.Placemark[1].ExtendedData.LatLonBox.north
                    };

                    current_point_geodata.WikimapiaAddressDetails = [];
                    YMaps.jQuery.getJSON('wikimapia.php', params, function(data, textStatus, jqXHR){
                        YMaps.jQuery.each(data, function(k, v){
                            current_point_geodata.WikimapiaAddressDetails.push(v);
                        });
                        is_search_done.wikimapia_geocoder = true;
                        hide_overlay();
                        dump();
                    });
                    //TODO: get wikimapia data here

                    is_search_done.google_geocoder = true;
                    hide_overlay();
                    dump();
                }
            });

            dump();
        }

        function dump() {
            /*var res = document.getElementById('res');
            h = '';

            h += '<b>Latitude</b> : ' + current_point_geodata.Latitude + '<br />';
            h += '<b>Longtitude</b> : ' + current_point_geodata.Longtitude + '<br />';

            h += '<br /><b>Yandex Address Details</b><hr />';
            YMaps.jQuery.each(current_point_geodata.YandexAddressDetails, function(k, v){
                h += '<b>' + k + '</b> : ' + v + '<br />';
            });
            h += '<br /><b>Google Address Details</b><hr />';
            YMaps.jQuery.each(current_point_geodata.GoogleAddressDetails, function(k, v){
                h += '<b>' + k + '</b> : ' + v + '<br />';
            });
            h += '<br /><b>Wikimapia Address Details</b><hr />';
            h += current_point_geodata.WikimapiaAddressDetails.join(', ');

            h += '<br /><br /><b>Metro stations</b><hr />';
            YMaps.jQuery.each(current_point_geodata.metroStations, function(k, v){
                YMaps.jQuery.each(v, function(k, v){
                    h += '<b>metro ' + k + '</b> : ' + v + '<br />';
                });
            });

            res.innerHTML = h;*/
        }

        function getFlatAddressDetails(AddressDetails, FlatAddressDetails) {
            YMaps.jQuery.each(AddressDetails, function(k, v){
                if(typeof v == 'object') FlatAddressDetails = getFlatAddressDetails(v, FlatAddressDetails);
                if(typeof v == 'string') FlatAddressDetails[k] = v;
            });

            return FlatAddressDetails;
        }

        function Distance(lat1,lng1,lat2,lng2) {
            var plq = new YMaps.GeoCoordSystem();
            Rasto = plq.distance(new YMaps.GeoPoint(lat1,lng1), new YMaps.GeoPoint(lat2,lng2));
            return Rasto;
        }

        function get_wikimapia_url(LatLonBox) {
            return 'http://api.wikimapia.org/?function=box&key=' + wikimapia_key + '&language=ru&lon_min=' + LatLonBox.west + '&lat_min=' + LatLonBox.south + '&lon_max=' + LatLonBox.east + '&lat_max=' + LatLonBox.north + '&rsaquo&count=100';
        }

        function proccess_geo_data(data) {
            var res = {};

            res.Latitude = data.Latitude;
            res.Longtitude = data.Longtitude;

            res.Country = '';

            if(typeof data.GoogleAddressDetails.CountryName != 'undefined') {
                res.Country = data.GoogleAddressDetails.CountryName;
            }
            if(typeof data.YandexAddressDetails.CountryName != 'undefined') {
                res.Country = data.YandexAddressDetails.CountryName;
            }

            res.City = '';

            if(typeof data.GoogleAddressDetails.LocalityName != 'undefined') {
                res.City = data.GoogleAddressDetails.LocalityName;
            }
            if(typeof data.YandexAddressDetails.LocalityName != 'undefined') {
                res.City = data.YandexAddressDetails.LocalityName;
            }

            res.Distinct = '';

            if(typeof data.YandexAddressDetails.DependentLocalityName != 'undefined') {
                res.Distinct = data.YandexAddressDetails.DependentLocalityName;
            }

            res.Street = '';

            if(typeof data.GoogleAddressDetails.ThoroughfareName != 'undefined') {
                res.Street = data.GoogleAddressDetails.ThoroughfareName;
            }
            if(typeof data.YandexAddressDetails.ThoroughfareName != 'undefined') {
                res.Street = data.YandexAddressDetails.ThoroughfareName;

                if(typeof data.YandexAddressDetails.PremiseNumber != 'undefined') {
                    res.Street += ', ' + data.YandexAddressDetails.PremiseNumber;
                }
            }

            res.MetroStations = data.metroStations;

            res.MicroDistincts = [];
            YMaps.jQuery.each(data.WikimapiaAddressDetails, function(k, v){
                if(v != res.Country && v != res.City && v != res.Distinct) {
                    res.MicroDistincts.push(v);
                }
            });

            return res;
        }

        YMaps.jQuery(map_holder_selector).bind('change', function(event, data){
            var res = document.getElementById('res');
            h = '<table>';

            h += '<tr><th align="left">Point(lat, lng)</th><td>' + data.Latitude + ', ' +  data.Longtitude + '</td></tr>';
            h += '<tr><th align="left">Страна</th><td>' + data.Country + '</td></tr>';
            h += '<tr><th align="left">Город</th><td>' + data.City + '</td></tr>';
            h += '<tr><th align="left">Район</th><td>' + data.Distinct + '</td></tr>';
            h += '<tr><th align="left">Район (неофиц.)</th><td>' + data.MicroDistincts.join(', ') + '</td></tr>';
            h += '<tr><th align="left">Улица</th><td>' + data.Street + '</td></tr>';

            var m = [];
            YMaps.jQuery.each(data.MetroStations, function(k, v){
                m.push(v.Name + ' (' + v.Distance + 'м.)');
            });
            h += '<tr><th align="left">Метро</th><td>' + m.join(', ') + '</td></tr>';

            res.innerHTML = h;
        });

    });
</script>
</head>
<body>
    <table>
        <tr>
            <td>
                <div id="map" style="width:600px;height:600px;"></div>
            </td>
            <td>
                <div id="res"></div>
            </td>
        </tr>
    </table>
</body>
</html>
```

**wikimapia.php**

```php
<?php
//TODO: cache results
$key = '831889EA-4C4527B7-870F69D2-C37B4C61-46DBC5D8-04493657-73823C37-F7A4545C';

$lat = isset($_REQUEST['lat']) ? floatval($_REQUEST['lat']) : false;
$lat_max = isset($_REQUEST['lat_max']) ? floatval($_REQUEST['lat_max']) : false;
$lat_min = isset($_REQUEST['lat_min']) ? floatval($_REQUEST['lat_min']) : false;
$lon = isset($_REQUEST['lon']) ? floatval($_REQUEST['lon']) : false;
$lon_max = isset($_REQUEST['lon_max']) ? floatval($_REQUEST['lon_max']) : false;
$lon_min = isset($_REQUEST['lon_min']) ? floatval($_REQUEST['lon_min']) : false;

if(!$lat || !$lat_max || !$lat_min || !$lon || !$lon_max || !$lon_min) {
    die('wrong params');
}

$places = get_places_from_cache($key, $lat, $lat_max, $lat_min, $lon, $lon_max, $lon_min);

//echo 'Total: ' . count($places) . '<br />';

$tags = array();
foreach($places as $place) {
    $in_poly = is_in_polygon($place['points'], $lat, $lon);
    if($in_poly)
        $tags[] = $place['name'];
}

$tags = array_unique($tags);

//foreach($tags as $tag) {
//  echo $tag . '<br />';
//}

echo json_encode($tags);

/////////////////////////////////////////////////

function get_cache_key($lat_max, $lat_min, $lon_max, $lon_min) {
    return (string)$lat_max . '-' . (string)$lat_min . '-' . (string)$lon_max . '-' . (string)$lon_min;
}

function get_places_from_cache($key, $lat, $lat_max, $lat_min, $lon, $lon_max, $lon_min) {
    $cache_key = get_cache_key($lat_max, $lat_min, $lon_max, $lon_min);
    $cache_file = dirname(__FILE__) . '/cache/' . $cache_key;
    $places = array();
    if(file_exists($cache_file)) {
        $places = unserialize(file_get_contents($cache_file));
    } else {
        $places = get_places_from_wikimapia($key, $lat, $lat_max, $lat_min, $lon, $lon_max, $lon_min);
        file_put_contents($cache_file, serialize($places));
    }
    return $places;
}

function get_places_from_wikimapia($key, $lat, $lat_max, $lat_min, $lon, $lon_max, $lon_min) {
    $page = 1;
    $results = true;
    $places = array();

    do {
        $url = 'http://api.wikimapia.org/?function=box&key=' . $key . '&language=ru&lon_min=' . $lon_min . '&lat_min=' . $lat_min . '&lon_max=' . $lon_max . '&lat_max=' . $lat_max . '&rsaquo&count=100&page=' . $page;
        $xml = file_get_contents($url); //TODO: try catch here

        $doc= new DOMDocument();
        $doc->loadXML($xml);

        $found = $doc->getElementsByTagName('folder')->item(0)->getAttribute('found');
        $pages = ceil($found / 100);

        if($found == 0 || $page++ >= $pages) {
            $results = false;
        }

        $items = get_places_from_xml($doc);
        foreach($items as $item) {
            $places[] = $item;
        }

    } while($results);

    return $places;
}

function get_places_from_xml(DOMDocument $doc) {
    $places = array();

    foreach ($doc->getElementsByTagName('place') as $node) {
        $name = $node->getElementsByTagName('name')->item(0)->nodeValue;

        $points = array();
        foreach ($node->getElementsByTagName('point') as $p) {
            $lng = $p->getAttribute('x');
            $lat = $p->getAttribute('y');
            $points[] = array(
                'lng' => floatval($lng),
                'lat' => floatval($lat)
            );
        }

        $places[] = array(
            'name' => $name,
            'points' => $points
        );
    }

    return $places;
}

function is_in_polygon($points, $lat, $lng) {
    $j=count($points)-1;
    $in_poly=false;

    for ($i = 0; $i < count($points); $i++) {
        if ($points[$i]['lng'] < $lng && $points[$j]['lng'] >= $lng ||  $points[$j]['lng'] < $lng && $points[$i]['lng'] >= $lng) {
            if ($points[$i]['lat'] + ($lng - $points[$i]['lng']) / ($points[$j]['lng'] - $points[$i]['lng']) * ($points[$j]['lat'] - $points[$i]['lat']) < $lat) {
                $in_poly=!$in_poly;
            }
        }
        $j = $i;
    }

    return $in_poly;
}
```

В факту выбора нового адреса\положения на карте генерируется событие change на которое можно повесится откуда угодно. В качестве параметров передаются полученные данные.

TODO: интегрироваться в Drupal

TODO: research добавление координаты в Google Places, Народные карты Яндекс и Wikimapia
