# Reference

* See original post from digital ocean (https://www.digitalocean.com/community/tutorials/how-to-improve-database-searches-with-full-text-search-in-mysql-5-6-on-ubuntu-16-04)

<br/>

# Experimental results
```
mysql> CREATE DATABASE testdb;
Query OK, 1 row affected (0.01 sec)

mysql> USE testdb;
Database changed
mysql> describe
    -> ;
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '' at line 1
mysql> CREATE TABLE news (
    ->    id INT NOT NULL AUTO_INCREMENT,
    ->    title TEXT NOT NULL,
    ->    content TEXT NOT NULL,
    ->    author TEXT NOT NULL,
    ->
    ->    PRIMARY KEY (id)
    -> );
Query OK, 0 rows affected (0.03 sec)

mysql> INSERT INTO news (id, title, content, author) VALUES
    -> (1, 'Pacific Northwest high-speed rail line', 'Currently there are only a few options for traveling the 140 miles between Seattle and Vancouver and none of them are ideal.', 'Greg'),
    -> (2, 'Hitting the beach was voted the best part of life in the region', 'Exploring tracks and trails was second most popular, followed by visiting the shops and then traveling to local parks.', 'Ethan'),
    -> (3, 'Machine Learning from scratch', 'Bare bones implementations of some of the foundational models and algorithms.', 'Jo');
Query OK, 3 rows affected (0.01 sec)
Records: 3  Duplicates: 0  Warnings: 0

mysql> describe news
    -> ;
+---------+------+------+-----+---------+----------------+
| Field   | Type | Null | Key | Default | Extra          |
+---------+------+------+-----+---------+----------------+
| id      | int  | NO   | PRI | NULL    | auto_increment |
| title   | text | NO   |     | NULL    |                |
| content | text | NO   |     | NULL    |                |
| author  | text | NO   |     | NULL    |                |
+---------+------+------+-----+---------+----------------+
4 rows in set (0.01 sec)

mysql> select * from news;
+----+-----------------------------------------------------------------+------------------------------------------------------------------------------------------------------------------------------+--------+
| id | title                                                           | content                                                                                                                      | author |
+----+-----------------------------------------------------------------+------------------------------------------------------------------------------------------------------------------------------+--------+
|  1 | Pacific Northwest high-speed rail line                          | Currently there are only a few options for traveling the 140 miles between Seattle and Vancouver and none of them are ideal. | Greg   |
|  2 | Hitting the beach was voted the best part of life in the region | Exploring tracks and trails was second most popular, followed by visiting the shops and then traveling to local parks.       | Ethan  |
|  3 | Machine Learning from scratch                                   | Bare bones implementations of some of the foundational models and algorithms.                                                | Jo     |
+----+-----------------------------------------------------------------+------------------------------------------------------------------------------------------------------------------------------+--------+
3 rows in set (0.01 sec)

mysql> ALTER TABLE news ADD FULLTEXT (title, content, author);
Query OK, 0 rows affected, 1 warning (0.16 sec)
Records: 0  Duplicates: 0  Warnings: 1

mysql> describe news;
+---------+------+------+-----+---------+----------------+
| Field   | Type | Null | Key | Default | Extra          |
+---------+------+------+-----+---------+----------------+
| id      | int  | NO   | PRI | NULL    | auto_increment |
| title   | text | NO   | MUL | NULL    |                |
| content | text | NO   |     | NULL    |                |
| author  | text | NO   |     | NULL    |                |
+---------+------+------+-----+---------+----------------+
4 rows in set (0.00 sec)

mysql> SELECT * FROM news WHERE MATCH (title,content,author) AGAINST ('Seattle beach' IN NATURAL LANGUAGE MODE)
    -> ;
+----+-----------------------------------------------------------------+------------------------------------------------------------------------------------------------------------------------------+--------+
| id | title                                                           | content                                                                                                                      | author |
+----+-----------------------------------------------------------------+------------------------------------------------------------------------------------------------------------------------------+--------+
|  1 | Pacific Northwest high-speed rail line                          | Currently there are only a few options for traveling the 140 miles between Seattle and Vancouver and none of them are ideal. | Greg   |
|  2 | Hitting the beach was voted the best part of life in the region | Exploring tracks and trails was second most popular, followed by visiting the shops and then traveling to local parks.       | Ethan  |
+----+-----------------------------------------------------------------+------------------------------------------------------------------------------------------------------------------------------+--------+
2 rows in set (0.01 sec)

mysql> SELECT * FROM news WHERE MATCH (title,content,author) AGAINST ('Seattle beach' IN NATURAL LANGUAGE MODE)\G
*************************** 1. row ***************************
     id: 1
  title: Pacific Northwest high-speed rail line
content: Currently there are only a few options for traveling the 140 miles between Seattle and Vancouver and none of them are ideal.
 author: Greg
*************************** 2. row ***************************
     id: 2
  title: Hitting the beach was voted the best part of life in the region
content: Exploring tracks and trails was second most popular, followed by visiting the shops and then traveling to local parks.
 author: Ethan
2 rows in set (0.00 sec)

mysql> SELECT id, MATCH (title,content,author) AGAINST ('traveling to parks') as score FROM news;
+----+----------------------+
| id | score                |
+----+----------------------+
|  1 | 0.031008131802082062 |
|  2 |  0.25865283608436584 |
|  3 |                    0 |
+----+----------------------+
3 rows in set (0.01 sec)

mysql> SELECT * FROM news WHERE MATCH (title,content,author) AGAINST ('em' IN NATURAL LANGUAGE MODE)\G
Empty set (0.00 sec)

mysql> SELECT id, MATCH (title,content,author) AGAINST ('deal') as score FROM news;
+----+-------+
| id | score |
+----+-------+
|  1 |     0 |
|  2 |     0 |
|  3 |     0 |
+----+-------+
3 rows in set (0.00 sec)

mysql> SELECT * FROM news WHERE MATCH (title,content,author) AGAINST ('deal' IN NATURAL LANGUAGE MODE)\G
Empty set (0.00 sec)

mysql> SELECT * FROM news WHERE MATCH (title,content,author) AGAINST ('deal' IN BOOLEAN MODE)\G
Empty set (0.00 sec)

mysql> SELECT * FROM news WHERE MATCH (title,content,author) AGAINST ('travel' IN BOOLEAN MODE)\G
Empty set (0.00 sec)

mysql> SELECT * FROM news WHERE MATCH (title,content,author) AGAINST ('traveling' IN BOOLEAN MODE)\G
*************************** 1. row ***************************
     id: 1
  title: Pacific Northwest high-speed rail line
content: Currently there are only a few options for traveling the 140 miles between Seattle and Vancouver and none of them are ideal.
 author: Greg
*************************** 2. row ***************************
     id: 2
  title: Hitting the beach was voted the best part of life in the region
content: Exploring tracks and trails was second most popular, followed by visiting the shops and then traveling to local parks.
 author: Ethan
2 rows in set (0.00 sec)

mysql> SELECT * FROM news WHERE MATCH (title,content,author) AGAINST ('traveling' IN NATURAL LANGUAGE MODE)\G
*************************** 1. row ***************************
     id: 1
  title: Pacific Northwest high-speed rail line
content: Currently there are only a few options for traveling the 140 miles between Seattle and Vancouver and none of them are ideal.
 author: Greg
*************************** 2. row ***************************
     id: 2
  title: Hitting the beach was voted the best part of life in the region
content: Exploring tracks and trails was second most popular, followed by visiting the shops and then traveling to local parks.
 author: Ethan
2 rows in set (0.00 sec)

mysql> SELECT * FROM news WHERE MATCH (title,content,author) AGAINST ('travel' IN NATURAL LANGUAGE MODE)\G
Empty set (0.00 sec)
```