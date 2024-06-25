---
title: "xDB: A relational DBMS based on persistent key-value storage"
summary: A RDBMS based on RocksDB
tags:
  - Compiler
  - Database
date: '2024-05-19T13:53:00Z'

# Optional external URL for project (replaces project detail page).
external_link: ''

links:
url_code: 'https://github.com/ruitianzhong/xDB'
url_pdf: ''
url_slides: ''
url_video: ''

# Slides (optional).
#   Associate this project with Markdown slides.
#   Simply enter your slide deck's filename without extension.
#   E.g. `slides = "example-slides"` references `content/slides/example-slides.md`.
#   Otherwise, set `slides = ""`.
# slides: example
---

[GitHub link](https://github.com/ruitianzhong/xDB)

xDB is a relational DBMS built upon persistent key-value storag written by [Ruitian Zhong](https://zrt.ink).

## Features

* Expression(nested) evaluation (including `+-*/%^`, `AND/OR`, `... BETWEEN ... AND ...`, `IS NULL`,`IS NOT NULL` on `CHAR` and `INTEGER`)
* SQL comment
* `NOT NULL` column constraint
* Supported Datatype: `INT`, `FLOAT`, `CHAR(N)`
* Multi-line support
* sql file execution(`./xDB --filepath="/path/to/example.sql"`)
* Line editing and sql history
* Select multiple tables(a.k.a., Cartesian product)
* Based on persistent key-value storage(built upon LSM-Tree) like [MyRocks](http://myrocks.io/) and [TiDB](https://docs.pingcap.com/zh/tidb/stable)

More details in `Supported SQL (Example)`.

## Supported SQL (Example)

```sql
CREATE DATABASE example;
USE example;
CREATE TABLE user (id int,score float);
SHOW TABLES;
INSERT INTO user (id int NOT NULL) VALUES (1);
SELECT id from user WHERE id = 42;
UPDATE user SET id=1 WHERE id=42;
DELETE FROM user WHERE id=42;
SELECT * from user where id=(1+2*2+(id=id)+id^id+id) AND id = id%2 AND id IS NOT NULL;
select * from t1  where id is not null;
DROP TABLE user;
```


## Demo

![](./images/1.png)
![](./images/2.png)
![](./images/3.png)
![](./images/4.png)
![](./images/5.png)
![](./images/6.png)
![](./images/7.png)
![](./images/8.png)
![](./images/9.png)
![](./images/10.png)
![](./images/11.png)
![](./images/12.png)
![](./images/13.png)
![](./images/14.png)
![](./images/15.png)
![](./images/16.png)
![](./images/17.png)
![](./images/18.png)