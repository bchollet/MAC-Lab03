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
  "size": 0,
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
  "size": 0,
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

## Using different Analyzers

### D.8

Whitespace analyzer

```json
PUT /cacm_whitespace
{
  "settings": {
    "analysis": {
      "analyzer": {
        "whitespace_analyzer":{
          "tokenizer": "whitespace",
          "filter": []
        }
      }
    }
  }, 
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
		    "fielddata": true,
		    "analyzer": "whitespace_analyzer"
      },
      "date": {
        "type": "date"
      },
      "summary": {
        "type": "text",
        "fielddata": true,
        "index_options": "offsets",
		    "analyzer": "whitespace_analyzer"
      }
    }
  }
}
```

English analyzer

```json
PUT /cacm_english
{
  "settings": {
    "analysis": {
      "filter": {
        "english_stop": {
          "type": "stop",
          "stopwords": "_english_" 
        },
        "english_keywords": {
          "type": "keyword_marker",
          "keywords": [] 
        },
        "english_stemmer": {
          "type": "stemmer",
          "language": "english"
        },
        "english_possessive_stemmer": {
          "type": "stemmer",
          "language": "possessive_english"
        }
      },
      "analyzer": {
        "english_analyzer": {
          "tokenizer": "standard",
          "filter": [
            "english_possessive_stemmer",
            "lowercase",
            "english_stop",
            "english_keywords",
            "english_stemmer"
          ]
        }
      }
    }
  }, 
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
		    "fielddata": true,
		    "analyzer": "english_analyzer"
      },
      "date": {
        "type": "date"
      },
      "summary": {
        "type": "text",
        "fielddata": true,
        "index_options": "offsets",
		    "analyzer": "english_analyzer"
      }
    }
  }
}
``` 

Custom with lowercase filter, standard tokenizer and output shingles of size 1 and 2

```json
PUT /cacm_standard_custom1
{
  "settings": {
    "analysis": {
      "filter": {
        "shingle_custom1" : {
          "type": "shingle",
          "min_shingle_size": 2,
          "max_shingle_size": 2,
          "output_unigrams": true
        }
      }, 
      "analyzer": {
        "standard_custom1_analyzer": {
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "shingle_custom1"
          ]
        }
      }
    }
  }, 
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
		    "fielddata": true,
		    "analyzer": "standard_custom1_analyzer"
      },
      "date": {
        "type": "date"
      },
      "summary": {
        "type": "text",
        "fielddata": true,
        "index_options": "offsets",
		    "analyzer": "standard_custom1_analyzer"
      }
    }
  }
}
```

Custom with lowercase filter, standard tokenizer and output shingles of size 3

```json
PUT /cacm_standard_custom2
{
  "settings": {
    "analysis": {
      "filter": {
        "shingle_custom2" : {
          "type": "shingle",
          "min_shingle_size": 3,
          "max_shingle_size": 3,
          "output_unigrams": false
        }
      },
      "analyzer": {
        "standard_custom2_analyzer": {
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "shingle_custom2"
          ]
        }
      }
    }
  }, 
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
		    "fielddata": true,
		    "analyzer": "standard_custom2_analyzer"
      },
      "date": {
        "type": "date"
      },
      "summary": {
        "type": "text",
        "fielddata": true,
        "index_options": "offsets",
		    "analyzer": "standard_custom2_analyzer"
      }
    }
  }
}
```

Stop analyzer with custom stop words

```json
PUT /cacm_stop
{
  "settings": {
    "analysis": {
      "analyzer": {
        "stop_analyzer": {
          "type": "stop",
          "stopwords_path": "data/common_words.txt"
        }
      }
    }
  }, 
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
		    "fielddata": true,
		    "analyzer": "stop_analyzer"
      },
      "date": {
        "type": "date"
      },
      "summary": {
        "type": "text",
        "fielddata": true,
        "index_options": "offsets",
		    "analyzer": "stop_analyzer"
      }
    }
  }
}
```

### D.9

Whitespace: Breaks text into terms each time a whitespace is met. No lowercase filter is applied by default, so terms are stored as identically as they appear in the text.

English: Applies a set of filters regarding english language. In addition to the lowercase filter (which format every characters to lowercase), the filters consist of stem words (Like "magic" is a stem word for "magical" and "magically"), stop words (like "the", "is", ...), and possessive stem words (like "boy" is the possessive stem word of "boy's"). These filters are specific to the language they are set. In our case, those filters are specific to english only.

Custom Standard 1: The standard analyzer applies some basic filters like lowercase, stopwords and removing punctuation. To those filters we added a shingle one that will group words by group of 2 in addition to every isolated terms. For example "Hello World" will produce \["hello", "world", "hello world"\].

Custom Standard 2: The same as before, but here we set shingles to be of size 3 and exclusively of size 3. For example "It is raining" will produce \["it is raining"\]

Stop analyzer: It is a combination of the simple analyzer which breaks the text into token at any non-letter character (numbers, space, hyphen, apostrophes), discards non-letter characters and apply a lowercase filter. In addition to this, the stop analyzer allow a list of words that should not be indexed, these words are called stop words.


#### D.10

| Analyzer         	   | Number of indexed doc | Number of indexed terms in summary | Size of index | Time for indexing |
| :------------------  | :-------------------: | :--------------------------------: | :-----------: | :---------------: |
|`whitespace`  		   | 3202				   | 103275								| 1.78mb		| 298				|
|`english`	   		   | 3202				   | 72298								| 1.48mb		| 665				|
|shingle 1-2    	   | 3202				   | 237189								| 3.3 mb		| 677				|
|shingle 3 			   | 3202				   | 144518								| 3.63mb		| 968				|
|`stop`				   | 3202				   | 59988								| 1.48mb		| 787				|

Heres the top ten terms for each analyzer

| N°     | `whitespace` | `english` | shingle 1-2 | shingle 3 			   | `stop` 	 |
| :----: | :----------- | :-------- | :---------- | :--------------------- | :---------- |
|1		 |`of`			|`which`    |`the`	 	  |`in this paper` 		   | `computer`  |
|2		 |`the`			|`us`		|`of` 	 	  |`the use of`    		   | `system`	 |
|3		 |`is`			|`comput`   |`a`  	 	  |`the number of` 		   | `paper`	 |
|4		 |`and`			|`program`  |`is` 	 	  |`it is shown`   		   | `presented` |
|5		 |`a`			|`system`   |`and`	 	  |`a set of`      		   | `time`	  	 |
|6		 |`to`			|`present`  |`to`  	 	  |`in terms of`   		   | `program`	 |
|7		 |`in`			|`describ`  |`in` 	 	  |`the problem of`	   	   | `data`	  	 |
|8		 |`for`			|`paper`    |`for`	 	  |`is shown that` 		   | `method`	 |
|9		 |`The`			|`can`		|`are`	 	  |`a number of`   	   	   | `algorithm` |
|10		 |`are`			|`gener`    |`of the`	  |`as well as`    		   | `discussed` |
