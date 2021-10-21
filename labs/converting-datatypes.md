- Indice field's specification:

```bash
--------------------------------------+-----------+-----------+-----------
Field                                 | Datatype  | Character | Analyzer  
                                      |           | Limit     |           
--------------------------------------+-----------+-----------+-----------
@message                              | text      |           | standard  
--------------------------------------+-----------+-----------+-----------
@tags                                 | keyword   | 128       |           
--------------------------------------+-----------+-----------+-----------
@timestamp                            | date      |           |           
--------------------------------------+-----------+-----------+-----------
@version                              | keyword   | 256       |           
--------------------------------------+-----------+-----------+-----------
agent                                 | text      |           | standard  
--------------------------------------+-----------+-----------+-----------
agent.keyword                         | keyword   | 256       |           
--------------------------------------+-----------+-----------+-----------
bytes                                 | long      |           |           
--------------------------------------+-----------+-----------+-----------
clientip                              | ip        |           |           
--------------------------------------+-----------+-----------+-----------
extension                             | keyword   | 256       |           
--------------------------------------+-----------+-----------+-----------
geo.coordinates                       | geo_point |           |           
--------------------------------------+-----------+-----------+-----------
geo.dest                              | keyword   | 128       |           
--------------------------------------+-----------+-----------+-----------
geo.src                               | keyword   | 128       |           
--------------------------------------+-----------+-----------+-----------
geo.srcdest                           | keyword   | 128       |           
--------------------------------------+-----------+-----------+-----------
headings                              | keyword   | 256       |           
--------------------------------------+-----------+-----------+-----------
host                                  | keyword   | 256       |           
--------------------------------------+-----------+-----------+-----------
ip                                    | ip        |           |           
--------------------------------------+-----------+-----------+-----------
links                                 | keyword   | 256       |           
--------------------------------------+-----------+-----------+-----------
machine.os                            | keyword   | 256       |           
--------------------------------------+-----------+-----------+-----------
machine.ram                           | long      |           |           
--------------------------------------+-----------+-----------+-----------
memory                                | long      |           |           
--------------------------------------+-----------+-----------+-----------
phpmemory                             | long      |           |           
--------------------------------------+-----------+-----------+-----------
referer                               | keyword   | 256       |           
--------------------------------------+-----------+-----------+-----------
relatedContent.article:modified_time  | date      |           |           
--------------------------------------+-----------+-----------+-----------
relatedContent.article:published_time | date      |           |           
--------------------------------------+-----------+-----------+-----------
relatedContent.article:section        | keyword   | 128       |           
--------------------------------------+-----------+-----------+-----------
relatedContent.article:tag            | keyword   | 128       |           
--------------------------------------+-----------+-----------+-----------
relatedContent.og:description         | text      |           | standard  
--------------------------------------+-----------+-----------+-----------
relatedContent.og:image               | keyword   | 256       |           
--------------------------------------+-----------+-----------+-----------
relatedContent.og:image:height        | long      |           |           
--------------------------------------+-----------+-----------+-----------
relatedContent.og:image:width         | long      |           |           
--------------------------------------+-----------+-----------+-----------
relatedContent.og:site_name           | keyword   | 256       |           
--------------------------------------+-----------+-----------+-----------
relatedContent.og:title               | text      |           | standard  
--------------------------------------+-----------+-----------+-----------
relatedContent.og:title.keyword       | keyword   | 256       |           
--------------------------------------+-----------+-----------+-----------
relatedContent.og:type                | keyword   | 128       |           
--------------------------------------+-----------+-----------+-----------
relatedContent.og:url                 | text      |           | simple    
--------------------------------------+-----------+-----------+-----------
relatedContent.og:url.keyword         | keyword   | 256       |           
--------------------------------------+-----------+-----------+-----------
relatedContent.twitter:card           | keyword   | 128       |           
--------------------------------------+-----------+-----------+-----------
relatedContent.twitter:description    | text      |           | standard  
--------------------------------------+-----------+-----------+-----------
relatedContent.twitter:image          | keyword   | 256       |           
--------------------------------------+-----------+-----------+-----------
relatedContent.twitter:site           | keyword   | 128       |           
--------------------------------------+-----------+-----------+-----------
relatedContent.twitter:title          | text      |           | standard  
--------------------------------------+-----------+-----------+-----------
relatedContent.twitter:title.keyword  | keyword   | 128       |           
--------------------------------------+-----------+-----------+-----------
relatedContent.url                    | text      |           | simple    
--------------------------------------+-----------+-----------+-----------
relatedContent.url.keyword            | keyword   | 256       |           
--------------------------------------+-----------+-----------+-----------
request                               | text      |           | simple    
--------------------------------------+-----------+-----------+-----------
request.keyword                       | keyword   | 256       |           
--------------------------------------+-----------+-----------+-----------
response                              | keyword   | 128       |           
--------------------------------------+-----------+-----------+-----------
spaces                                | text      |           | whitespace
--------------------------------------+-----------+-----------+-----------
url                                   | text      |           | simple    
--------------------------------------+-----------+-----------+-----------
url.keyword                           | keyword   | 256       |           
--------------------------------------+-----------+-----------+-----------
utc_time                              | date      |           |           
--------------------------------------+-----------+-----------+-----------
xss                                   | text      |           | standard  
--------------------------------------+-----------+-----------+-----------
xss.keyword                           | keyword   | 512       |           
--------------------------------------+-----------+-----------+-----------
```

- Indice implementation:

```json
PUT logs_new
{
  "mappings": {
    "properties": {
      "@message": {
        "type": "text"
      },
      "@tags": {
        "type": "keyword",
        "ignore_above": 128
      },
      "@timestamp": {
        "type": "date"
      },
      "@version": {
        "type": "keyword",
        "ignore_above": 256
      },
      "agent": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          }
        }
      },
      "bytes": {
        "type": "long"
      },
      "clientip": {
        "type": "ip"
      },
      "extension": {
        "type": "keyword",
        "ignore_above": 256
      },
      "geo": {
        "properties": {
          "coordinates": {
            "type": "geo_point"
          },
          "dest": {
            "type": "keyword",
            "ignore_above": 128
          },
          "src": {
            "type": "keyword",
            "ignore_above": 128
          },
          "srcdest": {
            "type": "keyword",
            "ignore_above": 128
          }
        }
      },
      "headings": {
        "type": "keyword",
        "ignore_above": 256
      },
      "host": {
        "type": "keyword",
        "ignore_above": 256
      },
      "ip": {
        "type": "ip"
      },
      "links": {
        "type": "keyword",
        "ignore_above": 256
      },
      "machine": {
        "properties": {
          "os": {
            "type": "keyword",
            "ignore_above": 256
          },
          "ram": {
            "type": "long"
          }
        }
      },
      "memory": {
        "type": "long"
      },
      "phpmemory": {
        "type": "long"
      },
      "referer": {
        "type": "keyword",
        "ignore_above": 256
      },
      "relatedContent": {
        "properties": {
          "article:modified_time": {
            "type": "date"
          },
          "article:published_time": {
            "type": "date"
          },
          "article:section": {
            "type": "keyword",
            "ignore_above": 128
          },
          "article:tag": {
            "type": "keyword",
            "ignore_above": 128
          },
          "og:description": {
            "type": "text"
          },
          "og:image": {
            "type": "keyword",
            "ignore_above": 256
          },
          "og:image:height": {
            "type": "long"
          },
          "og:image:width": {
            "type": "long"
          },
          "og:site_name": {
            "type": "keyword",
            "ignore_above": 256
          },
          "og:title": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "og:type": {
            "type": "keyword",
            "ignore_above": 128
          },
          "og:url": {
            "type": "text",
            "analyzer": "simple",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "twitter:card": {
            "type": "keyword",
            "ignore_above": 128
          },
          "twitter:description": {
            "type": "text"
          },
          "twitter:image": {
            "type": "keyword",
            "ignore_above": 256
          },
          "twitter:site": {
            "type": "keyword",
            "ignore_above": 128
          },
          "twitter:title": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 128
              }
            }
          },
          "url": {
            "type": "text",
            "analyzer": "simple",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          }
        }
      },
      "request": {
        "type": "text",
        "analyzer": "simple",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          }
        }
      },
      "response": {
        "type": "keyword",
        "ignore_above": 128
      },
      "spaces": {
        "type": "text",
        "analyzer": "whitespace"
      },
      "url": {
        "type": "text",
        "analyzer": "simple",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          }
        }
      },
      "utc_time": {
        "type": "date"
      },
      "xss": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 512
          }
        }
      }
    }
  },
  "settings": {
    "number_of_shards": 4,
    "number_of_replicas": 3
  }
}

POST _reindex
{
  "source": {
    "index": "logs"
  },
  "dest": {
    "index": "logs_new"
  }
}
```