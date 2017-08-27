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
```