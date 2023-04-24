# MAC - Laboratoire 3

This request is used for reindexing

```json
POST _reindex
{
  "source": {
    "index": "cacm_raw"
  },
  "dest": {
    "index": "cacm_dynamic",
    "pipeline": "csv-split-remove"
  }
}
```

Getting all data from index

```curl
GET index_name/_search?pretty
```

## Indexing

### D.1

```json
PUT /cacm_standard
{
  "mappings": {
    "properties": {
      "id": {
        "type":"unsigned_long",
        "index":false
      },
      "authors": {
        "type": "keyword"
      },
      "title": {
        "type": "text",
		"fielddata": true
      },
      "date": {
        "type": "date"
      },
      "summary": {
        "type": "text",
        "fielddata": true,
        "index_options": "offsets"
      }
    }
  }
}
```

### D.2

```json
PUT /cacm_termvector
{
  "mappings": {
    "properties": {
      "id": {
        "type":"unsigned_long",
        "index":false
      },
      "authors": {
        "type": "keyword"
      },
      "title": {
        "type": "text",
		"fielddata": true
      },
      "date": {
        "type": "date"
      },
      "summary": {
        "type": "text",
        "fielddata": true,
		"index_options": "offsets",
        "term_vector": "with_positions_offsets"
      }
    }
  }
}
```

### D.3

```json
GET /cacm_termvector/_termvectors/<_id>
```

### D.4

A term vector refers as a vector contraining multiple informations regarding terms in a text field. It can store for each its term frequency, its position in the text or its length.

### D.5

It nearly doubles the size of the index. Since every summary field of each document has its term vector, we can assume the storage is like doubling the field itself because it must process every word. We don't exactly double the size of the index however because not every docs have a summary field and also because some terms a repeated (like "a", "the", "this", etc..)

## Reading Index

### D.6

```json
GET cacm_standard/_search
{
  "aggs": {
    "most_published_author": {
      "terms": {
        "field": "authors",
        "size": 1
      }
    }
  }
}
```

The most published author is Thacher Jr., H. C. whose name appears in 38 docs


## D.7

Request:
```json
GET cacm_standard/_search
{
  "aggs": {
    "top_ten_terms_in_title": {
      "terms": {
        "field": "title",
        "size": 10
      }
    }
  }
}
```

Result:
Note that the term frequency is found under the property "doc_count"
```json
[
	{
	  "key": "of",
	  "doc_count": 1138
	},
	{
	  "key": "algorithm",
	  "doc_count": 975
	},
	{
	  "key": "a",
	  "doc_count": 895
	},
	{
	  "key": "for",
	  "doc_count": 714
	},
	{
	  "key": "the",
	  "doc_count": 645
	},
	{
	  "key": "and",
	  "doc_count": 434
	},
	{
	  "key": "in",
	  "doc_count": 416
	},
	{
	  "key": "on",
	  "doc_count": 340
	},
	{
	  "key": "an",
	  "doc_count": 275
	},
	{
	  "key": "computer",
	  "doc_count": 275
	}
]
```