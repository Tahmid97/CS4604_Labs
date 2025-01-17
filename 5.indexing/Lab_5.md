## Objective
Understanding indexing

## Lab

### Setup

* Important: This lab will create a `lab5.db` with size around 600MB, make sure you have enough disk space.

* Prepare lab5 database
  ```
  > cp ../db/labs.db lab5.db
  ```

* Connect to a database
   ```
   > sqlite3 lab5.db
   ```

* Create a table: `big_cards`
  ```
  create table big_cards as select * from cards;
  insert into big_cards select * from cards;
  insert into big_cards select * from big_cards;
  insert into big_cards select * from big_cards;
  insert into big_cards select * from big_cards;
  insert into big_cards select * from big_cards;
  insert into big_cards select * from big_cards;
  insert into big_cards select * from big_cards;
  insert into big_cards select * from big_cards;
  insert into big_cards select * from big_cards;
  insert into big_cards select * from big_cards;
  ```

A data set for the collection of cards for [Hearthstone](https://playhearthstone.com/en-us/),
the popular online card game by Blizzard. This data set is freely available from 
[Kaggle](https://www.kaggle.com/jeradrose/hearthstone-cards).

### Indexing

Indexes are used to retrieve data from the database very fast. The users cannot see the indexes, they are just used to speed up searches/queries.

#### EXPLAIN QUERY PLAN 

In order to see the impact of our indexing efforts, we will use the `EXPLAIN QUERY PLAN` feature of SQLite.

First: turn SQL time on
```sql
sqlite> .timer ON
```

Example:

```sql
sqlite> EXPLAIN QUERY PLAN SELECT count(*) FROM big_cards;
```

Record output below:

```
QUERY PLAN
`--SCAN TABLE big_cards
Run Time: real 0.002 user 0.000000 sys 0.000000
```

#### Using Indexes to improve performance

As a developer you know that your application will perform the query below *a lot*. It's not quite as fast as you would like it to be so you start performance tuning it.

**Query**: `select card_id, name from big_cards where race = 'TOTEM';`

You suspect an index will help, but before you make any changes you want to get a baseline explain plan and execution time. You do this using `EXPLAIN QUERY PLAN`.

**Execute**: `EXPLAIN QUERY PLAN SELECT card_id, name FROM big_cards WHERE race = 'TOTEM';`

Record output below:

```
QUERY PLAN
`--SCAN TABLE big_cards
Run Time: real 0.002 user 0.000000 sys 0.000000
```

You suspect that an index on the race column will help. Let's create it.

**Execute**: `CREATE INDEX IDX1_big_cards ON big_cards(race);`

**Execute**: `EXPLAIN QUERY PLAN SELECT card_id, name FROM big_cards WHERE race = 'TOTEM';`

Record output below:

```
sqlite> CREATE INDEX IDX1_big_cards ON big_cards(race);
Run Time: real 3.786 user 2.656250 sys 1.046875

sqlite> EXPLAIN QUERY PLAN SELECT card_id, name FROM big_cards WHERE race = 'TOTEM';
QUERY PLAN
`--SEARCH TABLE big_cards USING INDEX IDX1_big_cards (race=?)
Run Time: real 0.001 user 0.000000 sys 0.000000
```

Would it be possible to satisfy the query with an index only and further speed up the query?

**Execute**: `CREATE INDEX IDX2_big_cards ON big_cards(race, card_id, name);`

**Execute**: `EXPLAIN QUERY PLAN SELECT card_id, name FROM big_cards WHERE race = 'TOTEM';`

Record output below:

```
sqlite> CREATE INDEX IDX2_big_cards ON big_cards(race, card_id, name);
Run Time: real 7.336 user 5.203125 sys 1.687500

sqlite> EXPLAIN QUERY PLAN SELECT card_id, name FROM big_cards WHERE race = 'TOTEM';
QUERY PLAN
`--SEARCH TABLE big_cards USING COVERING INDEX IDX2_big_cards (race=?)
Run Time: real 0.001 user 0.000000 sys 0.000000
```

If you issue command `VACUUM big_cards;` and re-analyze you will likely see an explain plan that *is* satisfied by the index (and consequently much faster). However, subsequent updates to the table would cause this query to go back to the table to check the visibility map.

**Execute**: `VACUUM;`

**Execute**: `EXPLAIN QUERY PLAN SELECT card_id, name FROM big_cards WHERE race = 'TOTEM';`

Record output below:

```
sqlite> VACUUM;
Run Time: real 21.266 user 5.640625 sys 12.500000

sqlite> EXPLAIN QUERY PLAN SELECT card_id, name FROM big_cards WHERE race = 'TOTEM';
QUERY PLAN
`--SEARCH TABLE big_cards USING COVERING INDEX IDX2_big_cards (race=?)
Run Time: real 0.001 user 0.000000 sys 0.000000
```

#### The performance cost of Indexes 

In general, we don't want to create unused indexes because they incur a performance penalty. The penalty is often minimal unless the application has a very high rate of updates. But it is something to be aware of.

Let's analyze an update of every row while our two indexes exist:

**Execute**: `EXPLAIN QUERY PLAN update big_cards set race = 'FOO';`

Note the Execution time.

Record output below:

```
QUERY PLAN
`--SCAN TABLE big_cards
Run Time: real 0.001 user 0.000000 sys 0.000000
```


Now let's drop the indexes and try again:

**Execute**: `drop index idx1_big_cards;`

**Execute**: `drop index idx2_big_cards;`

**Execute**: `EXPLAIN QUERY PLAN update big_cards set race = 'BAR';`

Record output below:

```
sqlite> drop index idx1_big_cards;
Run Time: real 0.118 user 0.000000 sys 0.093750

sqlite> drop index idx2_big_cards;
Run Time: real 0.313 user 0.093750 sys 0.187500

sqlite> EXPLAIN QUERY PLAN update big_cards set race = 'BAR';
QUERY PLAN
`--SCAN TABLE big_cards
Run Time: real 0.001 user 0.000000 sys 0.000000
```

Does the update took less time without the indexes? 
Your answer:
```
No, the values that I got didn't show any significant changes in update time with or without the index.
```

Describe your findings of this Lab 5 from the recorded outputs, is everything working fine? or is anything not working? etc. Please indicate your SQLite version:

```
SQLite version: 3.34.1
Findings:
Indexing speeds up lookup time, but it slows down update time.
The values that I got showed the improvement of performance after indexing.  However, the times remained unchanged (0.001) after executing VACUUM, updating with indexing and updating without indexing. Therefore, it is hard to say whether this is working correctly with the values that I found. 
```

ps. Use this command to check your SQLite version. `sqlite3 --version`

The moral of this lab is: create the indexes you need, but *only* the indexes you need.

ps. Please remember to delete lab5.db and release disk space especially if you are using either ap1 or rlogin.

### Your turn
Now it's your turn to apply what you've learned.

1. Complete this `Lab_5.md` with your result 
2. Covert your `Lab_5.md` to a PDF file. You can use something like this: https://www.markdowntopdf.com/
3. Rename your filename to `yourPID_Lab_5.pdf` and submit it to Canvas `Lab 5 assignment`
