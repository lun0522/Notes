##Relational Database

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

If use B+ tree but unclustered index, when retrieve more than 5% of tuples, just scan all the data, don't use the tree. (For unclustered index, we can sort data entries according to the page they are going to read, to avoid that we read the same page for multiple times)

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

A **refinement** version is, suppose that we have L and S pages in two relations (L > S), and B buffer pages in the main memory, in pass 0, it outputs L/B and S/B runs for each relation (each run is of length B); in pass 1, we read one page from each run, which requires that B-1 >= L/B + S/B, into the main memory, and do the join.

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

#Design of Database

1. Conceptual design (consider what realation to put into the database, create data model) - **ER Model**
2. Schema refinement/normalization (remove bad properties like redundancy) - **Function Dependencies**

##ER Model

- **Entity set**: collection of similar objects (similar to a class), represented by a **rectangle**
- **Attribute**: property of an entity set, represented by an **oval**
- **Relationship set**: collection of connections between two or more entities, represented by a **diamond**, consisting of tuples with one component for each related entity set. Can be multiways (multiple columns)
- **Superkey**: in a relationship set, the key of **all** the entities involved

One drinker must have one favorite beer, but one beer can be favorited by several drinkers **(arrow means "one")**:
![](http://os9hogvk5.bkt.clouddn.com/16__ER.jpg)

Assuming a beer cannot be made by more than one manufacturer, and **all (thick line means total participation)** manufacturers must have exactly one best-seller respectively:
![](http://os9hogvk5.bkt.clouddn.com/16___ER.jpg)

One entity can play as different roles in a relationship:
![](http://os9hogvk5.bkt.clouddn.com/16_ER_1.jpg)

- **Subclass**: no multiple inheritance, represented by **isa triangle** (pointing to the superclass). no need to store instances of the subclass into another table; can create a table that relates instances of the subclass to attributes that only the subclass has, or add a column of the attribute that only the subclass has, and assign NULL to those not of the subclass
![](http://os9hogvk5.bkt.clouddn.com/16_ER_2.jpg)

- **Weak entity**: an owner entity can own many weak entities, while each weak entity has a partial key to identify itself *only* in its owner entity, so a weak entity is unique only when combined with its owner entity **(note the bold lines; underscore the primary key; use dotted underscore lines for the weak side)**
![](http://os9hogvk5.bkt.clouddn.com/16_ER_3.jpg)

- **Aggregation**: treat a relationship set as an entity set, to participate in another relatioship
![](http://os9hogvk5.bkt.clouddn.com/16_ER_4.jpg)

##Function Dependencies

If the rating(R) of a sailor **determines** his wage(W), we don't have to store both information in each tuple, but only store the rating, and use another table to map rating to wage, and say R→W. Note:

- different wage can correspond to the same rating, so it is **not** necessary that W→R
- if rating is a key of Sailor (not necessary to be primary), it should **not** be regarded as redunduncy, because the "another table" will have the same row number as Sailor, and thus will not help us save any space

This is a **decomposition** accoding to **function dependency** to remove **redundancy**.

###Finding FDs

Armstrong's Axioms:

- **Reflexivity**: if Y⊆X, then X→Y
- **Augmentation**: if X→Y, then XZ→YZ for any Z
- **Transitivity**: if X→Y and Y→Z, then X→Z

Suppose that F is a set of FDs, if we can start from it and apply Armstrong's Axioms to obtain a new FD, then we say the new FD is **derived** by F, and we call the collection of all FDs that can be derived from F **closure** F+.

###Attribute Closures

If we just want to find out whether a certain FD X→Y can be derived from F, we don't have to calculate the whole F+, which can be huge, but only calculate the **attribute closure** of X, and see whether Y is in the result closure:

1. let closure = X
2. for each pair of dependency U→V in F, if U⊆F, closure += V
3. after that, the closure may contain more attributes, so we loop again to see whether more attributes can be added into the closure
4. stop the outer loop if no attribute is added to the closure during a certain inner loop

For example, if we have F = {A→D, AB→E, BI→E, CD→I, E→C}, and we want to know whether F implies AE→D, we should find out whether D is in (AE)+:

1. let closure = AE
2. loop 1: add all attributes that can be inferred from A, E or AE. because A→D and E→C, add D and C to the closure. now closure = ACDE
3. loop 2: because CD→I, add I to the closure. now closure = ACDEI
4. loop 3: no attributes can be added

As the result, we can infer ACDEI from AE, which includes D, so we have AE→D.

Suppose that *a* is the number of attributes, and *f* is number of FDs in F, we need to compare at most *a* attributes each time, in each loop we need to compare for at most *f* times, and in the worst case we need *f* loops, the time complexity is O(*a(f^2)*).

Other uses of attribute closures:

- To check whether X is a **key** of U, we can check whether the attribute closure of X contains all attributes of the relation (X→U)
- If (F+) = (G+), then F is a **cover** for G and vice versa

###Lossless Join Decomposition

To remove the redundancy, we can decompose a relation into two or more relations. The decomposition of R into X and Y is lossless-join (no information lose) if and only if:

**X∩Y→X**, or **X∩Y→Y**, *i.e.* the intersection column if X and Y must be a **key** of X or Y.

###Dependency Preserving Decomposition

For lossless join decomposition, to verify some FDs, we have to join the decomposed tables. For example, if we have CSJDPQV and {JP→C, SD→P}, and decompose CSJDPQV into CSJDQV and SDP according to SD→P, whne a new data is inserted, we will not be able to verify JP→C without a join.

Dependency preserving decomposition avoids it. It is not free of redundancy, but because join is expensive, it enhances the performance.

If F is FDs of R, and X is a subset of attributes of R, then Fx means FDs where in each U→V, U and V are in X. A decomposition of R into X and Y is **dependency preserving** if:

**((Fx ∪ Fy)+) = (F+)**

In that case, when a record is inserted, we just have ti check Fx in X, and Fy in Y respectively, without a join.

###Boyce-Codd Normal Form (BCNF)

If a relation R is of BCNF, for each U→V in FDs (U is a set of atrributes while V is a single atrribute), it must be either:

- **U∋V** (called a trivial FD)
- **U contains a key** for R

If the condition is violated, decompose R into **R-V** and **UV**, and repeat if necessary. 

Note that because V is a single column and is not in U because of the violation, R-V should contain U, so (R-V)∩(UV) must contain U; we also have U→V, so we can say (R-V)∩(UV)→V, which makes it a lossless-join.

For example, if we have CSJDPQV, where C is a key, and FDs {JP→C, SD→P, J→S}. 

1. for SD→P, decompose CSJDPQV into CSJDQV and SDP
2. for J→S, decompose CSJDQV into CJDQV and JS
3. for JP→C, J and P are not in the same table anymore, so do nothing

(the order of decomposition affects the result)

###Third Normal Form (3NF)

Similar to the relationship between lossless join decomposition and dependency preserving decomposition, 3NF is less strict than BCNF.

If a relation R is of BCNF, for each U→V in FDs (U is a set of atrributes while V is a single atrribute), it must be either:

- **U∋V** (called a trivial FD)
- **U contains a key** for R
- **U is part of some keys** for R

To obtain the 3NF of relation R, whose FDs are F, firstly, we have to find the minimal cover G for F where:

- (G+) = (F+)
- right side of each FD in G is a single attribute
- in G, if any FD is deleted, or any attribute in a FD is deleted, the closure changes (we can compute the attribute closure to see whether a FD is redundant)

For example, if we have F {A→B, ABCD→E, EF→G, EF→H, ACDF→EG}, then:

1. for ABCD→E, we can only preserve ACD→E because A→B
2. for ACDF→EG, if we take it away and compute (ACDF)+ in F, we will find that EG is still included in the attribute closure, so ACDF→EG is redundant

So G is {A→B, ACD→E, EF→G, EF→H}.

Then we compute the lossless join decomposition of G. Finally, if some U→V in G is not preserved, add UV.

##Transactions and Concurrency Control

- Transaction: sequence of steps
- Atomicity: a transaction should execute completely (including **commit**) or not at all (be **rolled back**)
- Consistency: preservation of data consistency
- Isolation: no harmful interference between transactions is permitted
- Durability: once a transaction commits, the changes should be persistent
- Schedule: a sequence of operations performed by a set of transactions on the DB

```sql
START TRANSACTION;

UPDATE ...
SET ...
WHERE ...

UPDATE ...
SET ...
WHERE ...

COMMIT;
```

If no transaction block is specified, each operation runs in its own transaction.

**R-read W-write C-commit A-abort**

Three isolation anomalies:

- Dirty read: reading a value modified by an uncommitted transaction, especially when that transacrion then aborts. eg: **RW1(A)** *RW2(A)* *RW2(B)* **RW1(B)**
- Unrepeatable read: Read the same data item twice, get different. eg: **R1(A)** *W2(A)* *C2* **R1(A)**
answer
- Lost Update: transactions' writing get mixed up giving an inconsistent DB at the end. eg: **W3(A)** *W4(A)* *W4(B)* **W3(B)**

###Serializability

####Final-State Serializability

If there is some **serial** schedule S' involving the same transactions as S, such that the final DB is the same after executing S and S.

eg: **R1(A)** *W2(A)* **R1(A)** **C1** *C2*

It cares more about the writing, so that the final data stored on the disk will be the same. But the example above can still contain unrepeatable read.

####View Serializability

S is view serializable if it is view-equivalent to some serial schedule:

![](http://os9hogvk5.bkt.clouddn.com/20_1.png)

This is nice but difficult to implement in practice.

####Conflict Serializability

Two operations by **different** transactions on the same object **conflict** if **at least one is a write** when interleaving (WW->lost update; RW->unrepeatable read; WR->dirty read). Conflicting operations do not commute (will have different result if change in order)

In a conflict graph, each node is a transaction, and the edge from i to j represents an operation of i that **conflicts** with and **precedes** an operation of j.

![](http://os9hogvk5.bkt.clouddn.com/21_2.jpg)

A schedule is conflict serializable if its conflict graph contains **no cycle** (cycles can only be caused by interleaving, so conflict graphs of serial schedules can have edges, but no cycles).

![](http://os9hogvk5.bkt.clouddn.com/21_1.jpg)

###Abort (Rolling Back)

####Recoverable

A transaction **commits** only after the commit of all transactions whose changes it has read. 

However, it is hard to implement.

####Avoids Cascading Aborts (ACA)

A transaction never **reads** uncommitted writes. 

Limit on the time of read, so it's more strict. But it only cares about read, and cannot deal with the situation where two transanctions write to the same object, but then the earlier one aborts.

####Strict

Whenever a transaction T **writes** to an object, no other transaction can read or write to that object until T has committed or aborted.

![](http://os9hogvk5.bkt.clouddn.com/21_4.jpg)

###Protocols and Properties

Protocols are used for implementing properties. They just make sure that no bad thing happens, but some good things are also forbidden. 

To reduce the good things forbidden, we want to minimize the difference between the what the property (serializability or abort) allows and what the protocol allows.

####Locking-Based Protocols

- read locks – can be shared (S)
- write locks – must be exclusive (X)

If every transaction require all locks at the beginning , and only release the lock after commit, conflict-serializability is enforced, since there is no interleaving. 

If we allow them to acquire locks when needed, it can cause deadlock and no transaction starts. If we allow them to release the lock before commit when not need it, it can cause cascading abort if it aborts later.

####Two-Phase Locking (2PL)

Allow acquiring locks when needed, release locks when not needed, and guarantee conflict-serializability:

- Phase 1: only acquire locks 
- Phase 2: only release locks

![](http://os9hogvk5.bkt.clouddn.com/22_2.jpg)

Pros and cons

- Conservative: no deadlocks, but need to know all objects up front, and has less concurrency
- Strict: no cascading aborts, but has less concurrency

"2PL" can refer to all 4 combinations of conservation and strictness, or only to the non-strict and non-conservative version. In this course, "2PL" without indication refers to **strict conservative 2PL**.

In practice, strict non-conservative 2PL is the most commonly used protocol. We have to deal with deadlocks in that case.

####Deadlock Prevention

Assign priorities based on timestamps. Older transactions, *i.e.* lower timestamps have higher priorities. The transaction with higher priority never aborts.

If Ti wants a lock that Tj holds:

- For **wound-wait** policy, if Pi > Pj, Tj aborts; else Ti waits
- For **wait-die** policy, if Pi > Pj, Ti waits; else Ti aborts (*Pro*: transaction that hold locks never abort. *Con*: younger transactions that cause deadlocks are aborted repeatedly)

####Deadlock Detection

Create a waits-for graph, where nodes are transactions T, and the edge from i to j means Ti waits for Tj. While a circle in the graph is detected, abort one of the transactions.

####Phantom Problem

If T1 is only interested in "red" things, it only requires a lock on them. Then T2 inserts another "red" thing into the DB. After that if T1 reads all "red" things again, it will find that the number of them is increased, *i.e.* an unrepeateable read is produced.

####Index Locking

To solve the phantom problem, we can lock more pages than we currently need, so that no "red" can be inserted when T1 hasn't committed.

For a B+ tree, that means we should not only lock the data entry. The easiest way is to lock all index pages that have been traversed, including the root node. However, that makes the root node and some nodes near the root unaccessible for all other transactions, and makes the performance poor.

Instead, when we move to high level nodes of the tree, we can realease nodes near to the root, though it violates 2PL. 

For insert, simply release the lock on the parent when we reach the child. For insert and delete, what we have to find is the **safe node**: changes will not **propagate up beyond** this node. All locks on its ancestor can be released.

Requirement for safe node:

- Insert: node is not full
- Delete: node is not half-empty

![](http://os9hogvk5.bkt.clouddn.com/23_1.jpg)

Search 38:

1. Acquire lock on A
2. Acquire lock on B; release lock on A
3. Acquire lock on C; release lock on B
4. Acquire lock on D; release lock on C
5. Find 38, release all locks

Insert 45 (assume that only insert one entry):

1. Acquire lock on A
2. Acquire lock on B; since B is not full and thus is safe, release lock on A
3. Acquire lock on C; since C is full and thus is not safe (may propagate up after insertion), hold lock on B
4. Acquire lock on D, insert, release all locks

####Multiple-Granularity Locks

Put database-tables-pages-tuples in one tree. We can lock on any level (node).

Use **intention locks** in this protocol. To get S lock, must hold IS (intention-S) lock on the parent node; to get X lock, must hold IX lock on the parent node.