Data modelling
==============

In the previous section, we saw how a "service" can be modelled as a "manager
of resources on behalf of a client" and how to construct references for these
managed "resources" for the purpose of interacting with the service via
transfer of "representations" -- the "REST" (REpresentational State Transfer)
approach.

In this section, we'll take off on the "todo list" API constructed in class
based on a global variable as the storage and consider how to store and
retrieve the information using a relational database in a persistent manner.
With the service constructed in class, restarting the server will cause it to
forget all the todo items it had stored earlier and that's what we want to
address.

The "resource" way of thinking about the service translates fairly easily onto
a relational database. We'll assume you've already dealt with relational
databases at a conceptual level, but haven't actually used one for data
modelling.

Tables
------

The core data structure in a relational database is the **table**, whose rows
capture related values of some aspect of the data. Hence "relational database".
In the case of the todo list, we can think of the todo list itself as a table,
whose rows give information about each of the items in the list. We saw that
each list item has a number of attributes to keep track of.

1. The URI or equivalently the ``id`` of a todo item. We can consider them
   to be equivalent because the URL of the todo item can be derived if we
   know the ``id`` of the item and vice versa.
2. The ``text`` of the TODO item.
3. The ``done`` state of the TODO item.

We could add more attributes to the above list, but that wouldn't lead us to
any new concepts beyond what we used for the above, so you can take that as an
exercise to extend it with more information such as "creation date-time",
"completion date-time", "priority" etc.

.. csv-table:: TODOList
    :header: "id", "text", "status"

    1, "Model the todolist app as resources", "complete"
    2, "Create a relational data model", "pending"
    3, "Specify the schema for the data model in SQL", "pending"
    4, "Create the tables in sqlite3", "pending"
    5, "Implement the service handlers to use the sqlite3 db instead of the global variable", "pending"

.. index::
    single: schema

Notice that in the above table, we're considering our "id" column to be an
integer, the "text" column to have, well, text values and the "status" column
to also have text value with only two values allowed -- "complete" or
"pending". The "status" column is a effectively a boolean and could also have
been modelled as an integer with ``0`` standing for "incomplete" and ``1``
standing for "completeyes". Similarly, you may want to use a string as an
identifier for a todo item as well, instead of an integer. Such a description
of the table is called its "schema".

.. index::
   single: schema migration

There are always such choices to be made when constructing a data model.
Sometimes these choices don't feel very consequential, such as perhaps in our
case so far, but if there is more information available, the choices may have
real consequences. For example, suppose we know that there is an upcoming
feature for our todolist app where the status of the todolist item can be an
arbitrary word given by the user, such as "pending", "WIP", "done", "archived",
etc. In such a case, the text representation for that column has a clear
advantage over the integer/boolean representation, since we'll then have to
overhaul the table and change its schema (called a "schema migration").

Now that we've acknowledged that our app will evolve over time and so might
our table "schema", we should perhaps keep some metadata about our schema
-- its "version number" to start with.

.. csv-table:: metadata
    :header: "schema", "version", "date"

    "todolist", 1, "2024-08-25"

.. note::  The metadata table itself has a schema, but we expect it to change
   much less frequently than the other table schemas. While we've only captured
   the version number for simplicity, it is common to maintain a table of
   instructions on how to migrate from version :math:`n` to version
   :math:`n+1`, along with the current version, so that the migration steps can
   be run automatically. We'll not go there yet.

SQL
---

The "Structured Query Language" -- abbreviated "SQL" and pronounced "sequel" --
is a ubiquitous language used in describing data and interacting with
relational databases. The advantage of SQL is that it abstracts the details of
how the data is actually stored by a given database, and thus permits the
programmer to think and work at the level of the data they need to work with.

SQLite_ is a ubiquitous, simple yet powerful such database which stores all its
data in a single file and makes it available for manipulation via SQL.
Different databases have slightly different variations of the SQL language they
support, but these differences are far less important than the substantial
commonality that exists between them.


.. _sqlite3 documentation:
.. _sqlite3:
.. _sqlite: https://www.sqlite.org/index.html 

Other common larger scale databases in use include PostgreSQL_ and MySQL_. In
my opinion, of the relational databases available today, PostgreSQL_ is perhaps
the best of the cadre w.r.t. its engineering and the ecosystem of extensions
available for it.

.. _PostgreSQL: https://www.postgresql.org/
.. _MySQL: https://www.mysql.com/

There are also a class of databases called "NoSQL" databases which came up
during a period of (in my opinion) irrational backlash against the perceived
complexity of SQL databases. Our general advice is that if you don't know
enough to choose a database, choose one which has a SQL interface and don't
choose a "NoSQL" database (such as "MongoDB").

SQLite3
-------

As mentioned, we'll be using the "SQLite v3" as a simple database storage for
our todolist app's backend.

Python comes with the ``sqlite3`` package preinstalled usually. In case your
installation does not have it, you can always install it with ``pip install
sqlite3``.

The `sqlite3 documentation`_ is the canonical place to go to for information
about the supported commands. Its documentation is excellent even for beginners
and it provides "railroad diagrams" that illustrate the syntax.

.. note:: SQL syntax is not case sensitive for its keywords.

The process for opening a sqlite3 database and sending it commands to manipulate
and query data is as follows --

1. "Connect" to the database. The language used here generalizes different
   locations for the database -- a) in-memory, b) on disk or c) on another
   computer over the network. This step gets you a database connection.

2. When you want to do something with the database, you create a "cursor"
   object from the connection and ask it to execute your SQL commands given as
   a string.

3. Once you're done with the cursor object you created, you close the cursor.

The above is the simplest of usage scenarios and is a common mode of usage
across different databases. The process might vary a bit, for example, when you
have to deal with a long running "transaction".

.. admonition:: **sqlite3 repl**

    ``sqlite3`` comes with a repl you can run from the shell using ``sqlite3``.
    You can run SQL commands as well as what are called "meta commands" which
    start with a period "." character. When writing SQL on the repl, the SQL statements
    can be multi-line and are terminated by a ";".

.. code:: SQL

    create table todolist (
        id INTEGER PRIMARY KEY,
        text TEXT,
        status TEXT
    );

The above SQL command does what it looks like it is supposed to do -- create a
table with three columns with the given types. The part that needs explanation
is ``PRIMARY KEY``. This phrase when used next to the type of a column
indicates that that column serves as a unique index to identify a row. So the
database will ensure that the table will not have more than one row with the
same "id" value in our case.

.. note:: Within the context of a database, such a "create table" statement is called
   the "schema" of the table named "todolist". 

.. _create table: https://www.sqlite.org/lang_createtable.html


Insert data
-----------

Now let's consider the commands to insert new rows into our brand new ``todolist``
table.

.. code:: SQL

    insert into todolist values
        (1, 'Model the todolist app as resources', 'complete'),
        (2, 'Create a relational data model', 'pending'),
        (3, 'Specify the schema for the data model in SQL', 'pending'),
        (4, 'Create the tables in sqlite3', 'pending'),
        (5, 'Implement the service handlers to use the sqlite3 db instead.', 'pending');

Refer to the `insert into`_ documentation on the syntax. For our simple table, the following hold --

.. _insert into: https://www.sqlite.org/lang_insert.html

1. The values supplied within parentheses are (and must be) in the same order
   in which the columns we declared in the schema (i.e. "create table"
   statement).

2. String values are given enclosed in single-quote characters. If a string
   itself is to include the single quote character, use two single-quotes
   instead -- like in ``'this SQL string has a ''single-quoted'' part'``.

3. The types of the values will be cast to what we specified in the schema. So
   if we'd declared "id" to be "TEXT" but gave a number when inserting data,
   the number will be converted into a string and stored.

4. There is a hidden column available called ``rowid`` which is also an integer and which
   SQLite can auto insert for you, so in our case, we don't really need an "id" column.

5. Supposing we insert a row with an "id" that already exists in the table, it is considered
   an error, because we've marked the "id" column as being the ``PRIMARY KEY``.

Retrieve rows
-------------

Retrieving rows from a table is done using the `select statement`_. For example, to retrieve
the set of rows of completed todo items, we can issue the following command --

.. _select statement: https://www.sqlite.org/lang_select.html

.. code:: SQL

    select * from todolist
    where status = 'complete';

In the above case, the "*" indicates "get me all the columns in the table". While this is useful
for debugging and testing on the sqlite3 repl, it is better to be specific about the information
we need. That way, if the schema grew to 10 columns and we only needed two in the first place, we
don't end up wasting 80% of the data fetched.

.. code:: SQL

    select text, status from todolist
    where status = 'complete';

While such usage of the select statement is simple to understand, much of the
complexity of working with tables using SQL lies in constructing select
statements. In particular querying information from multiple tables (called
"join operations") presents much complexity and tricky performance
considerations when tables become large.

Update rows
-----------

To mark the "create a relational data model" row as "complete", we use the
update_ statement.

.. _update: https://www.sqlite.org/lang_update.html

.. code:: SQL

    update todolist
    set status = 'complete'
    where id = 5;

Note the following --

1. The ``where id = 5`` part identifies the rfws whose ``status`` field need to
   be marked as ``complete``.

2. In principle, there could be more than one row identified by the given
   constraints. All of them will be updated by the statement. In our case
   though, since ``id`` is the "primary key" for the table, the value ``5`` is
   guaranteed to uniquely identify one row, if it exists. 

   .. admonition:: **Warning**

        In general, beware when you make update statements which they're
        destructive updates and you might accidentally match more rows than you
        intended to. Remember the "precision" and "recall" concepts. You want
        high precision **and** recall for your update statements, but the
        precision is more important than the recall, since you can find out
        about rows that have not been modified and issue new commands to modify
        them. If your selection is has low precision though, you'll have
        modified some rows unintentionally and it can be hard to determine
        which rows were affected.

3. ``update`` statements cannot add or remove columns. That is considered a
   change in schema and must not be done without careful thought.

Indices
-------

Consider the ``select`` statement we wrote earlier --

.. code:: SQL

    select text, status from todolist
    where status = 'complete';

We can imagine that the database engine that runs this program steps through the rows
of the example, examining each row for the conditions indicated in the ``where`` clause
and returning the requested columns when there is a match. We might think of it as
equivalent to the python "list comprehension" --

.. code:: python
    
    [ (item["text"], item["status"])
      for item in g_todolist
      if item["status"] == "complete"
    ]

The list comprehension is a useful beginning mental model of querying tables
to have in mind. However, as computer scientists, we can quickly notice that
this is an inefficient means of retrieval since it has to go through the
entire list for each query. What if the list has a million items and only 5
of them have been completed?

.. index::
   single: index tables

To speed up such cases, SQL databases can construct auxiliary tables, called
"index tables" or "indices" for short, which maintain additional information
that helps then run such ``select`` queries fast -- often in logarithmic time
complexity. This is a classic case of "trade off some extra storage space for a
great reduction in time".

These indices are not automatically created though. Since every new index table
places demands on compute and storage, we need to tell the engine explicitly 
which indices need to be created for our particular uses. 

.. note:: A good rule of thumb is to list out all the queries you make,
   identify the ones that can be expensive without an index and only create
   indices over the columns relevant to those queries.

In this case, if we wish to create an index for the "status" column so we
can quickly locate the pending items (assuming these are far fewer than
the completed items), we can do so like this --

.. code:: SQL

    create index idx_todolist_status
    on todolist ( status );

While creating such indices manually seems onerous, the saving grace is that
these indices are used by the engine automatically when running ``select``
queries and we don't need to explicitly specify which index to use to speed
things up.

.. admonition:: **Performance note**

   Indices are most effective when there is high information content in a
   column. This is why "primary key" columns and "unique" columns benefit the
   most, since if we know the value of this column, then we know exactly which
   row we need to be looking at. In our case, the "status" column makes for a
   weak index because it can take only one of two values and therefore if there
   is an even split between the number of "complete" items and "pending" items,
   the advantage we gain is not all that much over a simple linear search.


sqlite3 and python
------------------

The following python code does these steps -- you can try these in the python REPL.
We're creating our "todolist table" in this step using the `create table`_ command.

.. code:: python

   import sqlite3

   db = sqlite3.connect("todolist.db")
   # Now, if you didn't have a file called "todolist.db" in the current
   # directory, one will be created and opened as a sqlite3 database.
   # If one exists already, sqlite3 will try to open it as a database.
   # In case it isn't an sqlite3 database, this step will raise an exception.

   cursor = db.cursor()
   cursor.execute("""
   create table todolist (
        id INTEGER PRIMARY KEY,
        text TEXT,
        status TEXT
    )
    """)
    cursor.close()

    # The above way has a problem. Suppose there was an exception raised during
    # the `execute` step, then we'll miss closing the cursor. To avoid this,
    # we can use the python `with` clause like this --

    from contextlib import closing

    with closing(db.cursor()) as cur:
        cur.execute("""...""")

    # The above `with` clause will ensure that the cursor opened is closed
    # whether or not the SQL statement completes successfully.
    # We'll use this approach going forward.


Here is an example of how we would typically write functions that call into the
database to retrieve items. We do not construct SQL statements using string
concatenation. Instead we mark the variable parts of the statements using `?`
and supply arguments using a separate python list of arguments. 

.. code:: python

    def items_by_status(db, status):
        with closing(db.cursor()) as cur:
            rows = cur.execute("select text, status from todolist where status = ?", [status])
            return rows.fetchall()


You can also used dictionaries to supply values for named parameters like this -

.. code:: python

    def items_by_status(db, status):
        with closing(db.cursor()) as cur:
            rows = cur.execute("""
                select text, status
                from todolist
                where status = :status
            """, { "status": status })
            return rows.fetchall()

The ``:status`` marks the named parameter in the SQL statement. The ``rows``
object returned by ``cur.execute`` is an iterator and ``rows.fetchall()`` is
essentially the same as ``[r for r in rows]``. Naturally, such named parameters
can only be used within programming languages and not at the sqlite3 repl.

.. admonition:: **SQL injection attack**

    Many early web programs used to present web forms, take values from them,
    and construct SQL queries using string concatenation and run the queries
    and return the results. Once some of these services started being
    commercially signficant, hackers with malicious intent would try to input
    SQL expression fragments into these web forms and try to disrupt the SQL
    query to retrieve more information than they're authorized for. This is
    called a "SQL injection attack" and is pretty much why the positional and
    named parameters exist in the programming language APIs for SQL databases.
    The API implementation will construct the SQL query in a safe manner behind
    the scenes that won't permit inadvertent "SQL injection attacks" due to
    programmer error.

Task
----

Complete your "TODO List" backend but now use a SQLite3 backed database
instead of the global variable ``g_todolist`` which we used in class.






























