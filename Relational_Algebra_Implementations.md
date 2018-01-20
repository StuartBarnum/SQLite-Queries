Relational Algebra Implementations
================

Selected exercises from Coursera's Data Manipulation at Scale (University of Washington) covering relational algebra and relational databases.

``` r
library(DBI)
reuters <- dbConnect(RSQLite::SQLite(), dbname = "reuters.db")
```

Coursera exercise: Write an SQL statement that is equivalent to the following relational algebra expression:

πterm(σdocid=10398\_txt\_earn and count=1(frequency)) U πterm(σdocid=925\_txt\_trade and count=1(frequency))

``` sql

SELECT count(*) FROM (
    SELECT term FROM frequency
    WHERE docid = '10398_txt_earn'
    AND count = 1
    UNION SELECT term FROM frequency
    WHERE docid = '925_txt_trade'
    AND count = 1);
```

| count(\*) |
|:----------|
| 324       |

Coursera exercise: Write an SQL statement to count the number of documents containing the word "parliament."

``` sql

SELECT count(*) FROM (
    SELECT docid FROM frequency
    WHERE term = 'law'
    AND count > 0
    UNION SELECT docid FROM frequency
    WHERE term = 'legal'
    AND count > 0);
```

| count(\*) |
|:----------|
| 58        |

Coursera exercise: Write an SQL statement to find all documents that have more than 300 total terms, including duplicate terms.

``` sql

SELECT COUNT(*) FROM (  
    SELECT docid, Total FROM (
        SELECT docid, SUM(count/count) as Total FROM frequency
        GROUP BY docid
        ORDER BY docid)
        WHERE Total > 300
);
```

| COUNT(\*) |
|:----------|
| 11        |

Coursera exercise: Write an SQL statement to count the number of unique documents that contain both the word 'transactions' and the word 'world'.

``` sql

SELECT COUNT(*) FROM (
    
    SELECT docid From frequency
    WHERE TERM = 'transactions'
    
    INTERSECT
    
    SELECT docid From frequency
    WHERE TERM = 'world'
);
```

| COUNT(\*) |
|:----------|
| 3         |
