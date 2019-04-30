---
title: "Data 607 Assignment 13"
author: "Yohannes Deboch "
date: "04/29/2019"
output: 
  html_document: 
    keep_md: yes
---

-----

###  Instructions of the Assignment 

For this assignment, you should take information from a relational database and migrate it to a NoSQL database of your own choosing. 

For the relational database, you might use the flights database, the tb database, the "data skills" database your team created for Project 3, or another database of your own choosing or creation.

For the NoSQL database, you may use MongoDB (which we introduced in week 7), Neo4j, or another NoSQL database of your choosing.

Your migration process needs to be reproducible.  R code is encouraged, but not required.  You should also briefly describe the advantages and disadvantages of storing the data in a relational database vs. your NoSQL database.

### Migrating tables from MySQL to NoSQL database

I would like to start by briefly descibing about MYSQL & NOSQL. MySQL is the most popular open source relational database management system where as a NoSQL database is a non-relational database, which provides a mechanism for storage and retrieval of data. In the NoSQL database, data is modeled in means other than the tabular relations used in relational databases. MySQL database with its settled market encompasses a huge community whereas the NoSQL database with the short span arrival has a comparatively short community.
MySQL is not so easily scalable with their rigid schema restrictions whereas NoSQL can be easily scaled with their dynamic schema nature.
The detailed database model is required before database creation in MySQL whereas no detailed modeling is required in the case of NoSQL database types.
MySQL is one of the types of relational database whereas NoSQL is more of design based database type with examples like MongoDB, Couch DB, etc.
MySQL is available with a wide array of reporting tools help application's validity whereas NoSQL databases lack reporting tools for analysis and performance testing.
MySQL being a relational database is less flexible with its design constraint whereas NoSQL being non-relational in nature, provides a more flexible design as compared to MySQL.
MySQL is being used with a standard query language called SQL whereas NoSQL like databases misses a standard query language.
MySQL like a relational database can provide a performance issue for a huge amount of data, hence require optimization of queries whereas NoSQL databases like MongoDB are good at performance even with the dataset is huge in size.


For this assignment I will be doing a migration of MYSQL database having two tables (movies and movie ratings ).

-----




```r
library(RMySQL)
library(mongolite)
library(dbplyr)
library(dplyr)
library(knitr)
```



-----

### Connect to MySQL database running locally.



```r
con <- DBI::dbConnect(RMySQL::MySQL(), dbname = "data607_hw2", user=db_user, password=db_pw)

#get info on the database
dbplyr::src_dbi(con)
```

```
## src:  mysql 5.7.21-log [root@localhost:/data607_hw2]
## tbls: movie_ratings, movies
```

-----

### Query MySQL tables and create data frame.




```r
#movies table
movies <- dplyr::tbl(con, sql("SELECT movie_id, movie_title, movie_year FROM movies"))

#ratings table
ratings <- dplyr::tbl(con, sql("SELECT * FROM movie_ratings"))
```

-----

### Join MySQL `movies` and `ratings` tables.

In MongdoDB, a document should contain all data related to each item. Each document represents a rating of a movie. There are `12` movie ratings present in this small database from homework 2.

I will be joining  the `movies` and `ratings` tables.


```r
#JOIN of both movies and ratings table
movies_ratings <- tbl(con, sql("SELECT a.rating_id, a.rating, b.movie_title, b.movie_year
             FROM movie_ratings a JOIN movies b ON a.movie_id = b.movie_id"))

movies_ratings_df <- as.data.frame(movies_ratings)

#data frame: 
dim(movies_ratings_df)
```

```
## [1] 12  4
```

```r
kable(movies_ratings_df, format="markdown")
```



| rating_id| rating|movie_title                                | movie_year|
|---------:|------:|:------------------------------------------|----------:|
|         1|      5|PETER RABBIT                               |       2018|
|         7|      4|PETER RABBIT                               |       2018|
|         2|      5|BLACK PANTHER                              |       2018|
|         8|      3|BLACK PANTHER                              |       2018|
|         3|      5|JUMANJI: WELCOME TO THE JUNGLE             |       2017|
|         9|      3|JUMANJI: WELCOME TO THE JUNGLE             |       2017|
|         4|      4|DEN OF THIEVES                             |       2018|
|        10|      2|DEN OF THIEVES                             |       2018|
|         5|      3|FIFTY SHADES FREED FAVORITE THEATER BUTTON |       2018|
|        11|      4|FIFTY SHADES FREED FAVORITE THEATER BUTTON |       2018|
|         6|      5|THE GREATEST SHOWMAN                       |       2018|
|        12|      4|THE GREATEST SHOWMAN                       |       2018|

-----

### Connect to MongoDB database and collection. 




```r
# create connection, database and collection
my_collection = mongolite::mongo(collection = "Movie Ratings", db = "Data607_Week13")
```

-----

### Insert movie ratings data in MongdoDB database collection. 



```r
#insert movies
my_collection$insert(movies_ratings_df)
```

```
## List of 5
##  $ nInserted  : num 12
##  $ nMatched   : num 0
##  $ nRemoved   : num 0
##  $ nUpserted  : num 0
##  $ writeErrors: list()
```

-----

### Check movies data inserted in MongoDB collections.




```r
my_collection$count()
```

```
## [1] 12
```

-----

### Display records in MongoDB collection.  


```r
kable(my_collection$find(), format="markdown")
```



| rating_id| rating|movie_title                                | movie_year|
|---------:|------:|:------------------------------------------|----------:|
|         1|      5|PETER RABBIT                               |       2018|
|         7|      4|PETER RABBIT                               |       2018|
|         2|      5|BLACK PANTHER                              |       2018|
|         8|      3|BLACK PANTHER                              |       2018|
|         3|      5|JUMANJI: WELCOME TO THE JUNGLE             |       2017|
|         9|      3|JUMANJI: WELCOME TO THE JUNGLE             |       2017|
|         4|      4|DEN OF THIEVES                             |       2018|
|        10|      2|DEN OF THIEVES                             |       2018|
|         5|      3|FIFTY SHADES FREED FAVORITE THEATER BUTTON |       2018|
|        11|      4|FIFTY SHADES FREED FAVORITE THEATER BUTTON |       2018|
|         6|      5|THE GREATEST SHOWMAN                       |       2018|
|        12|      4|THE GREATEST SHOWMAN                       |       2018|

-----


