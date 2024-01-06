# Roster

## Table of Content
- [Project Overview](#project-overview)
- [Data Sources](#data-sources)
- [Data Model](#data-model)
- [Tools](#tools)
- [Code](#code)
- [Code Description](#code-description)
- [Results](#results)
  

## Project Overview
The purpose of this project is to demonstrate how we can write a python application that  will read roster data in a JSON format, parse the file, and then produce an SQLite database that contains a User, Course, and Member table and populate the tables from the data file. This is a type of data model relationship that has many to many relationships and there is no separate primary key. As a result, we add a connection table (also called junction table) with two foreign keys. This project described a relationship where there is users and courses, one user is a member of many courses and  each course has many users. The relationship between the courses and the users (ie the membership of the relatioship) is many on bpth ends. We can't just model directly so we create a junction table called member. We still have the course and the user table and this time they have primary keys. Then the member table has each row that has two foreign keys.

### Data Sources
The primary data used for this project is "roster_data.json" file containing a user, course and role. this data is gotten from "Python for Everybody" autograder assignment in [Coursera](https://www.coursera.org/).

### Data Model
![data model](https://github.com/chinenyejuliet/Roster/assets/142748509/2c38b40e-772f-4203-8255-2c5548a58ac4)

### Tools
- python3
- SQlite
- json

### Code
```
-python3
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

fname = input('Enter file name: ')
if len(fname) < 1:
    fname = 'roster_data.json'


str_data = open(fname).read()
json_data = json.loads(str_data)

for entry in json_data:

    name = entry[0]
    title = entry[1]
    role = entry[2]

    print((name, title,role))

    cur.execute('''INSERT OR IGNORE INTO User (name)
        VALUES ( ? )''', ( name, ) )
    cur.execute('SELECT id FROM User WHERE name = ? ', (name, ))
    user_id = cur.fetchone()[0]

    cur.execute('''INSERT OR IGNORE INTO Course (title)
        VALUES ( ? )''', ( title, ) )
    cur.execute('SELECT id FROM Course WHERE title = ? ', (title, ))
    course_id = cur.fetchone()[0]

    cur.execute('''INSERT OR REPLACE INTO Member
        (user_id, course_id,role) VALUES ( ?, ?, ? )''',
        ( user_id, course_id, role ) )

    conn.commit()
    
str_data.close()
```
```
-sql
SELECT User.name,Course.title, Member.role FROM 
    User JOIN Member JOIN Course 
    ON User.id = Member.user_id AND Member.course_id = Course.id
    ORDER BY  Course.title DESC, Member.role DESC LIMIT 10;
```
### Code Description
- we import json and sqlite3 modules
- we make a database connection to create database 'rosterdb.sqlite' and also create a 'cursor()' connection to enable sending and retrieving commands to the database.
- we drop table if exists to avoid a traceback error incase we are creating a table that already exists.
- we create user, course and member table with user and course tables having primary keys that are automatically incremented while member table has two foreign keys.
- Now we prompt the user to input a json file as fname but if the user wants to maintain reading roster_data.json the he/she should hit enter (whose len < 1), open the file and read it. We parse the json file through json.loads and we get a list objects. It's a list of lists so we iterate over it to get the values.
- We use the 'cursor()' method of the connection to send an 'INSERT OR IGNORE' command to the database, inserting data into both the 'user' table and the 'title' table. This SQL statement inserts a new record into the 'User' table and the 'Course' table with the specified 'name' and 'title only if there is no existing record with the same 'name' and 'title. This is because during the creation of the table they were specified as unique values.
- 'VALUES ( ? )' this code is usually written this way with a tuple containing the value to be inserted to avoid SQL injection.
- WE use 'cursor()' method of the connection to send an 'SELECT' command to the database, retrieving the course_id and user_id from the first column of 'User' and 'Course' tables.
- We also 'INSERT OR REPLACE' values into the member table
- Then we 'commit()' to save changes in the database and close the file.
- So now we are going to use 'SELECT User.name,Course.title, Member.role FROM 
    User JOIN Member JOIN Course 
    ON User.id = Member.user_id AND Member.course_id = Course.id
    ORDER BY User.name DESC, Course.title DESC, Member.role DESC LIMIT 10;' to join all those together into one big long row. We use the ON clause and the Member.user_id is equal to the user's id and the members and the courses, the membership course id is equal to the course's id. Member.course_id equals the course's id. We're going to order by the course title then the member role. The way these. And then member role descending and then the user name

### Results
![result](https://github.com/chinenyejuliet/Roster/assets/142748509/c6f19a0a-3742-4118-a257-99e2876c6516)

