
# 添加
```json
PUT /movies/movie/1
{
    "title": "The Godfather",
    "director": "Francis Ford Coppola",
    "year": 1972
}
```

# 返回信息
{
  "_index": "movies",
  "_type": "movie",
  "_id": "1",
  "_version": 1,
  "result": "created",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 4,
  "_primary_term": 1
}

# 第二次执行添加操作后，就是更新操作updated
PUT /movies/movie/1
{
    "title": "The Godfather",
    "director": "Francis Ford Coppola",
    "year": 1972,
    "genres": ["Crime", "Drama"]
}
{
  "_index": "movies",
  "_type": "movie",
  "_id": "1",
  "_version": 2,
  "result": "updated",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 5,
  "_primary_term": 1
}


# 搜索
# 搜索所有
GET _search
{
  "query": {
    "match_all": {}
  }
}
# 按需指定搜索
GET /movies/movie/1



# 删除操作
DELETE /movies/movie/1
# 返回信息
{
  "_index": "movies",
  "_type": "movie",
  "_id": "1",
  "_version": 3,
  "result": "deleted",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 6,
  "_primary_term": 1
}

#删除完了，再检索一次
GET /movies/movie/1
# 返回的数据
{
  "_index": "movies",
  "_type": "movie",
  "_id": "1",
  "found": false
}








# 添加多组数据
PUT /movies/movie/1
{
    "title": "The Godfather",
    "director": "Francis Ford Coppola",
    "year": 1972,
    "genres": ["Crime", "Drama"]
}



PUT /movies/movie/2
{
    "title": "Lawrence of Arabia",
    "director": "David Lean",
    "year": 1962,
    "genres": ["Adventure", "Biography", "Drama"]
}

PUT /movies/movie/3
{
    "title": "To Kill a Mockingbird",
    "director": "Robert Mulligan",
    "year": 1962,
    "genres": ["Crime", "Drama", "Mystery"]
}

PUT /movies/movie/4
{
    "title": "Apocalypse Now",
    "director": "Francis Ford Coppola",
    "year": 1979,
    "genres": ["Drama", "War"]
}

PUT /movies/movie/5
{
    "title": "Kill Bill: Vol. 1",
    "director": "Quentin Tarantino",
    "year": 2003,
    "genres": ["Action", "Crime", "Thriller"]
}

PUT /movies/movie/6
{
    "title": "The Assassination of Jesse James by the Coward Robert Ford",
    "director": "Andrew Dominik",
    "year": 2007,
    "genres": ["Biography", "Crime", "Drama"]
}



# _search端点
使用_search端点，可选择使用索引和类型。也就是说，按照以下模式向URL发出请求：<index>/<type>/_search。其中，index和type都是可选的。

POST _search   # 搜索 所有索引 和 所有类型。

POST /movies/_search # 在 电影索引 中搜索所有类型

POST /movies/movie/_search # 在 电影索引 中显式搜索 电影类型 的文档。

# 搜索请求正文和ElasticSearch查询DSL
为了创建更有用的搜索请求，还需要向请求正文中提供查询。 请求正文是一个JSON对象，除了其它属性以外，它还要包含一个名称为“query”的属性，这就可使用ElasticSearch的查询DSL。
{
    "query": {
        //Query DSL here
    }
}
你可能想知道查询DSL是什么。它是ElasticSearch自己基于JSON的域特定语言，可以在其中表达查询和过滤器


# 基本自由文本搜索
查询DSL具有一长列不同类型的查询可以使用。 对于“普通”自由文本搜索，最有可能想使用一个名称为“查询字符串查询”。
查询字符串查询是一个高级查询，有很多不同的选项，ElasticSearch将解析和转换为更简单的查询树。如果忽略了所有的可选参数，并且只需要给它一个字符串用于搜索，它可以很容易使用。
现在尝试在两部电影的标题中搜索有“kill”这个词的电影信息：
POST /movies/_search
{
    "query": {
        "query_string": {
            "query": "kill"
        }
    }
}
返回两条数据
{
  "took": 0,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 2,
    "max_score": 0.2876821,
    "hits": [
      {
        "_index": "movies",
        "_type": "movie",
        "_id": "5",
        "_score": 0.2876821,
        "_source": {
          "title": "Kill Bill: Vol. 1",
          "director": "Quentin Tarantino",
          "year": 2003,
          "genres": [
            "Action",
            "Crime",
            "Thriller"
          ]
        }
      },
      {
        "_index": "movies",
        "_type": "movie",
        "_id": "3",
        "_score": 0.2876821,
        "_source": {
          "title": "To Kill a Mockingbird",
          "director": "Robert Mulligan",
          "year": 1962,
          "genres": [
            "Crime",
            "Drama",
            "Mystery"
          ]
        }
      }
    ]
  }
}



# 指定搜索的字段
在前面的例子中，使用了一个非常简单的查询，一个只有一个属性“query”的查询字符串查询。 如前所述，查询字符串查询有一些可以指定设置，如果不使用，它将会使用默认的设置值。
这样的设置称为“fields”，可用于指定要搜索的字段列表。如果不使用“fields”字段，ElasticSearch查询将默认自动生成的名为“_all”的特殊字段，来基于所有文档中的各个字段匹配搜索。
为了做到这一点，修改以前的搜索请求正文，以便查询字符串查询有一个fields属性用来要搜索的字段数组：
POST /movies/_search
{
    "query": {
        "query_string": {
            "query": "ford",
            "fields": ["title"]
        }
    }
}
返回一条数据
{
  "took": 0,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 0.69607234,
    "hits": [
      {
        "_index": "movies",
        "_type": "movie",
        "_id": "6",
        "_score": 0.69607234,
        "_source": {
          "title": "The Assassination of Jesse James by the Coward Robert Ford",
          "director": "Andrew Dominik",
          "year": 2007,
          "genres": [
            "Biography",
            "Crime",
            "Drama"
          ]
        }
      }
    ]
  }
}

正如预期的得到一个命中，电影的标题中的单词“ford”，意思是 属性为title中的值中包含“Ford”这个单词才可以。
现在，从查询中移除fields属性，应该能匹配到 3 行数据：
POST /movies/_search
{
    "query": {
        "query_string": {
            "query": "ford"
        }
    }
}
返回三条数据
{
  "took": 0,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 3,
    "max_score": 0.8781843,
    "hits": [
      {
        "_index": "movies",
        "_type": "movie",
        "_id": "4",
        "_score": 0.8781843,
        "_source": {
          "title": "Apocalypse Now",
          "director": "Francis Ford Coppola",
          "year": 1979,
          "genres": [
            "Drama",
            "War"
          ]
        }
      },
      {
        "_index": "movies",
        "_type": "movie",
        "_id": "6",
        "_score": 0.69607234,
        "_source": {
          "title": "The Assassination of Jesse James by the Coward Robert Ford",
          "director": "Andrew Dominik",
          "year": 2007,
          "genres": [
            "Biography",
            "Crime",
            "Drama"
          ]
        }
      },
      {
        "_index": "movies",
        "_type": "movie",
        "_id": "1",
        "_score": 0.2876821,
        "_source": {
          "title": "The Godfather",
          "director": "Francis Ford Coppola",
          "year": 1972,
          "genres": [
            "Crime",
            "Drama"
          ]
        }
      }
    ]
  }
}

# 过滤















