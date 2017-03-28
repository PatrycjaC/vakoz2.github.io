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

Generowanie geojsonów już sobie odpuściełem (jest to oczywiście trywialne - wystarczy wywołać **sql2csv**(CSVKit) z odpowiednim zapytaniem i przy pomocy **csvjson** przerobić wynik na geojson). Na dowód, że wszystko jest ok, zadam pytanie o to samo, o co pytałem w ostatnim przykładzie w Elasticku.
```markdown
SELECT "Date", "Primary Type", "Description", "Location Description", "Arrest"
FROM crimes 
WHERE "Primary Type" LIKE '%THEFT%' AND geom &&
ST_MakeEnvelope(-87.93834686279295, 41.95693703889415, -87.87483215332031, 42.0064481470799, 4326)
```
Wynik:
```markdown
        Date         |    Primary Type     |   Description   |              Location Description              | Arrest
---------------------+---------------------+-----------------+------------------------------------------------+--------
 2016-06-06 14:00:00 | THEFT               | PURSE-SNATCHING | CTA TRAIN                                      | f
 2015-07-27 16:30:00 | THEFT               | OVER $500       | AIRPORT TERMINAL UPPER LEVEL - SECURE AREA     | f
 2013-03-28 15:30:00 | THEFT               | OVER $500       | AIRPORT TERMINAL LOWER LEVEL - NON-SECURE AREA | f
 2014-03-21 08:00:00 | MOTOR VEHICLE THEFT | AUTOMOBILE      | AIRPORT VENDING ESTABLISHMENT                  | f
 2016-08-21 14:09:00 | THEFT               | $500 AND UNDER  | AIRCRAFT                                       | f
 2012-03-30 00:01:00 | THEFT               | OVER $500       | PARKING LOT/GARAGE(NON.RESID.)                 | f
 2015-09-02 12:00:00 | THEFT               | $500 AND UNDER  | AIRPORT VENDING ESTABLISHMENT                  | f
 2013-10-06 20:00:00 | THEFT               | $500 AND UNDER  | AIRPORT TERMINAL LOWER LEVEL - SECURE AREA     | f
 2016-11-06 11:00:00 | THEFT               | FROM BUILDING   | AIRPORT TERMINAL UPPER LEVEL - SECURE AREA     | f
 2014-07-30 16:00:00 | THEFT               | $500 AND UNDER  | AIRPORT TERMINAL UPPER LEVEL - NON-SECURE AREA | f
 2013-03-14 17:00:00 | THEFT               | $500 AND UNDER  | AIRPORT TERMINAL LOWER LEVEL - SECURE AREA     | f
 2016-01-22 14:00:00 | THEFT               | $500 AND UNDER  | AIRPORT TERMINAL LOWER LEVEL - SECURE AREA     | f
 2015-09-20 18:00:00 | THEFT               | RETAIL THEFT    | AIRPORT VENDING ESTABLISHMENT                  | f
(13 wierszy)
```
Jak widać wszystkie punkty się zgadzają.
