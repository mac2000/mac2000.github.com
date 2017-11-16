---
layout: post
title: ModX Google Map TV

tags: [google, map, modx]
---

В ModX есть возможность добавлять кастомные поля к документам. Это крайне полезно при создании сайтов любого типа, от сайта визитки, до полноценного портала.

Но приходит момент, когда существующих типов полей становиться недостаточно.

В моем случае, мне нужна карта Google Map в качестве TV параметра к странице.

К сожалению разработчики не предусмотрели такой возможности, и придется лезть во внутренности ModX для решения такой задачи.

Править будем всего два файла: `manager/actions/mutate_tmplvars.dynamic.php` и `manager/includes/tmplvars.inc.php`. Так же понадобиться простенький чанк, для вывода карты на страницах сайта.

Кстати, можно достаточно легко сделать виджет для этого дела, но это отдельная история.

Добавляем новый тип TV
----------------------

Для этого необходимо в файле `manager/actions/mutate_tmplvars.dynamic.php` в районе строки 310 вставить следующий код:

```php
<option value="map" <?php echo ($content['type']=='map')? "selected='selected'":""; ?>>Map</option>
```

Тем самым мы добавляем новый пункт в выпадающий список типов TV на странице создания нового TV.

<amp-img src="/images/wp/modx-add-new-tv-type.png" alt="screenshot" width="300" height="156"></amp-img>

Добавляем Google Maps в админку
-------------------------------

Для этого необходимо в файле `manager/includes/tmplvars.inc.php` в районе строки 221 вставить следующий код:

```php
//Handle map input
case "map":
        $field_html .= "<script src=\"http://maps.google.com/maps?file=api&amp;v=2&amp;sensor=false&amp;key=ABQIAAAALi8oup_hVN3coIirpDRtGBSSY0Zgq2o_FhJKf_QweInG70_auRQ7W64WzKxUfZauYW3SJMv8sNc57g\" type=\"text/javascript\"></script>
            <script type=\"text/javascript\">
            jQuery(document).ready(function() {
                if (GBrowserIsCompatible()) {
                    var map = new GMap2(document.getElementById('map'));
                    map.setUIToDefault();
                    var center = new GLatLng(50.49866,30.58489);
                    map.setCenter(center, 15);
                    geocoder = new GClientGeocoder();
                    var marker = new GMarker(center, {draggable: true});
                    map.addOverlay(marker);
                    document.getElementById('tv".$field_id."').value = center.lat().toFixed(5) + '||' + center.lng().toFixed(5);

                    GEvent.addListener(marker, 'dragend', function() {
                        var point = marker.getPoint();
                        map.panTo(point);
                        document.getElementById('tv".$field_id."').value = point.lat().toFixed(5) + '||' + point.lng().toFixed(5);
                    });

                    GEvent.addListener(map, 'moveend', function() {
                        map.clearOverlays();
                        var center = map.getCenter();
                        var marker = new GMarker(center, {draggable: true});
                        map.addOverlay(marker);
                        document.getElementById('tv".$field_id."').value = center.lat().toFixed(5) + '||' + center.lng().toFixed(5);

                        GEvent.addListener(marker, 'dragend', function() {
                            var point = marker.getPoint();
                            map.panTo(point);
                            document.getElementById('tv".$field_id."').value = point.lat().toFixed(5) + '||' + point.lng().toFixed(5);
                        });
                    });
                }
            });

            function showAddress(address) {
                var map = new GMap2(document.getElementById('map'));
                map.addControl(new GSmallMapControl());
                map.addControl(new GMapTypeControl());
                if (geocoder) {
                    geocoder.getLatLng(
                        address,
                        function(point) {
                            if (!point) {
                                alert(address + ' not found');
                            } else {
                                document.getElementById('tv".$field_id."').value = point.lat().toFixed(5) + '||' + point.lng().toFixed(5);
                                map.clearOverlays()
                                map.setCenter(point, 14);
                                var marker = new GMarker(point, {draggable: true});
                                map.addOverlay(marker);

                                GEvent.addListener(marker, 'dragend', function() {
                                    var pt = marker.getPoint();
                                    map.panTo(pt);
                                    document.getElementById('tv".$field_id."').value = pt.lat().toFixed(5) + '||' + pt.lng().toFixed(5);
                                });

                                GEvent.addListener(map, 'moveend', function() {
                                    map.clearOverlays();
                                    var center = map.getCenter();
                                    var marker = new GMarker(center, {draggable: true});
                                    map.addOverlay(marker);
                                    document.getElementById('tv".$field_id."').value = center.lat().toFixed(5) + '||' + center.lng().toFixed(5);

                                GEvent.addListener(marker, 'dragend', function() {
                                    var pt = marker.getPoint();
                                    map.panTo(pt);
                                    document.getElementById('tv".$field_id."').value = pt.lat().toFixed(5) + '||' + pt.lng().toFixed(5);
                                });
                            });
                        }
                    });
                }
            }
            </script>
            <div style=\"position:relative;width:500px;height:300px;\">
                <div id=\"map\" style=\"width: 500px; height: 300px\"></div>
                <div style=\"position:absolute;right:7px;top:33px;background-color:#fff;padding:2px;border:1px solid #333;\">
                    <input type=\"text\" id=\"map".$field_id."\"  style=\"width:200px\" value=\"Ukraine, Kiev, pr. Mayakovskogo 3a\" /> <a href=\"javascript:void(0)\" onclick=\"showAddress(document.getElementById('map".$field_id."').value)\" style=\"text-decoration:none;\"><img style=\"width: 16px; height: 16px;\" src=\"media/style/MODxCarbon/images/icons/preview.png\"></a>
                </div>
            </div>
            <div style=\"display:none\">
                <input type=\"text\" id=\"tv".$field_id."\" name=\"tv".$field_id."\" value=\"".htmlspecialchars($field_value)."\" ".$field_style." tvtype=\"".$field_type."\" onchange=\"documentDirty=true;\" style=\"width:100%;\" />
            </div>";
break;
```

Тем самым мы добавляем обработчик для нашего нового типа, который будет отображать в админке, карту и позволять выбрать на ней точку, которая и будет храниться в TV.

Выглядеть это будет вот так:

<amp-img src="/images/wp/modx-add-new-tv-type2.png" alt="screenshot" width="300" height="153"></amp-img>

Использование Google Map в ModX
-------------------------------

Собственно вот и все, теперь в любом месте сайта можно использовать чанк map для вывода координат, осталась самая малость – показать карту с балунчиком указывающим на выбранные координаты.

Для этого создаем следующий чанк:

```php
<script src="http://maps.google.com/maps?file=api&amp;v=2&amp;sensor=false&amp;key=ABQIAAAALi8oup_hVN3coIirpDRtGBSSY0Zgq2o_FhJKf_QweInG70_auRQ7W64WzKxUfZauYW3SJMv8sNc57g" type="text/javascript"></script>
<script type="text/javascript">
jQuery(document).ready(function(){
                var pt = '[*map*]'.split('||');
                var message = '[*pagetitle*]';

                if (GBrowserIsCompatible()) {
                            var map = new GMap2(document.getElementById('map[*id*]'));
                            map.setUIToDefault();
                            map.setCenter(new GLatLng(pt[0], pt[1]), 13);
                            var marker = new GMarker(new GLatLng(pt[0], pt[1]));
                            marker.value = 1;

                            GEvent.addListener(marker,"click", function() {
                                            var myHtml = message;
                                            map.openInfoWindowHtml(new GLatLng(pt[0], pt[1]), myHtml);
                            });
                            map.addOverlay(marker);
                }
});
</script>
<div id="map[*id*]" style="width:100%;height:300px;"></div>
```

Следует обратить особое внимание на переменную message – в которую можно запихнуть любой html.

Ну и собственно скриншот:

<amp-img src="/images/wp/modx-add-new-tv-type3.png" alt="screenshot" width="300" height="246"></amp-img>
