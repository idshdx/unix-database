# shell-database
The world’s simplest database, implemented as two Bash functions


````
#!/bin/bash


db_set () {
  echo "$1,$2" >> database
}
db_get () {
  grep "^$1," database | sed -e "s/^$1,//" | tail -n 1
}
````
These two functions implement a key-value store. You can call `db_set` key value,
which will store key and value in the database. The key and value can be (almost)
anything you like—for example, the value could be a JSON document. 
You can then call `db_get` key, which looks up the most recent value associated with that particular
key and returns it.


And it works:
````
$ db_set 123456 '{"name":"Test","skills":["one","two"]}'
$ db_set 42 '{"name":"Andreu","skills":["Bash"]}'
$ db_get 42
{"name":"Andrei","skills":["Bash"]}
````

The underlying storage format is very simple: a text file where each line contains a
key-value pair, separated by a comma (roughly like a CSV file, ignoring escaping
issues). Every call to `db_set` appends to the end of the file, so if you update a key several
times, the old versions of the value are not overwritten—you need to look at the
last occurrence of a key in a file to find the latest value (hence the tail -n 1 in
db_get):
````
$ db_set 42 '{"name":"Test","attractions":["Javascript"]}'
$ db_get 42
{"name":"Andrei","attractions":["Javascript"]}
$ cat database
123456,{"name":"Test","attractions":["one","two"]}
42,{"name":"Andrei","attractions":["Bash"]}
42,{"name":"Andrei","attractions":["Javascript"]}
````

The `db_set` function actually has pretty good performance for something that is so
simple, because appending to a file is generally very efficient.

On the other hand, the `db_get` function has terrible performance if you have a large
number of records in your database. Every time you want to look up a key, `db_get`
has to scan the entire database file from beginning to end, looking for occurrences of
the key. In algorithmic terms, the cost of a lookup is O(n): if you double the number
of records n in your database, a lookup takes twice as long.


