## Zadanie GEO

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
