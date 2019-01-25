---
layout: post
title: Drop duplicate rows in postgresql
---

When adding a unique constrain in a postgresql database which is not empty adding the constraint will fail if you have duplicate values. There are multiple solutions to detect the duplicate rows.
A pretty easy but postgresql dependend version is using the internal ctid (the internal row id) and the `USING` keywords. This works even if there is no unique key in your table.

So you can a statement before creation of the unique index

``` sql
DELETE
FROM tablea_to_tableb ta1 USING tablea_to_tableb ta2
WHERE fta1.ctid < fta2.ctid
  AND fta1.key1 = fta2.key1
  AND fta1.key2 = fta2.key2;
```

If you want to check if there are duplicate rows, you can create an sql statement using a select join on your table.
``` sql
SELECT fta1.*
FROM table2_to_tableb fta1
  , table2_to_tableb fta2
WHERE fta1.ctid < fta2.ctid          
  AND fta1.key1 = fta2.key1
  AND fta1.key2 = fta2.key2;
  ```
{% include twitter.html %}
