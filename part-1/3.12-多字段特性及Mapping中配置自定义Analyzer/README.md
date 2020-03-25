# 多字段特性及Mapping中配置自定义Analyzer
## 课程Demo
```
#使用char filter进行替换
POST _analyze
{
  "tokenizer": "standard",
  "char_filter": [
      {
        "type" : "mapping",
        "mappings" : [ "- => _"]
      }
    ],
  "text": "123-456, I-test! test-990 650-555-1234"
}

#char filter 替换表情符号
POST _analyze
{
  "tokenizer": "standard",
  "char_filter": [
      {
        "type" : "mapping",
        "mappings" : [ ":) => happy", ":( => sad"]
      }
    ],
    "text": ["I am felling :)", "Feeling :( today"]
}

#white space and snowball
GET _analyze
{
  "tokenizer": "whitespace",
  "filter": ["stop","snowball"],
  "text": ["The gilrs in China are playing this game!"]
}


#whitespace与stop
GET _analyze
{
  "tokenizer": "whitespace",
  "filter": ["stop","snowball"],
  "text": ["The rain in Spain falls mainly on the plain."]
}


#remove 加入lowercase后，The被当成 stopword删除
GET _analyze
{
  "tokenizer": "whitespace",
  "filter": ["lowercase","stop","snowball"],
  "text": ["The gilrs in China are playing this game!"]
}

#正则表达式
GET _analyze
{
  "tokenizer": "standard",
  "char_filter": [
      {
        "type" : "pattern_replace",
        "pattern" : "http://(.*)",
        "replacement" : "$1"
      }
    ],
    "text" : "http://www.elastic.co"
}

# 自定义分词器
PUT /logs/
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "type": "custom",
          "char_filter":["my_char_filter"]
          "tokenizer": "my_tokenizer",
          "filter": [
            "lowercase"
          ]
        }
      },
      "tokenizer": {
        "my_tokenizer": {
          "type": "pattern",
          "pattern": "[!|&()]"
        }
      },
     "char_filter": {
        "my_char_filter": {
          "type": "pattern_replace",
          "pattern": """[!|&()\[\]]""",
          "replacement": " "
        }
      }
    }
  }
}

# 使用分词器
PUT /logs/_mapping
{
  "properties":{
    "level":{
      "type":"text",
      "analyzer" : "my_analyzer"
    },
    "password" : {
      "type" : "text"
    }
  }
}

#测试分词器
GET /logs/_analyze
{
  "analyzer": "my_analyzer",
  "text": "ABC|BCD&CC(CCC|DFFF)=>adffgagsd"
}
```

## 相关阅读
