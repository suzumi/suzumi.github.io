---
title: テスト記事
date: 2017-01-28 02:50:22
tags:
---


はじめまして！

```go
import (
    "github.com/jinzhu/gorm"
    _ "github.com/jinzhu/gorm/dialects/mysql"
)
...
toml.DecodeFile("config.toml", &config)
dbconf := config.Mysql.User + ":" + config.Mysql.Pass + "@tcp(" + config.Mysql.Host+ ":3306)/" + config.Mysql.DB
db, err = gorm.Open("mysql", dbconf)
if err != nil {
    panic(err.Error())
}
defer db.Close() // 関数がリターンする直前に呼び出される
```