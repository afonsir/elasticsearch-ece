```json
PUT romeo_and_juliet
{
  "mappings" : {
    "properties" : {
      "line_id" : {
        "type" : "integer"
      },
      "line_number" : {
        "type" : "text",
        "fields" : {
          "keyword" : {
            "type" : "keyword",
            "ignore_above" : 256
          }
        }
      },
      "play_name" : {
        "type" : "keyword"
      },
      "speaker" : {
        "type" : "keyword"
      },
      "speech_number" : {
        "type" : "integer"
      },
      "text_entry" : {
        "type" : "text",
        "fields" : {
          "keyword" : {
            "type" : "keyword",
            "ignore_above" : 256
          }
        }
      },
      "type" : {
        "type" : "text",
        "fields" : {
          "keyword" : {
            "type" : "keyword",
            "ignore_above" : 256
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
```

```json
PUT _ingest/pipeline/shakespeare-tokenizer
{
  "description": "Tokenizes the text_entry field into an array. Adds a word_count field. Removes the play_name field.",
  "processors": [
    {
      "split": {
        "field": "_source.text_entry",
        // Treat all consecutive whitespace characters as a single separator
        "separator": "\\s+",
        "target_field": "word_array"
      }
    },
    {
      "remove": {
        "field": "play_name"
      }
    },
    {
      "script": {
        "lang": "painless",
        "source": "ctx.word_count = ctx.word_array.length"
      }
    }
  ]
}
```

```json
POST _reindex
{
  "source": {
    "index": "shakespeare",
    "query": {
      "match": {
       "play_name": "Romeo and Juliet"
     }
    }
  },
  "dest": {
    "index": "romeo_and_juliet",
    "pipeline": "shakespeare-tokenizer"
  }
}
```
