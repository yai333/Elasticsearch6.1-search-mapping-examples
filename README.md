## Mapping and Analyzer

Allow to search nested object and term with hyphens

```
PUT MyIndex
{
  "mappings" : {
  "myType" : {
      "properties" : {
          "usergroup" : { "type" : "text","analyzer":"whitespace"
          },
          "form": {
            "type": "nested"
         },
         "notes": {
            "type": "nested"
         }
      }
    }
  }
}
```

## Reindex

```
------reindex-----
POST /_reindex
{
  "source": {
    "index": "myIndex"
  },
  "dest": {
    "index": "myIndex-v2"
  }
}
```

## Search

Range query

```
GET myIndex/_search
{
 "query": {
   "bool":{
     "must": [
        {"range" : {
         "created_at" : {
           "gte" : "now-1d/d",
           "lt" :  "now/d"
         }
       }},
       {"term":{
           "usergroup":"my-group-1"
         }}
     ]
   }

 },
 "sort" : [
     { "timestamp" : {"order" : "desc"}}
 ]
}
```

Query nested object

```
GET myIndex/_search
{
    "query": {
      "bool":{
        "filter": [
           {"range" : {
            "created_at" : {
              "gte" : "now-1d/d",
              "lt" :  "now/d"
            }
          }},
          {"term":{
              "usergroup":"my-group-1"
            }},
            {
              "nested": {
                "path": "form",
                "query": {
                "bool": {
                    "filter": [
                        { "term": { "form.isValid": true }}]
                	}
            	}
              }
            },
            { "term":  { "is_archived": 0 }}
        ]
      }

    },
    "sort" : [
        { "timestamp" : {"order" : "desc"}}
    ]
}
```

Dis Max Query, search forms if form status is empty or form status is new

```
GET myIndex/_search
{
    "query": {
      "bool":{
        "filter": [
           {"range" : {
            "created_at" : {
              "gte" : "now-1d/d",
              "lt" :  "now/d"
            }
          }},
          {"term":{
              "usergroup":"my-group-1"
            }},
            {
              "nested": {
                "path": "form",
                "query": [
                        {"dis_max" : {
                          "queries" : [
                              {
                                   "term":{
                                          "form.status":"new"
                                        }
                              },
                              {
                                "bool": {
                                    "must_not": {
                                        "exists": {
                                            "field": "form.status"
                                        }
                                    }
                                }
                              }

                          ]
                      }
                    },
                    {
                      "bool": {
                      "filter": [{ "term": { "form.isValid": true } }]
                      }
                    }

                ]
              }
            },
           { "term":  { "is_archived": 0 }}

        ]  
      }

    },
    "sort" : [
        { "created_at" : {"order" : "desc"}}
    ]
}
```

Multi Search

```
GET _msearch
 {"index" : "myIndex"}
 {"query" : {"match_all" : {}}, "from" : 0, "size" : 10}
 {"index" : "myIndex2"}
 {"query" : {"match_all" : {}}, "from" : 0, "size" : 10}
```

Update by query
```
POST /myindex/_update_by_query
{
  "script" : {
        "source": "ctx._source.updatefield='new value'",
        "lang": "painless",
        "params" : {
            "username" : "my username"
        }
    }
}
```
