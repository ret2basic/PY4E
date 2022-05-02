# Databases

## Autograder: Single Table SQL

### Instruction

Create a SQLITE database or use an existing database and create a table in the database called "Ages":

```sql
CREATE TABLE Ages ( 
  name VARCHAR(128), 
  age INTEGER
)
```

Then make sure the table is empty by deleting any rows that you previously inserted, and insert these rows and only these rows with the following commands:

```sql
DELETE FROM Ages;
INSERT INTO Ages (name, age) VALUES ('Caylin', 23);
INSERT INTO Ages (name, age) VALUES ('Imogem', 28);
INSERT INTO Ages (name, age) VALUES ('Kile', 25);
INSERT INTO Ages (name, age) VALUES ('Joelle', 34);
INSERT INTO Ages (name, age) VALUES ('Tillie', 40);
INSERT INTO Ages (name, age) VALUES ('Ruta', 15);
```

Once the inserts are done, run the following SQL command:

```sql
SELECT hex(name || age) AS X FROM Ages ORDER BY X
```

Find the first row in the resulting record set and enter the long string that looks like `53656C696E613333`.

**Note:** This assignment must be done using SQLite - in particular, the `SELECT` query above will not work in any other database. So you cannot use MySQL or Oracle for this assignment.

### Implementation

```python
# firstdb.py
import sqlite3

conn = sqlite3.connect('firstdb.sqlite')
cur = conn.cursor()

cur.execute("""
    CREATE TABLE Ages ( 
        name VARCHAR(128), 
        age INTEGER
    )
""")
cur.execute('DELETE FROM Ages')
conn.commit()

cur.execute("INSERT INTO Ages (name, age) VALUES ('Caylin', 23)")
cur.execute("INSERT INTO Ages (name, age) VALUES ('Imogem', 28)")
cur.execute("INSERT INTO Ages (name, age) VALUES ('Kile', 25)")
cur.execute("INSERT INTO Ages (name, age) VALUES ('Joelle', 34)")
cur.execute("INSERT INTO Ages (name, age) VALUES ('Tillie', 40)")
cur.execute("INSERT INTO Ages (name, age) VALUES ('Ruta', 15)")
conn.commit()

cur.execute('SELECT hex(name || age) AS X FROM Ages ORDER BY X')
for row in cur:
    print(row)
```

## Quiz: Single Table SQL

### Question 1

Structured Query Language (SQL) is used to (check all that apply)

**Answers:**

- [ ] Create a table
- [ ] Insert data
- [ ] Delete data
- [ ] Check Python code for errors

### Question 2

Which of these is the right syntax to make a new table?

**Answers:**

- [x] CREATE TABLE people;
- [ ] MAKE DATASET people;
- [ ] MAKE people;
- [ ] CREATE people;

### Question 3

Which SQL command is used to insert a new row into a table?

**Answers:**

- [x] INSERT INTO
- [ ] INSERT ROW
- [ ] INSERT AFTER
- [ ] ADD ROW

### Question 4

Which command is used to retrieve all records from a table?

**Answers:**

- [x] SELECT * FROM Users
- [ ] RETRIEVE all FROM User
- [ ] RETRIEVE * FROM Users
- [ ] SELECT all FROM Users

### Question 5

Which keyword will cause the results of the query to be displayed in sorted order?

**Answers:**

- [ ] GROUP BY
- [x] ORDER BY
- [ ] WHERE
- [ ] None of these

### Question 6

In database terminology, another word for table is

**Answers:**

- [x] relation
- [ ] row
- [ ] field
- [ ] attribute

### Question 7

In a typical online production environment, who has direct access to the production database?

**Answers:**

- [ ] Project Manager
- [x] Database Administrator
- [ ] Developer
- [ ] UI/UX Designer

### Question 8

Which of the following is the database software used in this class?

**Answers:**

- [ ] MySQL
- [ ] Oracle
- [ ] SQL Server
- [ ] Postgres
- [x] SQLite

### Question 9

What happens if a DELETE command is run on a table without a WHERE clause?

**Answers:**

- [ ] The first row of the table will be deleted
- [ ] All the rows without a primary key will be deleted
- [x] All the rows in the table are deleted
- [ ] It is a syntax error

### Question 10

Which of the following commands would update a column named "name" in a table named "Users"?

**Answers:**

- [ ] Users->name = 'new name' WHERE ...
- [ ] Users.name='new name' WHERE ...
- [ ] UPDATE Users (name) VALUES ('new name') WHERE ...
- [x] UPDATE Users SET name='new name' WHERE ...

### Question 11

What does this SQL command do?

```sql
SELECT COUNT(*) FROM Users
```

**Answers:**

- [x] It counts the rows in the table Users
- [ ] It is a syntax error
- [ ] It only retrieves the rows of Users if there are at least two rows
- [ ] It adds a COUNT column to the Users table

## Autograder: Counting Email in a Database

### Description

This application will read the mailbox data (`mbox.txt`) and count the number of email messages per organization (i.e. domain name of the email address) using a database with the following schema to maintain the counts.

```sql
CREATE TABLE Counts (org TEXT, count INTEGER)
```

When you have run the program on `mbox.txt` upload the resulting database file above for grading.
If you run the program multiple times in testing or with dfferent files, make sure to empty out the data before each run.

You can use this code as a starting point for your application: http://www.py4e.com/code3/emaildb.py.

The data file for this application is the same as in previous assignments: http://www.py4e.com/code3/mbox.txt.

Because the sample code is using an UPDATE statement and committing the results to the database as each record is read in the loop, it might take as long as a few minutes to process all the data. The commit insists on completely writing all the data to disk every time it is called.

The program can be speeded up greatly by moving the commit operation outside of the loop. In any database program, there is a balance between the number of operations you execute between commits and the importance of not losing the results of operations that have not yet been committed.

### Implementation

```python
import sqlite3

conn = sqlite3.connect('emaildb.sqlite')
cur = conn.cursor()

cur.execute('DROP TABLE IF EXISTS Counts')

cur.execute('''
CREATE TABLE Counts (org TEXT, count INTEGER)''')

fname = 'mbox.txt'
with open(fname) as fh:
    for line in fh:
        if not line.startswith('From: '): continue
        pieces = line.split('@')
        org = pieces[1]
        cur.execute('SELECT count FROM Counts WHERE org = ? ', (org,))
        row = cur.fetchone()
        if row is None:
            cur.execute('''INSERT INTO Counts (org, count)
                    VALUES (?, 1)''', (org,))
        else:
            cur.execute('UPDATE Counts SET count = count + 1 WHERE org = ?',
                        (org,))
        conn.commit()

# https://www.sqlite.org/lang_select.html
sql_query = 'SELECT org, count FROM Counts ORDER BY count DESC LIMIT 10'

for row in cur.execute(sql_query):
    print(str(row[0]), row[1])

cur.close()
```

## Autograder: Multi-Table Database - Tracks

### Description

This application will read an iTunes export file in XML and produce a properly normalized database with this structure:

```sql
CREATE TABLE Artist (
    id  INTEGER NOT NULL PRIMARY KEY AUTOINCREMENT UNIQUE,
    name    TEXT UNIQUE
);

CREATE TABLE Genre (
    id  INTEGER NOT NULL PRIMARY KEY AUTOINCREMENT UNIQUE,
    name    TEXT UNIQUE
);

CREATE TABLE Album (
    id  INTEGER NOT NULL PRIMARY KEY AUTOINCREMENT UNIQUE,
    artist_id  INTEGER,
    title   TEXT UNIQUE
);

CREATE TABLE Track (
    id  INTEGER NOT NULL PRIMARY KEY 
        AUTOINCREMENT UNIQUE,
    title TEXT  UNIQUE,
    album_id  INTEGER,
    genre_id  INTEGER,
    len INTEGER, rating INTEGER, count INTEGER
);
```

If you run the program multiple times in testing or with different files, make sure to empty out the data before each run.

You can use this code as a starting point for your application: http://www.py4e.com/code3/tracks.zip. The ZIP file contains the Library.xml file to be used for this assignment. You can export your own tracks from iTunes and create a database, but for the database that you turn in for this assignment, only use the Library.xml data that is provided.

To grade this assignment, the program will run a query like this on your uploaded database and look for the data it expects to see:

```sql
SELECT Track.title, Artist.name, Album.title, Genre.name 
    FROM Track JOIN Genre JOIN Album JOIN Artist 
    ON Track.genre_id = Genre.ID and Track.album_id = Album.id 
        AND Album.artist_id = Artist.id
    ORDER BY Artist.name LIMIT 3
```

### Implementation

```python
import xml.etree.ElementTree as ET
import sqlite3

conn = sqlite3.connect('trackdb.sqlite')
cur = conn.cursor()

# Make some fresh tables using executescript()
cur.executescript('''
DROP TABLE IF EXISTS Artist;
DROP TABLE IF EXISTS Album;
DROP TABLE IF EXISTS Track;

CREATE TABLE Artist (
    id  INTEGER NOT NULL PRIMARY KEY AUTOINCREMENT UNIQUE,
    name    TEXT UNIQUE
);

CREATE TABLE Album (
    id  INTEGER NOT NULL PRIMARY KEY AUTOINCREMENT UNIQUE,
    artist_id  INTEGER,
    title   TEXT UNIQUE
);

CREATE TABLE Track (
    id  INTEGER NOT NULL PRIMARY KEY 
        AUTOINCREMENT UNIQUE,
    title TEXT  UNIQUE,
    album_id  INTEGER,
    genre_id INTEGER,
    len INTEGER,
    rating INTEGER,
    count INTEGER
);

CREATE TABLE GENRE (
    ID  INTEGER NOT NULL PRIMARY KEY
        AUTOINCREMENT UNIQUE,
    name  TEXT UNIQUE  
)
''')


fname = 'Library.xml'

# <key>Track ID</key><integer>369</integer>
# <key>Name</key><string>Another One Bites The Dust</string>
# <key>Artist</key><string>Queen</string>
def lookup(d, key):
    found = False
    for child in d:
        if found : return child.text
        if child.tag == 'key' and child.text == key :
            found = True
    return None

stuff = ET.parse(fname)
all = stuff.findall('dict/dict/dict')
print('Dict count:', len(all))
for entry in all:
    if ( lookup(entry, 'Track ID') is None ) : continue

    name = lookup(entry, 'Name')
    artist = lookup(entry, 'Artist')
    album = lookup(entry, 'Album')
    count = lookup(entry, 'Play Count')
    rating = lookup(entry, 'Rating')
    length = lookup(entry, 'Total Time')
    genre = lookup(entry, 'Genre')

    if name is None or artist is None or album is None or genre is None: 
        continue

    print(name, artist, album, count, rating, length, genre)

    cur.execute('''INSERT OR IGNORE INTO Artist (name) 
        VALUES ( ? )''', ( artist, ) )
    cur.execute('SELECT id FROM Artist WHERE name = ? ', (artist, ))
    artist_id = cur.fetchone()[0]

    cur.execute('''INSERT OR IGNORE INTO Genre (name) 
        VALUES ( ? )''', ( genre, ))
    cur.execute('SELECT id FROM Genre WHERE name = ? ', (genre, ))
    genre_id = cur.fetchone()[0]

    cur.execute('''INSERT OR IGNORE INTO Album (title, artist_id) 
        VALUES ( ?, ? )''', ( album, artist_id ) )
    cur.execute('SELECT id FROM Album WHERE title = ? ', (album, ))
    album_id = cur.fetchone()[0]

    cur.execute('''INSERT OR REPLACE INTO Track
        (title, album_id, len, rating, count, genre_id) 
        VALUES ( ?, ?, ?, ?, ?, ? )''', 
        ( name, album_id, length, rating, count, genre_id ) )

    conn.commit()
```

## Quiz: Data Modeling

### Question 1

What is the primary added value of relational databases over flat files?

- [ ] Ability to execute JavaScript in the file
- [ ] Ability to execute Python code within the file
- [x] Ability to scan large amounts of data quickly
- [ ] Ability to store data in a format that can be sent across a network
- [ ] Ability to quickly convert data to HTML

### Question 2

What is the purpose of a primary key?

- [ ] To track the number of duplicate values in another column
- [ ] To look up a row based on a string that comes from outside the program
- [ ] To point to a particular row in another table
- [x] To look up a particular row in a table very quickly

### Question 3

Which of the following is NOT a good rule to follow when developing a database model?

- [ ] Model each "object" in the application as one or more tables
- [ ] Use integers as primary keys
- [ ] Never repeat string data in more than one table in a data model
- [x] Use a person's email address as their primary key

### Question 4

If our user interface (i.e., like iTunes) has repeated strings on one column of the user interface, how should we model this properly in a database?

- [ ] Encode the entire row as JSON and store it in a TEXT column in the database
- [ ] Put the string in the first row where it occurs and then put that row number in the column of all of the rest of the rows where the string occurs
- [ ] Put the string in the last row where it occurs and put the number of that row in the column of all of the rest of the rows where the string occurs
- [x] Make a table that maps the strings in the column to numbers and then use those numbers in the column
- [ ] Put the string in the first row where it occurs and then put NULL in all of the other rows

### Question 5

Which of the following is the label we give a column that the "outside world" uses to look up a particular row?


- [x] Logical key
- [ ] Foreign key
- [ ] Remote key
- [ ] Primary key
- [ ] Local key

### Question 6

What is the label we give to a column that is an integer and used to point to a row in a different table?

- [ ] Local key
- [ ] Primary key
- [ ] Remote key
- [ ] Logical key
- [x] Foreign key

### Question 7

What SQLite keyword is added to primary keys in a CREATE TABLE statement to indicate that the database is to provide a value for the column when records are inserted?

- [ ] AUTO_INCREMENT
- [x] AUTOINCREMENT     
- [ ] ASSERT_UNIQUE
- [ ] INSERT_AUTO_PROVIDE

### Question 8

What is the SQL keyword that reconnects rows that have foreign keys with the corresponding data in the table that the foreign key points to?

- [ ] COUNT
- [ ] CONNECT
- [ ] CONSTRAINT
- [ ] APPEND
- [x] JOIN

### Question 9

What happens when you JOIN two tables together without an ON clause?

- [ ] Leaving out the ON clause when joining two tables in SQLite is a syntax error
- [ ] The rows of the left table are connected to the rows in the right table when their primary key matches
- [ ] You get no rows at all
- [ ] You get all of the rows of the left table in the JOIN and NULLs in all of the columns of the right table
- [x] The number of rows you get is the number of rows in the first table times the number of rows in the second table

### Question 10

When you are doing a SELECT with a JOIN across multiple tables with identical column names, how do you distinguish the column names?

- [ ] tablename->columnname
- [ ] tablename['columnname']
- [x] tablename.columnname
- [ ] tablename/columnname

## Autograder: Many Students in Many Courses

### Description

This application will read roster data in JSON format, parse the file, and then produce an SQLite database that contains a User, Course, and Member table and populate the tables from the data file.

You can base your solution on this code: http://www.py4e.com/code3/roster/roster.py - this code is incomplete as you need to modify the program to store the role column in the Member table to complete the assignment.

Each student gets their own file for the assignment. Download this file and save it as roster_data.json. Move the downloaded file into the same folder as your roster.py program.

Once you have made the necessary changes to the program and it has been run successfully reading the above JSON data, run the following SQL command:

```sql
SELECT User.name,Course.title, Member.role FROM 
    User JOIN Member JOIN Course 
    ON User.id = Member.user_id AND Member.course_id = Course.id
    ORDER BY User.name DESC, Course.title DESC, Member.role DESC LIMIT 2;
```

The output should look as follows:

```
Zubair|si364|0
Zofia|si310|0
```

Once that query gives the correct data, run this query:

```sql
SELECT 'XYZZY' || hex(User.name || Course.title || Member.role ) AS X FROM 
    User JOIN Member JOIN Course 
    ON User.id = Member.user_id AND Member.course_id = Course.id
    ORDER BY X LIMIT 1;
```

You should get one row with a string that looks like `XYZZY53656C696E613333`.

### Implementation

```python
import json
import sqlite3

conn = sqlite3.connect('rosterdb.sqlite')
cur = conn.cursor()

# Do some setup
cur.executescript('''
DROP TABLE IF EXISTS User;
DROP TABLE IF EXISTS Member;
DROP TABLE IF EXISTS Course;

CREATE TABLE User (
    id     INTEGER NOT NULL PRIMARY KEY AUTOINCREMENT UNIQUE,
    name   TEXT UNIQUE
);

CREATE TABLE Course (
    id     INTEGER NOT NULL PRIMARY KEY AUTOINCREMENT UNIQUE,
    title  TEXT UNIQUE
);

CREATE TABLE Member (
    user_id     INTEGER,
    course_id   INTEGER,
    role        INTEGER,
    PRIMARY KEY (user_id, course_id)
)
''')

fname = 'roster_data.json'

# [
#   [ "Charley", "si110", 1 ],
#   [ "Mea", "si110", 0 ],

str_data = open(fname).read()
json_data = json.loads(str_data)

for entry in json_data:

    name = entry[0]
    title = entry[1]
    role = entry[2]

    # print((name, title, role))

    cur.execute('''INSERT OR IGNORE INTO User (name)
        VALUES ( ? )''', ( name, ) )
    cur.execute('SELECT id FROM User WHERE name = ? ', (name, ))
    user_id = cur.fetchone()[0]

    cur.execute('''INSERT OR IGNORE INTO Course (title)
        VALUES ( ? )''', ( title, ) )
    cur.execute('SELECT id FROM Course WHERE title = ? ', (title, ))
    course_id = cur.fetchone()[0]

    cur.execute('''INSERT OR REPLACE INTO Member
        (user_id, course_id, role) VALUES ( ?, ?, ? )''',
        ( user_id, course_id, role ) )

    conn.commit()

# cur.execute('''SELECT User.name,Course.title, Member.role FROM 
#     User JOIN Member JOIN Course 
#     ON User.id = Member.user_id AND Member.course_id = Course.id
#     ORDER BY User.name DESC, Course.title DESC, Member.role DESC LIMIT 2;''')
# for entry in cur:
#     print(entry)

cur.execute('''SELECT 'XYZZY' || hex(User.name || Course.title || Member.role ) AS X FROM 
    User JOIN Member JOIN Course 
    ON User.id = Member.user_id AND Member.course_id = Course.id
    ORDER BY X LIMIT 1;''')
for entry in cur:
    print(entry)
```

## Quiz: Many to Many

### Question 1

How do we model a many-to-many relationship between two database tables?

- [ ] We add 10 foreign keys to each table with names like artict_id_1, artist_id2, etc.
- [ ] We use a BLOB column in both tables
- [ ] We use the ARRAY column type in both of the tables
- [x] We add a table with two foreign keys

### Question 2

In Python, what is a database "cursor" most like?

- [ ] A Python dictionary
- [x] A file handle
- [ ] A function
- [ ] A method within a class

### Question 3

What method do you call  in an SQLIte cursor object in Python to run an SQL command?

- [ ] `socket()`
- [ ] `send()`
- [x] `execute()`
- [ ] `run()`

### Question 4

In the following SQL,

```sql
cur.execute('SELECT count FROM Counts WHERE org = ? ', (org, ))
```

what is the purpose of the `?` ?

- [ ] It is a search wildcard
- [ ] It is a syntax error
- [x] It is a placeholder for the contents of the "org" variable
- [ ] It allows more than one boolean operation in the WHERE clause

### Question 5

In the following Python code sequence (assuming `cur` is a SQLite cursor object),

```sql
cur.execute('SELECT count FROM Counts WHERE org = ? ', (org, ))
row = cur.fetchone()
```

what is the value in row if no rows match the WHERE clause?

- [ ] -1
- [ ] An empty list
- [ ] An empty dictionary
- [x] None

### Question 6

What does the LIMIT clause in the following SQL accomplish?

```sql
SELECT org, count FROM Counts 
   ORDER BY count DESC LIMIT 10
```

- [x] It only retrieves the first 10 rows from the table
- [ ] It only sorts on the first 10 characters of the column
- [ ] It reverses the sort order if there are more than 10 rows
- [ ] It avoids reading data from any table other than Counts

### Question 7

What does the `executescript()` method in the Python SQLite cursor object do that the normal `execute()` method does not do?

- [ ] It allows embedded JavaScript to be executed
- [ ] It allows database tables to be created
- [ ] It allows embeded Python to be executed
- [x] It allows multiple SQL statements separated by semicolons

### Question 8

What is the purpose of "OR IGNORE" in the following SQL:

```sql
INSERT OR IGNORE INTO Course (title) VALUES ( ? )
```

- [x] It makes sure that if a particular title is already in the table, there are no duplicate rows inserted
- [ ] It ignores errors in the SQL syntax for the statement
- [ ] It updates the created_at value if the title already exists in the table
- [ ] It ignores any foreign key constraint errors

### Question 9

What do we generally avoid in a many-to-many junction table?

- [x] An AUTOINCREMENT primary key column
- [ ] A logical key
- [ ] Two foreign keys
- [x] Data items specific to the many-to-many relationship
