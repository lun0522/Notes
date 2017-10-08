##Releational Database

- **Database** is a collection of one or more relations
- **Relational database** is a collection of relations with distinct relation names
- **Relation** consists of a relation schema and a relation instance
- **Relation schema** specifies name and type of each attribute (column)
- **Relation instance** is a table, a set of tuples
- **Tuple** is a row, and all rows have the same number of fields

##SQL (Structured Query Language)

###CREATE, ALTER, INSERT, DELETE, DROP

```sql
CREATE TABLE Students (
sid CHAR(20),
name CHAR(20),
login CHAR(10), 
age INT, 
gpa REAL,
UNIQUE (name, age),
CONSTRAINT StudentsKey PRIMARY KEY (sid) );
// "UNIQUE" declares that a subset of the columns constitute a key
// if the constraint set by "CONSTRAINT" is violated, the constraint
// name (StudentsKey) will be returned and can be used to identify the error

CREATE TABLE Enrolled (
sid CHAR(20), 
cid CHAR(20),grade CHAR(2),
PRIMARY KEY (sid,cid),
FOREIGN KEY (sid) REFERENCES Students (sid)
	ON DELETE CASCADE
	ON UPDATE CASCADE );
	// NO ACTION (default setting)
	// CASCADE: delete all tuples that refer to deleted tuple
	// SET NULL / SET DEFAULT: set foreign key value of referencing tuple

// altered by adding a new field; every tuple gets a "null" in the new field
ALTER TABLE Students ADD COLUMN nationality CHAR(30);

// altered by setting "sid" as the primary key
ALTER TABLE STUDENTS ADD PRIMARY KEY (sid);

// insert a tuple; column names can be omitted (not recommend to do so)
INSERT INTO Students (sid, name, login, age, gpa)
VALUES (12345, 'Smith', 'smith@ee', 18, 3.2, 'US');

// delete tuples that satisfy the conditions
DELETE FROM Students WHERE name = 'Smith';

// delete schema info and tuples of Students
DROP TABLE Students;
```

- **Super key** is a set of *one or more* attributes (columns) to uniquely identify rows in a table
- **Candidate key** is the shortest (minimal) super keys (*i.e.* with no redundant attribute); they are candidates of the primary key
- **Primary key** is selected from candidate keys; a table has *only one* primary key; primary keys *cannot* contain "null" 
- **Primary key ⊆ candidate key ⊆ super key**
- **Foreign key** is a field (or collection of fields) in one table that refers to the *primary key* in another table

Table `Enrolled` now refers to `Students` by the foreign key. If the primary key `sid` of a student is deleted/modified in `Students`, we have three options:

1. **`NO ACTION`**: disallow the deletion/modification if this student has enrolled in any class
2. **`CASCADE`**: delete/modify all rows in `Enrolled` that refer to this student
3. **`SET NULL / SET DEFAULT`**: for all rows in `Enrolled` that refer to this student, set `sid` to that of a default student, or `null`

###QUERY

```sql
// basic query
// "DISTINCT" is used eliminate duplicates; this is not done by default
// return names of sailors that have boat NO.101:
SELECT DISTINCT S.sname
FROM Sailors S, Reserves R
WHERE S.sid=R.sid AND R.bid=101;

// "AS" is used to name fields in result
// "LIKE" is used for string matching
// (’_’ -> any one character; ’%’ -> 0 or more arbitrary characters)
SELECT S.age, S.age > 30 AS isOver30, 2*S.age AS age2
FROM Sailors S
WHERE S.sname LIKE 'B_%B';

// "SELECT *" means to return all attributes (columns)
// three ways to join two tables:
SELECT * FROM Sailors S JOIN Reserves R ON (S.sid = R.sid);
SELECT * FROM Sailors JOIN Reserves USING (sid);
SELECT * FROM Sailors NATURAL JOIN Reserves;

// operation on two quey results
SELECT ... FROM ... WHERE...
UNION / INTERSECT / EXCEPT
SELECT ... FROM ... WHERE...

// test the existence of any query result
// seems to return a boolean
// eg: select all sailors that have red boats
SELECT *
FROM   Sailors 
WHERE  (NOT) EXISTS (SELECT * 
					 FROM Boats 
					 WHERE Sailors.sid=Boats.sid AND Boats.color='red')

// we can also use IN
SELECT *
FROM   Sailors
WHERE  Sailors.sid (NOT) IN (SELECT Boats.sid 
					   		 FROM Boats 
					   		 WHERE Boats.color='red')

// statistical value: COUNT / SUM / AVG / MAX / MIN (aka. Aggregate Operators)
// seems to return a number (single column)
// eg: select the oldest sailor
SELECT S.sname, S.age 
FROM   Sailors S 
WHERE  S.age = (SELECT MAX(S2.age) 
				FROM Sailors S2);

// do comparisons with a set of query results: ALL / ANY
// seems to return a table
// eg: select sailors whose rating is greater than that of all sailors called Bob
SELECT *FROM   SailorsSWHERE  S.rating > ALL (SELECT S2.rating					   FROM SailorsS2					   WHERE S2.sname='Bob');

// GROUP BY... HAVING ..., which is executed after (SELECT ... FROM ... WHERE...) is done
// outputs one answer for each group
// attributes after SELECT must either be aggregate operations, or appear also after GROUP BY
// i.e. it must be one-column or will be grouped to be one-column
// attributes after HAVING must have a single value per qualifying group
// EVERY applies to each row and helps to filter out an entire group
SELECT   S.rating, MIN(S.salary) 
FROM     Sailors S
WHERE    S.rating>0GROUP BY S.rating;
HAVING   COUNT(*) > 1 AND EVERY (S.age <=60)

// nothing equals to (=) NULL
// should use IS (NOT) NULL
SELECT * FROM Sailors2 S WHERE S.ssn IS NULL;
```

###INNER / LEFT OUTER / RIGHT OUTER JOIN

![INNER JOIN](https://www.w3schools.com/sql/img_innerjoin.gif) ![LEFT OUTER JOIN](https://www.w3schools.com/sql/img_leftjoin.gif) ![RIGHT OUTER JOIN](https://www.w3schools.com/sql/img_rightjoin.gif)

##Relational Algebra

- Selection operator (σ): output a relation containing a subset of the **tuples** from the input relation
- Projection operator (π): output a relation containing a subset of the **columns** from the input relation
- Cross Product (×): input two relations and output a relation containing all **pairs of tuples** from both relations
- Join (⋈)
- Division (/): R/B is the largest relation T such that T×B⊆R (eg: to find the names of sailors who’ve reserved all boats = Reserves / Boats = π\[sid](Reserves) - π\[sid]((π\[sid](Reserves) × Boats - Reserves)), which means to list all possible combination of sid and bid, and subtract the Reserves, then if any sailor has reserved all boats, his sid will not appear in the result)

## Tree Indexing

![](http://images.slideplayer.com/16/4964545/slides/slide_12.jpg)

- Non-leaf pages (contain index entries)
- Leaf pages (contain data entries)

###ISAM (static)

![](http://images.slideplayer.com/32/9946948/slides/slide_7.jpg)

Actually it is a 2-node tree. Index nodes never change after construction, so there will be overflow pages:

![](http://images.slideplayer.com/26/8445128/slides/slide_7.jpg)

###B+ Tree (dynamic)

![](https://i.stack.imgur.com/pJXJI.jpg)

Suppose that each node has **F children nodes** (F = fanout), i.e. F pointers pointing downward; amount of data in total is **n**, and one page can contain **L data entries**, then there will be **N = n/L leaf pages**.

When we do a binary search, every time we divide the rest records into 2 parts, so the time complexity is **log2(n)**. When we use a B+ tree, each time we divide the rest part into F parts, so the time complexity becomes **logF(N)**, or logF(n/L).

Note that there are still L data per entry, and after we retrive an entry using B+ tree, we still have to find out the data we want from these L records. However, since L is a constant, this operation costs constant time, so we cannot see it in the time complexity.

Usually each node, except for the root node, should have at least 50% occupancy. If <50%, consider merging; If >100%, consider splitting. Each node contains d <= F <= 2d entries, where **d is the order** of the tree.

If use B+ tree but unclustered index, when retrieve more than 5% of tuples, just scan all the data, don't use the tree.

##Hash-Based Indexing

These metohds are only for **equality selections**, while tree indexing also supports range selections.

###Static Hashing

![](http://images.slideplayer.com/26/8697966/slides/slide_3.jpg)

Primary bucket pages are not modified since created, and overflow pages keep growing. We can do rehashing when there are too many overflow pages after the bucket page.

###Extendible Hashing

![](http://os9hogvk5.bkt.clouddn.com/hash_ppt.jpg)

In the example we use the last two digits as hash value.

When a bucket page cannot hold that many data entries, we have to add one more bucket page (local depth +1), and double the directory if necessary, to use one more digit to label bucket pages (global depth +1).

###Linear Hashing

![](http://images.slideplayer.com/26/8697966/slides/slide_14.jpg)

![](http://images.slideplayer.com/26/8697966/slides/slide_16.jpg)

This method does not use directory, but it allows overflow pages. It uses a "Next" pointer, pointing to the next bucket page to split. Suppose that before all splitting, there are N bucket pages.

When we search for a record with a certain hash value R (calculated by hash function h0), if R is equal to or larger than "Next", we can simply retrieve the record in position R; but if it is smaller than "Next", then the record might either locate in R **or** in R+N, since that bucket page has already been splitted, and thus we need to call the hash function that takes **one more digit** into consideration (h1).

As shown in the example, the "Next" pointer hasn't been pointing to the bucket page with h0 = 11 yet, so 43* is put into the overflow page. 

When to split bucket pages is determined by the people who implement it; there is no unified criteria. Just need to make sure that there are not so many overflow pages.

##Sorting

###2-Way External Merge Sort

run: a sorted portion of the file 

- Pass 0: read a page at a time, sort it any algorithm, write it to the disk (need only one buffer page in the main memory) (creates runs of length 1 page)

- Pass 1: merge two runs of length 1 page to a run of length 2

- Pass 2: merge two runs of length 2 page

And so forth. Suppose that we have **N** pages in total, pass 0 costs 2N I/O operations (read and then write, ignore the cost of sorting), while other passes cost 2Nlog2(N). In total, it costs 2N(log2(N)+1) I/O.

![](http://images.slideplayer.com/14/4497575/slides/slide_5.jpg)

It can use only three pages in the main memory: two for input and one for output. If **B** buffer pages can be used in the main memory, in pass 0, we can sort *B* pages and merge them into 1; for other passes, B-1 buffers pages can be used for input, so the number of runs will be multiplied by *B-1* after each run. 

Suppose that except for pass 0, we need **P** passes to sort all data, then B*(B-1)^P=N, hence P=log(B-1)(N/B). Each pass still needs 2N I/O, so the total I/O cost is 2N(log(B-1)(N/B)+1).

eg: if we have 250 pages to sort, and 5 buffer pages in the main memory:

- pass 0: use 5 buffer pages to create 50 sorted runs
- pass 1: use 4 buffer pages to read from runs, and 1 as output page, round_up(50/4) = 13
- pass 2: round_up(13/4) = 4
- pass 3: round_up(4/4) = 1

Total I/O: 4 passes * 250 pages = 1000

##Implementation

###Selection

Depends on what do we have.

- **Sequential scan**
- Has **B+ tree** (binary search, or turn to sequential scan if we have to retrieve >5% unclustered tuples)
- Has **hashing-based index** (only equality selections)

###Projection

Eliminating duplicates is expensive.

- **External merge sort**, delete unwanted columns in pass 0, and eliminate duplicates in later passes
- **Hashing-based** projection, all duplicates will go into the same bucket. If the whole hash table doesn't fit into the main memory, just process one butket at a time. If one bucket is still too large, apply another hashing function to split it. This way leads to **parallelizing**
- **Covering index** if we have. It should include all the projection attributes (can have some extra attributes, doesn't matter); best if projection attributes are prefix of search key. We don't have to retrive data, but do index-only scan and eliminate duplicates within single pass
- **RID joins**, if we have index for (SearchKey1, RID) and index for (SearchKey2, RID), but we want distinct (SearchKey1, SearchKey2), we can do projections using these two indices respectively, join them, get (SearchKey1, SearchKey2, RID), and delete RID at last

###Join

**Sailors**: 80 tuples / page, 500 pages

**Reserves**: 100 tuples / page, 1,000 pages

Both of them have no indices.

####TNLJ (Tuple Nested Loop Join):

```d
foreach tuple r in R do 
    foreach tuple s in S do
        if r.sid == s.sid then add <r, s> to result
```

**Total I/O = 1,000 + 100 * 1,000 * 500 = 50,001,000**

(Always have to read 1000 pages of R; then for each tuple in R (100 * 1000 tuples), read all pages of S (500 pages))

####PNLJ (Page Nested Loop Join):

Keep one page, rather than one tuple, from each table in main memory.

```d
foreach page p1 in R do 
    foreach page p2 in S do
        foreach r in p1 do 
            foreach s in p2 do
                if r.sid == s.sid then add <r, s> to result
```

**Totol I/O = 1,000 + 1,000 * 500 = 501,000**

(Always have to read 1000 pages of R; then for each page in R (1000 pages), read all pages of S (500 pages))

Smaller relation (Sailor) should be put in the outer loop instead:

**Totol I/O = 500 + 500 * 1,000 = 500,500**

(Always have to read 500 pages of S; then for each page in S (500 pages), read all pages of R (1000 pages))

####BNLJ (Block Nested Loop Join):

Main memory can hold more pages. Keep some pages of the outer relation when we scan the inner one.

Suppose that we have a 100-page block:

**Totol I/O = 500 + (500 / 100) * 1,000 = 5,500**

(Always have to read 500 pages of S; then for each block of S (500 / 100 blocks), read all pages of R (1000 pages))

Note that use 50 buffer pages for S and 50 for R will **not** increase the speed, because in the inner loop we always have to read all pages of R, so the total I/O  will only become 500 + (500 / 50) * 1,000 = 10,500.

But if the benefit of **sequential reads** is taken into consideration, which means reading 50 pages of R at a time is faster than reading one page for 50 times, we can consider to evenly distribute buffers between S and R.

####INLJ (Index Nested Loop Join):

Suppose that we have an hashing-based index on S, on the join attribute. (1.2 I/O for hashing-based, 2~4 I/O for B+ tree)

```d
foreach tuple r in R do
    foreach tuple s in S where ri == sj do
        add <r, s> to result
```

**Totol I/O = 1,000 + 100 * 1,000 * (1.2  + 1) = 221,000**

(Always have to read 1000 pages of R; then for each tuple in R (100 * 1000 tuples), use 1.2 I/O to find the index of corresponding tuple in S, and 1 I/O to retrieve the data)

If we have hashing-based index on R, and use R as inner relation, that is, we are to find several reserves made by each sailor (on average one sailor has made (100 * 1000) / (80 * 500) = 2.5 reserves), then if the index is **clustered** (1 I/O to retrieve 2.5 tuples in R):

**Totol I/O = 500 + 80 * 500 * (1.2  + 1) = 88,500**

If the index is **unclustered** (2.5 I/O to retrieve 2.5 tuples in R):

**Totol I/O = 500 + 80 * 500 * (1.2  + 2.5) = 148,500**

####SMJ (Sort-merge Join):

Fully sort R and S (eg: by sid). Then scan R and S simultaneously, and output where R.sid == S.sid.

**Cost = (cost to sort R) + (cost to sort S) + (scan cost)**

A refinement version is, suppose that we have L and S pages in two relations (L > S), and B buffer pages in the main memory, in pass 0, it outputs L/B and S/B runs for each relation (each run is of length B); in pass 1, we read one page from each run, which requires that B-1 >= L/B + S/B, into the main memory, and do the join.

- B-1 >= L/B + S/B
- B^2 - B >= L + S
- B > 2 * sqrt(L)

Pass 0 requires 2(L+S) I/O (read and write), while pass 1 requires only (L+S) I/O (read). So this method requires 3(L+S) I/O in total **(linear cost)**. However, the operations done in pass 1 seem to be really heavy.

**Totol I/O = (500 + 1,000) * 3 = 4,500**

The result is **sorted**.

####HJ (Hash Join):

![](http://images.slideplayer.com/31/9720375/slides/slide_13.jpg)

Suppose that we have relations L and S (L > S)

Firstly, partition both relations using a hash function. Note that if the main memory has B buffer pages, the hash function can partition tuples into B-1 partitions at most, because there should be one output buffer page waiting for each partition.

In the second step, for each hash value, we read up to B-2 pages from S, and one page from L, and do the join; and then read another page from L, etc. It is possible that more than B-2 pages in S have the same hash value, then we will have to use another function to split them. If we want to avoid more splitting, we should have:

- S / (B-1) <= B-2
- B > sqrt(S)

(In step 1, S can split itself into at most B-1 partitions, to reduce the tuples in each partition; in step 2, there can be at most B-2 tuples in each partition)

Unlike sort-merge join, the minimum number of buffer pages in the main memory is determined by the **smaller** relation in this method. So if **relation sizes differ greatly** (i.e. the smaller relation is extremely small), hash join is preferred. Also, it is highly **parallelizable**.

####Blocking / Non-Blocking

- Blocking: needs to read at least one input completely before producing output, eg: SMJ, HJ
- Non-Blocking: can start when part of input is ready, eg: NLJ

####Ripple joins

When we just want to take a random sample from DB, we may want to randomly read a tuple from each relation, join them and calculate some parameters (eg: confidence), and stop when the requirement is satisfied. Blocking algorithms cannot be used certainly.

A better way is:

1. Retrieve 1 tuple from each of 2 relations, join them, and get 1 tuple
2. Retrieve 1 more tuple from each of 2 relations, join **all tuples retrieved so far**, and get 4 tuples
3. Retrieve 1 more, join and get 9 tuples, and so on

###Intersection

Equality on all fields is join condition.

###Cross product

No equality condition

###Union(+) / Difference(-)

- Sorting based approach
- Hash based approach

###Aggregate Operations

Without grouping: scan the whole relation.

With grouping:

- Sort on group-by attributes
- Build a hash table with entries < grouping-value, running-info (eg: min, max, sum) >
- Index-only scan