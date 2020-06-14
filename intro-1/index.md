# Create simple table

```sql
postgres=# CREATE TABLE test (id integer, text varchar(255), PRIMARY KEY(id));
CREATE TABLE
postgres=# SELECT * FROM test;
 id | text 
----+------
(0 rows)
```
# Add column
```sql
postgres=# ALTER TABLE test
postgres-# ADD COLUMN bool_value boolean;
ALTER TABLE
postgres=# SELECT * FROM test;
 id | text | bool_value 
----+------+------------
(0 rows)

postgres=# ALTER TABLE test
ADD COLUMN int_value integer;
ALTER TABLE
postgres=# SELECT * FROM test;
 id | text | bool_value | int_value 
----+------+------------+-----------
(0 rows)
```

# Add records to table

```sql
postgres=# INSERT INTO test
VALUES
(1, 'Hier steht total viel Text drin!', FALSE, 0);
INSERT 0 1
postgres=# SELECT * FROM test;
 id |               text               | bool_value | int_value 
----+----------------------------------+------------+-----------
  1 | Hier steht total viel Text drin! | f          |         0
(1 row)
```

# Filter record set using `WHERE`

```sql
postgres=# SELECT * FROM test WHERE bool_value = FALSE;
 id |               text               | bool_value | int_value 
----+----------------------------------+------------+-----------
  1 | Hier steht total viel Text drin! | f          |         0
  5 | Hier Text drin!                  | f          |  10000000
  6 | Hier Text drin!                  | f          |  10000000
  7 | Hier Text drin!                  | f          |  30000000
(4 rows)
```

# Create relation between tables (m2m [Many-to-Many])

```sql
postgres=# CREATE TABLE users (id integer, name varchar(55), PRIMARY KEY(id));
CREATE TABLE

postgres=# CREATE TABLE user_tests (user_id integer REFERENCES users (id) ON UPDATE CASCADE ON DELETE CASCADE, test_id integer REFERENCES test (id) ON UPDATE CASCADE ON DELETE CASCADE, CONSTRAINT user_tests_pk PRIMARY KEY(user_id, test_id));
CREATE TABLE
postgres=# \d
           List of relations
 Schema |    Name    | Type  |  Owner   
--------+------------+-------+----------
 public | test       | table | postgres
 public | user_tests | table | postgres
 public | users      | table | postgres
(3 rows)
```
We create a "Lookup-Table" `user_tests` to simplify relational lookups, this is also the best way to handle m2m relations. 

# Constriaints

```sql
postgres=# INSERT INTO user_tests
VALUES
(8, 2), (9, 2), (10, 2), (1, 2);
ERROR:  duplicate key value violates unique constraint "user_tests_pk"
DETAIL:  Key (user_id, test_id)=(1, 2) already exists.
postgres=# INSERT INTO user_tests
VALUES
postgres=# INSERT INTO user_tests
VALUES
(2, 8), (2, 9), (2, 10);
INSERT 0 3
postgres=# select * From user_tests;
 user_id | test_id 
---------+---------
       1 |       1
       1 |       2
       1 |       3
       1 |       4
       1 |       5
       1 |       6
       1 |       7
       2 |       8
       2 |       9
       2 |      10
(10 rows)
```

# Joining tables ("raw" `JOIN`)
```sql
postgres=# SELECT * FROM test JOIN user_tests ON test.id = user_tests.user_id;
 id |               text               | bool_value | int_value | user_id | test_id 
----+----------------------------------+------------+-----------+---------+---------
  1 | Hier steht total viel Text drin! | f          |         0 |       1 |       1
  1 | Hier steht total viel Text drin! | f          |         0 |       1 |       2
  1 | Hier steht total viel Text drin! | f          |         0 |       1 |       3
  1 | Hier steht total viel Text drin! | f          |         0 |       1 |       4
  1 | Hier steht total viel Text drin! | f          |         0 |       1 |       5
  1 | Hier steht total viel Text drin! | f          |         0 |       1 |       6
  1 | Hier steht total viel Text drin! | f          |         0 |       1 |       7
  2 | Hier steht noch mehr Text drin!  | t          |         1 |       2 |       8
  2 | Hier steht noch mehr Text drin!  | t          |         1 |       2 |       9
  2 | Hier steht noch mehr Text drin!  | t          |         1 |       2 |      10
(10 rows)
```
we used the wrong column to join tables of `user_tests` and `test` as you can see we get multiple times record 1 and 2 of the `test` table.

```sql
postgres=# SELECT * FROM test JOIN user_tests ON test.id = user_tests.test_id;
 id |               text               | bool_value | int_value | user_id | test_id 
----+----------------------------------+------------+-----------+---------+---------
  1 | Hier steht total viel Text drin! | f          |         0 |       1 |       1
  2 | Hier steht noch mehr Text drin!  | t          |         1 |       1 |       2
  3 | Hier steht noch mehr Text drin!  | t          |         1 |       1 |       3
  4 | Hier steht noch mehr Text drin!  | t          |         1 |       1 |       4
  5 | Hier Text drin!                  | f          |  10000000 |       1 |       5
  6 | Hier Text drin!                  | f          |  10000000 |       1 |       6
  7 | Hier Text drin!                  | f          |  30000000 |       1 |       7
  8 | -                                | t          |       300 |       2 |       8
  9 | qewer                            | t          |      3001 |       2 |       9
 10 | LOREM IPSUM                      | t          |     30012 |       2 |      10
(10 rows)
```
Now we get the `user_id` for each `test` record.

We can also use other operators to join tables, for example `<=`.

But that doesn't make sense for this example.
```
postgres=# SELECT * FROM test JOIN user_tests ON test.id <= user_tests.test_id;
 id |               text               | bool_value | int_value | user_id | test_id 
----+----------------------------------+------------+-----------+---------+---------
  1 | Hier steht total viel Text drin! | f          |         0 |       1 |       1
  1 | Hier steht total viel Text drin! | f          |         0 |       1 |       2
  2 | Hier steht noch mehr Text drin!  | t          |         1 |       1 |       2
  1 | Hier steht total viel Text drin! | f          |         0 |       1 |       3
  2 | Hier steht noch mehr Text drin!  | t          |         1 |       1 |       3
  3 | Hier steht noch mehr Text drin!  | t          |         1 |       1 |       3
  1 | Hier steht total viel Text drin! | f          |         0 |       1 |       4
  2 | Hier steht noch mehr Text drin!  | t          |         1 |       1 |       4
  3 | Hier steht noch mehr Text drin!  | t          |         1 |       1 |       4
  4 | Hier steht noch mehr Text drin!  | t          |         1 |       1 |       4
  1 | Hier steht total viel Text drin! | f          |         0 |       1 |       5
  2 | Hier steht noch mehr Text drin!  | t          |         1 |       1 |       5
  3 | Hier steht noch mehr Text drin!  | t          |         1 |       1 |       5
  4 | Hier steht noch mehr Text drin!  | t          |         1 |       1 |       5
  5 | Hier Text drin!                  | f          |  10000000 |       1 |       5
  1 | Hier steht total viel Text drin! | f          |         0 |       1 |       6
  2 | Hier steht noch mehr Text drin!  | t          |         1 |       1 |       6
  3 | Hier steht noch mehr Text drin!  | t          |         1 |       1 |       6
  4 | Hier steht noch mehr Text drin!  | t          |         1 |       1 |       6
  5 | Hier Text drin!                  | f          |  10000000 |       1 |       6
  6 | Hier Text drin!                  | f          |  10000000 |       1 |       6
  1 | Hier steht total viel Text drin! | f          |         0 |       1 |       7
  2 | Hier steht noch mehr Text drin!  | t          |         1 |       1 |       7
  3 | Hier steht noch mehr Text drin!  | t          |         1 |       1 |       7
  4 | Hier steht noch mehr Text drin!  | t          |         1 |       1 |       7
  5 | Hier Text drin!                  | f          |  10000000 |       1 |       7
  6 | Hier Text drin!                  | f          |  10000000 |       1 |       7
  7 | Hier Text drin!                  | f          |  30000000 |       1 |       7
  1 | Hier steht total viel Text drin! | f          |         0 |       2 |       8
  2 | Hier steht noch mehr Text drin!  | t          |         1 |       2 |       8
  3 | Hier steht noch mehr Text drin!  | t          |         1 |       2 |       8
  4 | Hier steht noch mehr Text drin!  | t          |         1 |       2 |       8
  5 | Hier Text drin!                  | f          |  10000000 |       2 |       8
```
We can also join multiple tables at once.
```sql
postgres=# SELECT * FROM test JOIN user_tests ON test.id = user_tests.test_id JOIN users ON user_tests.user_id = users.id;
 id |               text               | bool_value | int_value | user_id | test_id | id |     name     
----+----------------------------------+------------+-----------+---------+---------+----+--------------
  1 | Hier steht total viel Text drin! | f          |         0 |       1 |       1 |  1 | SOME NAME
  2 | Hier steht noch mehr Text drin!  | t          |         1 |       1 |       2 |  1 | SOME NAME
  3 | Hier steht noch mehr Text drin!  | t          |         1 |       1 |       3 |  1 | SOME NAME
  4 | Hier steht noch mehr Text drin!  | t          |         1 |       1 |       4 |  1 | SOME NAME
  5 | Hier Text drin!                  | f          |  10000000 |       1 |       5 |  1 | SOME NAME
  6 | Hier Text drin!                  | f          |  10000000 |       1 |       6 |  1 | SOME NAME
  7 | Hier Text drin!                  | f          |  30000000 |       1 |       7 |  1 | SOME NAME
  8 | -                                | t          |       300 |       2 |       8 |  2 | Peter Lustig
  9 | qewer                            | t          |      3001 |       2 |       9 |  2 | Peter Lustig
 10 | LOREM IPSUM                      | t          |     30012 |       2 |      10 |  2 | Peter Lustig
(10 rows)
```
# Limiting columns

```sql
postgres=# SELECT user_tests.*, test.bool_value, test.int_value, users.name FROM test JOIN user_tests ON test.id = user_tests.test_id JOIN users ON user_tests.user_id = users.id;
 user_id | test_id | bool_value | int_value |     name     
---------+---------+------------+-----------+--------------
       1 |       1 | f          |         0 | SOME NAME
       1 |       2 | t          |         1 | SOME NAME
       1 |       3 | t          |         1 | SOME NAME
       1 |       4 | t          |         1 | SOME NAME
       1 |       5 | f          |  10000000 | SOME NAME
       1 |       6 | f          |  10000000 | SOME NAME
       1 |       7 | f          |  30000000 | SOME NAME
       2 |       8 | t          |       300 | Peter Lustig
       2 |       9 | t          |      3001 | Peter Lustig
       2 |      10 | t          |     30012 | Peter Lustig
(10 rows)
```

# Using postgresql functions
```sql
postgres=# SELECT user_tests.test_id, CONCAT(user_tests.user_id, users.name) as USER, test.bool_value, test.int_value FROM test JOIN user_tests ON test.id = user_tests.test_id JOIN users ON user_tests.user_id = users.id;
 test_id |     user      | bool_value | int_value 
---------+---------------+------------+-----------
       1 | 1SOME NAME    | f          |         0
       2 | 1SOME NAME    | t          |         1
       3 | 1SOME NAME    | t          |         1
       4 | 1SOME NAME    | t          |         1
       5 | 1SOME NAME    | f          |  10000000
       6 | 1SOME NAME    | f          |  10000000
       7 | 1SOME NAME    | f          |  30000000
       8 | 2Peter Lustig | t          |       300
       9 | 2Peter Lustig | t          |      3001
      10 | 2Peter Lustig | t          |     30012
(10 rows)

postgres=# SELECT user_tests.test_id, CONCAT(user_tests.user_id, ' ', users.name) as USER, test.bool_value, test.int_value FROM test JOIN user_tests ON test.id = user_tests.test_id JOIN users ON user_tests.user_id = users.id;
 test_id |      user      | bool_value | int_value 
---------+----------------+------------+-----------
       1 | 1 SOME NAME    | f          |         0
       2 | 1 SOME NAME    | t          |         1
       3 | 1 SOME NAME    | t          |         1
       4 | 1 SOME NAME    | t          |         1
       5 | 1 SOME NAME    | f          |  10000000
       6 | 1 SOME NAME    | f          |  10000000
       7 | 1 SOME NAME    | f          |  30000000
       8 | 2 Peter Lustig | t          |       300
       9 | 2 Peter Lustig | t          |      3001
      10 | 2 Peter Lustig | t          |     30012
(10 rows)
```
