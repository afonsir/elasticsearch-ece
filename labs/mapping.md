```json
PUT _template/customers
{
  "aliases": {
    "customers": {}
  },
  "index_patterns": [ "customers-*" ],
  "mappings": {
    "properties": {
      "year_to_date": {
        "type": "double"
      }
    },
    "dynamic_templates": [
      {
        "long_to_integer": {
          "match_mapping_type": "long",
          "mapping": {
            "type": "integer"
          }
        }
      }
    ]
  },
  "settings": {
    "number_of_shards": 4,
    "number_of_replicas": 3
  }
}
```

```json
PUT _template/partners
{
  "aliases": {
    "partners": {}
  },
  "index_patterns": [ "partners-*" ],
  "mappings": {
    "properties": {
      "address": {
        "type": "text"
      }
    },
    "dynamic_templates": [
      {
        "string_to_keyword": {
          "match_mapping_type": "string",
          "mapping": {
            "type": "keyword"
          }
        }
      }
    ]
  },
  "settings": {
    "number_of_shards": 4,
    "number_of_replicas": 3
  }
}
```

```json
PUT _template/leads
{
  "aliases": {
    "leads": {}
  },
  "index_patterns": [ "leads-*" ],
  "mappings": {
    "properties": {
      "address": {
        "type": "text"
      },
      "estimate": {
        "type": "double"
      }
    },
    "dynamic_templates": [
      {
        "string_to_keyword": {
          "match_mapping_type": "string",
          "match": "lead_*",
          "unmatch": "*_text",
          "mapping": {
            "type": "keyword"
          }
        }
      }
    ]
  },
  "settings": {
    "number_of_shards": 4,
    "number_of_replicas": 3
  }
}
```
