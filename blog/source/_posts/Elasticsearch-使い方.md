---
title: Elasticsearch 使い方
date: 2020-12-19 23:38:40
tags:
---
# 実行環境 
- Elasticsearch 7.10.1
- Kibana 7.10.1
# 検索

## 完全一致検索
データ項目のタイプがキーワードとして登録されている必要があります。  
検索対象の`type`が`keyword`で、`query`で`term`を指定して行う必要があります。
### request
```
#対象index作成 都道府県(pref)のtypeを完全一致の対象にする為にkeywordで登録する
PUT search_job
{
  "mappings" : {
    "properties": {
      "pref": {
        "type": "keyword"
      }
    }
  }
}

#type確認
GET search_job/_mapping
```

### response
```
{
  "search_job" : {
    "mappings" : {
      "properties" : {
        "pref" : {
          "type" : "keyword"
        }
      }
    }
  }
}
```

### request
```
データ登録
POST /search_job/_doc
{
  "pref" : "東京" 
}
# データ確認
GET /search_job/_search
```

### response
```
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "search_job",
        "_type" : "_doc",
        "_id" : "jS17e3YB0KvFjllCp3-y",
        "_score" : 1.0,
        "_source" : {
          "pref" : "東京"
        }
      }
    ]
  }
}
```

### request
```
# queryの種類をtermにすることで完全一致検索
GET /search_job/_search
{
  "query" : {
        "term": {
          "pref":"東京"
        }
  }
}
```

### response
```
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 0.2876821,
    "hits" : [
      {
        "_index" : "search_job",
        "_type" : "_doc",
        "_id" : "jS17e3YB0KvFjllCp3-y",
        "_score" : 0.2876821,
        "_source" : {
          "pref" : "東京"
        }
      }
    ]
  }
}
```

### request
```
# queryの種類をtermにすることで完全一致検索
GET /search_job/_search
{
  "query" : {
        "term": {
          "pref":"東"
        }
  }
}
```

### response
```
# 該当するものは無い
{
  "took" : 2,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 0,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  }
}
```

## indexを定義せずにデータ登録した場合

### request
```
POST /search_job2/_doc
{
  "pref" : "東京" 
}
```

### response
```
# typeでtextで登録されて、fieldsにtype:keywordでも登録されます
{
  "search_job2" : {
    "mappings" : {
      "properties" : {
        "pref" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        }
      }
    }
  }
}
```

### request
```
# termの対象をpref.keywordとすることで、fieldsのtype:keywordを対象とすることができます
GET /search_job2/_search
{
  "query" : {
    "term" : {
      "pref.keyword": "東京"
    }
  }
}
```

### response
```
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 0.2876821,
    "hits" : [
      {
        "_index" : "search_job2",
        "_type" : "_doc",
        "_id" : "ki2Ne3YB0KvFjllCJX_C",
        "_score" : 0.2876821,
        "_source" : {
          "pref" : "東京"
        }
      }
    ]
  }
}
```

## 複数検索条件の完全一致
queryの`bool`の`must`で複数`term`を指定する
```
#request
PUT search_job3
{
  "mappings" : {
    "properties": {
      "pref": {
        "type": "keyword"
      },
      "employment": {
        "type": "keyword"
      }
    }
  }
}
#response
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "search_job3"
}
#request
GET search_job3/_mapping
#response
{
  "search_job3" : {
    "mappings" : {
      "properties" : {
        "employment" : {
          "type" : "keyword"
        },
        "pref" : {
          "type" : "keyword"
        }
      }
    }
  }
}
#request
POST /search_job3/_doc
{
  "pref" : "東京",
  "employment" : "full-time"
}
#response
{
  "_index" : "search_job3",
  "_type" : "_doc",
  "_id" : "lS1OfXYB0KvFjllCZ3-D",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}
#request
GET /search_job3/_search
#response
{
  "took" : 484,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "search_job3",
        "_type" : "_doc",
        "_id" : "lS1OfXYB0KvFjllCZ3-D",
        "_score" : 1.0,
        "_source" : {
          "pref" : "東京",
          "employment" : "full-time"
        }
      }
    ]
  }
}

#request
#複数検索条件の完全一致
GET /search_job3/_search
{
  "query" : {
    "bool": {
      "must" : [
          {
            "term": {
              "pref": "東京"
            }
          },
          {
            "term": {
              "employment": "full-time"
            }
          }
        ]
    }
  }
}
#response
{
  "took" : 2,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 0.5753642,
    "hits" : [
      {
        "_index" : "search_job3",
        "_type" : "_doc",
        "_id" : "lS1OfXYB0KvFjllCZ3-D",
        "_score" : 0.5753642,
        "_source" : {
          "pref" : "東京",
          "employment" : "full-time"
        }
      }
    ]
  }
}
```

## ネストした検索対象への完全一致検索
```
#  路線が駅を持つようなネストしたデータを作成します
# request
PUT search_job4
{
  "mappings" : {
    "properties": {
      "pref": {
        "type": "keyword"
      },
      "employment": {
        "type": "keyword"
      },
      "line": {
        "properties" : {
          "name": {
            "type": "keyword"
          },
          "stations" : {
            "properties" : {
              "name" : {
                "type" : "keyword"
              }
            }
          }
        }
      }
    }
  }
}

POST /search_job4/_doc
{
  "pref" : "東京",
  "employment" : "full-time",
  "line" : {
    "name" : "山手線",
    "stations" : [
      {
        "name" : "渋谷"    
      },
      {
        "name" : "恵比寿"    
      },
      {
        "name" : "品川"    
      }
    ]
  }
}

GET /search_job4/_search

# response
# 路線が駅を持つようなネストしたデータが登録できました
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "search_job4",
        "_type" : "_doc",
        "_id" : "li1_fXYB0KvFjllCmn_A",
        "_score" : 1.0,
        "_source" : {
          "pref" : "東京",
          "employment" : "full-time",
          "line" : {
            "name" : "山手線",
            "stations" : [
              {
                "name" : "渋谷"
              },
              {
                "name" : "恵比寿"
              },
              {
                "name" : "品川"
              }
            ]
          }
        }
      }
    ]
  }
}

# resquest
# ネストされている駅名に対して完全一致検索をします
GET /search_job4/_search
{
  "query" : {
    "term": {
      "line.stations.name": "渋谷"
    }
  }
}
```

## ネストした検索対象にネスト構造の経路の関係(`Nested datatype`)をもたせる
ネストした経路の関係(`Nested datatype`)を持たせたい場合にtypeに`nested`指定してmappingをします。

先程のmappingだと、何線の何駅かというのは検索できません。  
下記のようなデータを追加した場合。  
山手線の渋谷駅というデータを絞りこめません。  
```
# request
POST /search_job4/_doc
{
  "pref" : "東京",
  "employment" : "full-time",
  "line" : [
    {
      "name" : "東横線",
      "stations" : [
        {
          "name" : "渋谷"    
        },
        {
          "name" : "中目黒"    
        },
        {
          "name" : "横浜"    
        }
      ]
    },
    {
      "name" : "山手線",
      "stations" : [
        {
          "name" : "東京"    
        },
        {
          "name" : "神田"    
        },
        {
          "name" : "浅草"    
        }
      ]
    }
  ]
}

GET /search_job4/_search

# response
{
  "took" : 2,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "search_job4",
        "_type" : "_doc",
        "_id" : "li1_fXYB0KvFjllCmn_A",
        "_score" : 1.0,
        "_source" : {
          "pref" : "東京",
          "employment" : "full-time",
          "line" : {
            "name" : "山手線",
            "stations" : [
              {
                "name" : "渋谷"
              },
              {
                "name" : "恵比寿"
              },
              {
                "name" : "品川"
              }
            ]
          }
        }
      },
      {
        "_index" : "search_job4",
        "_type" : "_doc",
        "_id" : "mC2sfXYB0KvFjllCHn8N",
        "_score" : 1.0,
        "_source" : {
          "pref" : "東京",
          "employment" : "full-time",
          "line" : [
            {
              "name" : "東横線",
              "stations" : [
                {
                  "name" : "渋谷"
                },
                {
                  "name" : "中目黒"
                },
                {
                  "name" : "横浜"
                }
              ]
            },
            {
              "name" : "山手線",
              "stations" : [
                {
                  "name" : "東京"
                },
                {
                  "name" : "神田"
                },
                {
                  "name" : "浅草"
                }
              ]
            }
          ]
        }
      }
    ]
  }
}

# request
# 山手線の渋谷駅で検索
GET /search_job4/_search
{
  "query" : {
    "bool": {
      "must" : [
          {
            "term": {
              "line.name": "山手線"
            }
          },
          {
            "term": {
              "line.stations.name": "渋谷"
            }
          }
        ]
    }
  }
}

# response
# 山手線の渋谷駅でないものも含まれてしまう。
{
  "took" : 0,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 0.47851416,
    "hits" : [
      {
        "_index" : "search_job4",
        "_type" : "_doc",
        "_id" : "li1_fXYB0KvFjllCmn_A",
        "_score" : 0.47851416,
        "_source" : {
          "pref" : "東京",
          "employment" : "full-time",
          "line" : {
            "name" : "山手線",
            "stations" : [
              {
                "name" : "渋谷"
              },
              {
                "name" : "恵比寿"
              },
              {
                "name" : "品川"
              }
            ]
          }
        }
      },
      {
        "_index" : "search_job4",
        "_type" : "_doc",
        "_id" : "mC2sfXYB0KvFjllCHn8N",
        "_score" : 0.47851416,
        "_source" : {
          "pref" : "東京",
          "employment" : "full-time",
          "line" : [
            {
              "name" : "東横線",
              "stations" : [
                {
                  "name" : "渋谷"
                },
                {
                  "name" : "中目黒"
                },
                {
                  "name" : "横浜"
                }
              ]
            },
            {
              "name" : "山手線",
              "stations" : [
                {
                  "name" : "東京"
                },
                {
                  "name" : "神田"
                },
                {
                  "name" : "浅草"
                }
              ]
            }
          ]
        }
      }
    ]
  }
}
```
これは、追加したデータがElasticsearchの内部ではネスト構造を保たず、項目ごとにグルーピングされたような構造(`Object datatype`)で解釈される為になります。
```
{
    "line" : {
        "name" : [
            "東横線",
            "山手線"
        ],
        "stations" : {
            "name" : [
                "渋谷",
                "中目黒",
                "横浜",
                "東京",
                "神田",
                "浅草"
            ]
        }
    }
}
```

路線の`type`を`nested`にしてネスト構造を保ったままmappingし、ネスト構造の経路を指定して検索してみる。
```
# request
PUT search_job5
{
  "mappings" : {
    "properties": {
      "pref": {
        "type": "keyword"
      },
      "employment": {
        "type": "keyword"
      },
      "line": {
        "type" : "nested",
        "properties" : {
          "name": {
            "type": "keyword"
          },
          "stations" : {
            "properties" : {
              "name" : {
                "type" : "keyword"
              }
            }
          }
        }
      }
    }
  }
}

GET search_job5/_mapping
# response
# line(路線)のtypeがnestedになっていることがわかる
{
  "search_job5" : {
    "mappings" : {
      "properties" : {
        "employment" : {
          "type" : "keyword"
        },
        "line" : {
          "type" : "nested",
          "properties" : {
            "name" : {
              "type" : "keyword"
            },
            "stations" : {
              "properties" : {
                "name" : {
                  "type" : "keyword"
                }
              }
            }
          }
        },
        "pref" : {
          "type" : "keyword"
        }
      }
    }
  }
}

# request
POST /search_job5/_doc
{
  "pref" : "東京",
  "employment" : "full-time",
  "line" : {
    "name" : "山手線",
    "stations" : [
      {
        "name" : "渋谷"    
      },
      {
        "name" : "恵比寿"    
      },
      {
        "name" : "品川"    
      }
    ]
  }
}

POST /search_job5/_doc
{
  "pref" : "東京",
  "employment" : "full-time",
  "line" : [
    {
      "name" : "東横線",
      "stations" : [
        {
          "name" : "渋谷"    
        },
        {
          "name" : "中目黒"    
        },
        {
          "name" : "横浜"    
        }
      ]
    },
    {
      "name" : "山手線",
      "stations" : [
        {
          "name" : "東京"    
        },
        {
          "name" : "神田"    
        },
        {
          "name" : "浅草"    
        }
      ]
    }
  ]
}

GET /search_job5/_search
# response
{
  "took" : 2,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "search_job5",
        "_type" : "_doc",
        "_id" : "mS3wfnYB0KvFjllCcn9x",
        "_score" : 1.0,
        "_source" : {
          "pref" : "東京",
          "employment" : "full-time",
          "line" : {
            "name" : "山手線",
            "stations" : [
              {
                "name" : "渋谷"
              },
              {
                "name" : "恵比寿"
              },
              {
                "name" : "品川"
              }
            ]
          }
        }
      },
      {
        "_index" : "search_job5",
        "_type" : "_doc",
        "_id" : "mi3wfnYB0KvFjllCk3_f",
        "_score" : 1.0,
        "_source" : {
          "pref" : "東京",
          "employment" : "full-time",
          "line" : [
            {
              "name" : "東横線",
              "stations" : [
                {
                  "name" : "渋谷"
                },
                {
                  "name" : "中目黒"
                },
                {
                  "name" : "横浜"
                }
              ]
            },
            {
              "name" : "山手線",
              "stations" : [
                {
                  "name" : "東京"
                },
                {
                  "name" : "神田"
                },
                {
                  "name" : "浅草"
                }
              ]
            }
          ]
        }
      }
    ]
  }
}

# request
# nestedのpathにlineを指定し、line(路線)のネスト構造の経路内で山手線の渋谷を検索する
GET /search_job5/_search
{
  "query": {
    "nested": {
      "path": "line",
      "query" : {
        "bool": {
          "must" : [
            {
              "term": {
                "line.name": "山手線"
              }
            },
            {
              "term": {
                  "line.stations.name": "渋谷"
              }
            }
          ]
        }
      }
    }
  }
}

# response
{
  "took" : 4,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.1162586,
    "hits" : [
      {
        "_index" : "search_job5",
        "_type" : "_doc",
        "_id" : "mS3wfnYB0KvFjllCcn9x",
        "_score" : 1.1162586,
        "_source" : {
          "pref" : "東京",
          "employment" : "full-time",
          "line" : {
            "name" : "山手線",
            "stations" : [
              {
                "name" : "渋谷"
              },
              {
                "name" : "恵比寿"
              },
              {
                "name" : "品川"
              }
            ]
          }
        }
      }
    ]
  }
}

```


# 参考url
- https://qiita.com/NAO_MK2/items/630f2c4caa0e8a42407c
- https://medium.com/hello-elasticsearch/elasticsearch-nested-type-vs-array-objects-2ea7bac68ed8
- https://mattintosh.hatenablog.com/entry/20190226/1551181539