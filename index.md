## Zadanie GEO

Przestępstwa dokonane w promieniu kilometra od ratusza
[click](http://https://github.com/vakoz2/vakoz2.github.io/blob/master/hello.md)
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
