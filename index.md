## Zadanie GEO
### Elasticsearch
Aby wygenerować poniższe geojsony należy uruchomić skrypt **el_geo.bat**, znajdujący się w głównym katalogu. Korzystam z bazy z zadania 1. Skrypt zapisuje pliki wynikowe do katalogu geojson.

Przestępstwa dokonane w promieniu kilometra od ratusza
```markdown
{
    "query": {
        "bool" : {
            "must" : {
                "match_all" : {}
            },
            "filter" : {
                "geo_distance" : {
                    "distance" : "1km",
                    "Location": [-87.631631, 41.88386]
                }
            }
        }
    }
}
```
<script src="https://embed.github.com/view/geojson/vakoz2/nosql/master/geojson/query1.geojson"></script>
Przestępstwa dokonane na zadanym obszarze (polygon)
```markdown
{
    "query": {
        "bool" : {
            "must" : {
                "match_all" : {}
            },
            "filter" : {
                "geo_polygon" : {
                    "Location" : {
                        "points" : [
                            [-87.63119101524353, 41.89085702404937],
                            [-87.62666344642639, 41.89085702404937],
                            [-87.62666344642639, 41.89322904173341],
                            [-87.63119101524353, 41.89322904173341]
                        ]
                    }
                }
            }
        }
    }
}
```
<script src="https://embed.github.com/view/geojson/vakoz2/nosql/master/geojson/query2.geojson"></script>
Kradzieże na terenie lotniska (bounding_box)
```markdown
{
    "query": {
        "bool" : {
            "must" : {
                "match" : {"Primary Type": "THEFT"}
            },
            "filter" : {
                "geo_bounding_box" : {
                    "Location": {
                      "top_left": [-87.9400634765625,42.00772369765501],
                      "bottom_right": [-87.86848068237305,41.956171100940026]
                    }
                }
            }
        }
    }
}

```
<script src="https://embed.github.com/view/geojson/vakoz2/nosql/master/geojson/query3.geojson"></script>

### Posgresql
Korzystając z bazy z zadania pierwszego zainstalowałem rozszerzenie [PostGIS](http://postgis.net). Aby móc korzystać z zapytań geolokacyjnych należy wykonać następujące komendy:
```markdown
psql -d test -c "CREATE EXTENSION IF NOT EXISTS postgis"
psql -d test -c "ALTER TABLE crimes ADD COLUMN geom geometry(POINT,4326)"
psql -d test -c "UPDATE crimes SET geom = ST_SetSRID(ST_MakePoint(\"Longitude\",\"Latitude\"),4326)"
```
Zwróciło **UPDATE 9773**, czyli wszystkie wiersze zostały zaktualizowane.

Generowanie geojsonów już sobie odpuściełem (jest to oczywiście trywialne - wystarczy wywołać **sql2csv**(CSVKit) z odpowiednim zapytaniem i przy pomocy **csvjson** przerobić wynik na geojson). Na dowód, że wszystko jest ok, zadam pytanie o to samo, o co pytałem w przykładzie 2 w Elasticku.
```markdown
SELECT "Date", "Primary Type", "Description", "Location Description", "Arrest", "Latitude", "Longitude" 
FROM crimes 
WHERE geom &&
ST_MakeEnvelope(-87.63119101524353,41.89085702404937, -87.62666344642639,41.89322904173341, 4326)
```
Wynik:
```markdown
        Date         |    Primary Type    |          Description          | Location Description | Arrest |   Latitude   |   Longitude
---------------------+--------------------+-------------------------------+----------------------+--------+--------------+---------------
 2014-03-07 19:30:00 | DECEPTIVE PRACTICE | ILLEGAL USE CASH CARD         | HOTEL/MOTEL          | f      |   41.8926701 | -87.628106353
 2013-01-24 17:30:00 | THEFT              | RETAIL THEFT                  | DEPARTMENT STORE     | t      |  41.89164013 | -87.630200131
 2013-10-27 01:15:00 | BATTERY            | SIMPLE                        | RESTAURANT           | t      | 41.892628411 |  -87.63115524
 2016-09-24 15:00:00 | THEFT              | FROM BUILDING                 | HOTEL/MOTEL          | f      | 41.891678767 | -87.627551654
 2013-01-14 15:30:00 | THEFT              | FROM BUILDING                 | RESTAURANT           | f      | 41.892628411 |  -87.63115524
 2012-06-21 15:30:00 | DECEPTIVE PRACTICE | CREDIT CARD FRAUD             | CTA TRAIN            | f      | 41.891404732 | -87.628061509
 2014-03-25 10:25:00 | THEFT              | RETAIL THEFT                  | GROCERY FOOD STORE   | t      |  41.89200307 | -87.628076963
 2014-04-18 23:00:00 | THEFT              | POCKET-PICKING                | CTA BUS              | f      | 41.892628411 |  -87.63115524
 2013-04-13 23:15:00 | BATTERY            | AGGRAVATED: OTHER DANG WEAPON | BAR OR TAVERN        | f      | 41.890851747 | -87.628728678
 2013-05-20 17:44:00 | BATTERY            | SIMPLE                        | CTA TRAIN            | t      | 41.891404732 | -87.628061509
(10 wierszy)
```
Jak widać wszystkie punkty się zgadzają.
