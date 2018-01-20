Matrix Multiplication for Text Analytics
================
Stuart Barnum
12/29/2017

Selected text analytics exercises from the Coursera course Data Manipulation at Scale (from the University of Washington)
-------------------------------------------------------------------------------------------------------------------------

``` r
#Requires that reuters.db and matrix.db be in the working directory

library(RSQLite)
library(DBI)

matrix <- dbConnect(RSQLite::SQLite(), dbname = "matrix.db")

reuters <- dbConnect(RSQLite::SQLite(), dbname = "reuters.db")
```

The contents of any matrix can be specified in a three column table, in which the first column contains specifications of row numbers of the matrix, the second column contains specifications of column numbers of the matrix, and the third column contains the entries of the matrix. For example, if the entry of the matrix at (matrix) row j and (matrix) column k is m, then the three-column table contains the row (j, k, m). This method can provide an efficient and useful means of representing very large but "sparse" matrices, in which most of the entries are zero (in which case only the nonzero entries need be listed in the table).

The following query conducts "matrix multiplication" on matrices thus speicified in a database. (Although the matrices multiplied, A and B, are within matrix.db, the query works for any two matrices A and B in the three-collumn format, so long as its columns are suitably labeled.)

Coursera exercise: Express A X B as an SQL query

``` sql

SELECT row, col, SUM(product) FROM (
    SELECT a.row_num as row, b.col_num as col, a.value * b.value AS product
        FROM A a, B b
        WHERE a.col_num = b.row_num)
    GROUP BY row, col; 
```

|  row|  col|  SUM(product)|
|----:|----:|-------------:|
|    0|    0|         10284|
|    0|    1|          5221|
|    0|    2|           990|
|    0|    3|          1320|
|    0|    4|           234|
|    1|    0|          9825|
|    1|    1|          2482|
|    1|    2|            54|
|    1|    3|          1269|
|    1|    4|          1041|

Each column of the Reuters dataset corresponds to a word that occurs in at least one of the documents that is covered by the data, with each row corresponding to one of the documents. Each entry, then, gives the frequency of the word (corresponding to the column) in the document (corresponding to the row). Given this representation of the word counts, the product of the matrix (the dataset) with its transpose provides a measure of document similarity, based on word count. Thus the following exercise.

Coursera exercise: Write an SQL query that produces the product of the Reuters dataset (which has the form of a matrix) with its transpose.

``` sql

SELECT row, col, SUM(product) FROM (
    SELECT a.docid as row, b.docid as col, a.count * b.count AS product
        FROM frequency a, frequency b
        WHERE a.term = b.term)
    GROUP BY row, col;
        
```

| row              | col               |  SUM(product)|
|:-----------------|:------------------|-------------:|
| 10000\_txt\_earn | 10000\_txt\_earn  |           127|
| 10000\_txt\_earn | 10054\_txt\_earn  |            33|
| 10000\_txt\_earn | 10080\_txt\_crude |           146|
| 10000\_txt\_earn | 10088\_txt\_acq   |            11|
| 10000\_txt\_earn | 10094\_txt\_earn  |            25|
| 10000\_txt\_earn | 10097\_txt\_earn  |            34|
| 10000\_txt\_earn | 1009\_txt\_earn   |             6|
| 10000\_txt\_earn | 10102\_txt\_acq   |            11|
| 10000\_txt\_earn | 10114\_txt\_earn  |            74|
| 10000\_txt\_earn | 1011\_txt\_earn   |            51|

Find the best matching document to the keyword query "washington taxes treasury". Then, compute the similarity matrix again, but filter for only similarities involving the "query document": docid = 'q'.

``` sql

SELECT row, col, SUM(product) FROM (
    SELECT a.docid as row, b.docid as col, a.count * b.count AS product
        FROM (SELECT * FROM frequency
        UNION
        SELECT 'q' as docid, 'washington' as term, 1 as count 
        UNION
        SELECT 'q' as docid, 'taxes' as term, 1 as count
        UNION 
        SELECT 'q' as docid, 'treasury' as term, 1 as count) a, 
        (SELECT * FROM frequency
        UNION
        SELECT 'q' as docid, 'washington' as term, 1 as count 
        UNION
        SELECT 'q' as docid, 'taxes' as term, 1 as count
        UNION 
        SELECT 'q' as docid, 'treasury' as term, 1 as count) b
    WHERE a.term = b.term)
GROUP BY row, col;
```

| row              | col               |  SUM(product)|
|:-----------------|:------------------|-------------:|
| 10000\_txt\_earn | 10000\_txt\_earn  |           127|
| 10000\_txt\_earn | 10054\_txt\_earn  |            33|
| 10000\_txt\_earn | 10080\_txt\_crude |           146|
| 10000\_txt\_earn | 10088\_txt\_acq   |            11|
| 10000\_txt\_earn | 10094\_txt\_earn  |            25|
| 10000\_txt\_earn | 10097\_txt\_earn  |            34|
| 10000\_txt\_earn | 1009\_txt\_earn   |             6|
| 10000\_txt\_earn | 10102\_txt\_acq   |            11|
| 10000\_txt\_earn | 10114\_txt\_earn  |            74|
| 10000\_txt\_earn | 1011\_txt\_earn   |            51|
