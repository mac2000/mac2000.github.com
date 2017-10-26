---
layout: post
title: MaxMind BigQuery GeoIP
tags: [maxmind, bigquery]
---

Idea here is to have maxmind ip lookup table in bigquery for further logs analysis

So, first of all we need clean everything up

```sh
bq rm -f -t rualogs:data.maxmind
rm *.txt *.csv *.zip
```

Download and unzip fresh data

```sh
wget http://geolite.maxmind.com/download/geoip/database/GeoLite2-Country-CSV.zip
wget http://geolite.maxmind.com/download/geoip/database/GeoLite2-ASN-CSV.zip

unzip -jo GeoLite2-Country-CSV.zip
unzip -jo GeoLite2-ASN-CSV.zip
```

Load data into BigQuery

```sh
bq load --skip_leading_rows=1 rualogs:data.tmp_country_ip GeoLite2-Country-Blocks-IPv4.csv "network:string,geoname_id:integer,registered_country_geoname_id:integer,represented_country_geoname_id:integer,is_anonymous_proxy:integer,is_satellite_provider:integer"

bq load --skip_leading_rows=1 rualogs:data.tmp_country_labels GeoLite2-Country-Locations-en.csv "geoname_id:integer,locale_code:string,continent_code:string,continent_name:string,country_iso_code:string,country_name:string"

bq load --skip_leading_rows=1 rualogs:data.tmp_asn GeoLite2-ASN-Blocks-IPv4.csv "network:string,autonomous_system_number:integer,autonomous_system_organization:string"
```

Now is most important stuff - we are going to process uploaded data and save result as new table

```sh
bq query --use_legacy_sql=false --allow_large_results --destination_table=rualogs:data.maxmind "$(cat maxmind.sql)"
```

After this step is done we may cleanup temp tables

```sh
bq rm -f -t rualogs:data.tmp_country_ip
bq rm -f -t rualogs:data.tmp_country_labels
bq rm -f -t rualogs:data.tmp_asn
```

From now on anywhere needed we may join our maxmind table like so

```sql
SELECT l.*, m.country_name, m.autonomous_system_organization
FROM data.log AS l
LEFT JOIN data.maxmind AS m ON CAST(NET.IPV4_TO_INT64(NET.IP_FROM_STRING(l.ip))/(256*256*256) AS INT64) = class_a AND NET.IPV4_TO_INT64(NET.IP_FROM_STRING(l.ip)) BETWEEN start_num AND end_num
WHERE (_PARTITIONTIME = TIMESTAMP(CURRENT_DATE()) OR _PARTITIONTIME IS NULL)
LIMIT 100
```

**maxmind.sql**

```sql
WITH raw AS (

SELECT
REGEXP_REPLACE(network, r'/\d+$', '') as net,
CAST(REGEXP_REPLACE(network, r'^\d+\.\d+\.\d+\.\d+/', '') AS int64) as mask,
CAST(CASE WHEN (-2 + POW(2, 32 - CAST(REGEXP_REPLACE(network, r'^\d+\.\d+\.\d+\.\d+/', '') AS int64))) < 1 THEN 1 ELSE (-2 + POW(2, 32 - CAST(REGEXP_REPLACE(network, r'^\d+\.\d+\.\d+\.\d+/', '') AS int64))) END AS INT64) as hosts,
* FROM `rualogs.data.tmp_country_ip`

), num AS (

select
NET.IPV4_TO_INT64(NET.IP_FROM_STRING(net)) + 1 as start_num,
NET.IPV4_TO_INT64(NET.IP_FROM_STRING(net)) + hosts as end_num,
* from raw

), ip AS (

select
cast(start_num/(256*256*256) as int64) as class_a,
NET.IP_TO_STRING(NET.IPV4_FROM_INT64(start_num)) as start_ip,
NET.IP_TO_STRING(NET.IPV4_FROM_INT64(end_num)) as end_ip,
* from num

), maxmind AS (

select
ip.*,
l.locale_code, l.continent_code, l.continent_name, l.country_iso_code, l.country_name,
a.autonomous_system_number, a.autonomous_system_organization
from ip
left join rualogs.data.tmp_country_labels l on ip.geoname_id = l.geoname_id
left join rualogs.data.tmp_asn a on ip.network = a.network

)
select * from maxmind
```