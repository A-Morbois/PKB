# 202003231610 ELK_CheatSheet
tags = #ELK



To NOT get the result from a query, use size =0
```
"body": {
  "size": 0,
  "query": {
    "bool": {
      "must": [
        {"range": {"Date": {"gte": "now-1d"}}},
        {"range": {"score": {"lte": 99}}}
      ]
    }
  }
}
```

`