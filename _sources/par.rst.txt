Concurrency and parallelism
===========================

The internet's architecture means that machines can (in general) talk to
each other if they know each others' IP address, which they can often
get if they know the relevant domain names. This means that a "server"
computer running an application as a service can be hit with a request
from any number of "client" computers at any time.

This poses some important questions to ask of our application and
scenarios to test rigorously for production use.

1. What happens when our service handlers are hit "at the same time"
   by two different requests?

2. If our service handler modifies some piece of data in our database,
   then what happens when another request comes in when the earlier
   request is in the process of being handled and the database is
   still working on the previous statement?

3. What happens if our service request handler needs to make a request to
   another service to fulfil what was asked of it and that service is doing
   some destructive operation while another request comes in?

You can see how the "simplicity" of what we've been doing in our backend code
can cross over quickly into "damn, this is broken" territory in such a
scenario!

A number of approaches have evolved to deal with these concurrency issues
over time while, for the most part, retaining programming simplicity.

We look at two -- coroutining on a single thread, and database transactions.

Coroutining on a single thread
------------------------------

We know that our computers can multiple processes in parallel -- as in two
instructions from two programs can be in the process of execution by the 
processor **at the same time** because our processors have multiple "cores",
with each "core" usually capable of service two "threads".

In the early days of the internet, every request from a client would be handled
by spawning off a process and these issues of concurrency were demonic to deal with
without proper engineering and testing. These days though, there is general recognition
that much of the hard work is delegated to other systems and the "server" process 
spends most of its time idling, waiting for a request, rather than servicing a
request.

To captialize on this observation, systems such as NodeJS and python now have
frameworks which essentially handle requests in a single operating system thread.
When we write our handlers as ordinary functions, this means that when one of
our handler functions is working, any request that comes in is placed into a
queue and will wait for the running handler to finish its job before the 
request gets handled. In this way, requests get "serialized".

That alone is not sufficient and describes only "single threaded operation" and
no "coroutining". If we now consider the scenario that the handler function
might actually be twiddling its metaphorical thumbs waiting for a reply from
another service from across the world, it would seem unfair to the request
waiting in the queue that the thread is idle and yet it won't process the
request in the queue. Typically, such "thumb twiddling" happens during I/O
requests, either to subsystems on the same computer such as the file system or
GPU, or another process on the same system, or another process on another
computer.

To address this, Javascript and Python's "async/await" mechanism puts the
"event loop" (the "thumb twiddler" at the core) in control and turns the
handlers into functions that can "return" multiple times. So a handler that
needs to wait for another request to complete would do the following --

1. Create and send the request to the remote party.
2. Ask the event loop to resume the handler once the remote party responds.
3. Transfer control back to the event loop.

Now, the event loop is free to handle the next request in the queue
even though the earlier handler has not finished. To be clear, this
still does not clear our plate of potential problems since the request
the handler sent out could be destructive in some way and if another
request for the same thing comes in (called a "race condition"), we're
still left with the question of what should actually happen.

The concept of "generators" underlies the "async/await" mechanism by providing
the ability to "return" from a function  multiple times while preserving its
execution state, using a keyword such as ``yield``.

In python, coroutines are created using the keywords ``async`` (short for
asynchronous) and paused for results from another asynchronous operation using
the keyword ``await``.

.. figure:: images/coroutining.png
   :align: center
   :alt: Illustrates co-routining between two "async" functions

   The sequence diagram illustrates how two routines (functions) cooperate
   via the event loop to yield time to other operations that need attention
   while within the same thread of control. Such "cooperating routines"
   are why they're called "coroutines", as opposed to a "routine" which
   takes up all the resources of a thread for itself.

Database transactions
---------------------

If we now think of the database as a "service" and our application as its
"client", we can see how the database also needs to contend with potentially
conflicting requests. While one request asks the DB to modify some set of rows
in one way, another might come in and ask to modify an overlapping set of rows
in another way. What is a DB to do in such a scenario?

DB creators do what is best in such a circumstance -- which is to provide
the programmer with mechanisms using which they can dictate what is to happen.
One such mechanism is the "transaction".

A "transaction" refers to a carved out *sequence* of operations, which when
executed as unit, the database will guarantee some properties of the outcome -- 

1. The DB guarantees that either the transaction completes in its entirety,
   or fails entirely and will not leave the database in an intermediate 
   "half done" state. Think about this a bit and you'll see that it can take
   substantial machinery to ensure this property ... called "Atomicity".

2. The DB guarantees that the transaction will complete and leave the database
   in a "consistent" state -- where all the database constraints (such as inter-table
   relationships, index tables, etc.) are all consistent with the contents
   of the database. This property is referred to as "consistency".

3. Furthermore, the DB also guarantees that two concurrent transactions where
   one is writing to the DB and another is reading from it, won't see
   each others' intermediate states. Transactions are therefore said to be
   "isolated".

4. Once the DB declares a transaction to be complete, it guarantees that any
   data stored as a consequence will be retained in storage even if in that instant
   the database were to crash or the computer's power be cut off (or imagine
   any other such violent interruptions). This is called "durability".

Databases which provide transactions with all four of these properties (which
PostgreSQL_ and MySQL_ do as well as SQLite3_) are said to provide "ACID
transactions" where "ACID" is the common acronym for "Atomic, Consistent,
Isolated and Durable" transactions.

.. _PostgreSQL: https://www.postgresql.org/
.. _MySQL: https://www.mysql.com/
.. _sqlite3 documentation:
.. _sqlite3:
.. _sqlite: https://www.sqlite.org/index.html 

Unless you have a good reason not to, it is always good to execute your SQL
queries within a "transaction". With sqlite3, this is marked by the "begin"
and "commit"/"rollback" statements. So your python code will look like -

.. code:: python

    def do_transaction(db, args):
        with closing(db.cursor()) as cur:
            cur.execute("begin")
            try:
                # Run your SQL commands that must be run as a unit.
                # cur.execute("...")
                # cur.execute("...")

                # Finally "commit" your transaction.
                cur.execute("commit")
            catch db.Error:
                # In case some failure occurred during the transaction,
                # we should "rollback" any changes that we happened to do
                # so that the database stays consistent.
                cur.execute("rollback")

With these, we're still running our database queries synchronously and therefore
need to address the question of "what happens when it takes a long time to
respond to a query?". The aiosqlite_ package in python turns the DB calls into
async calls and therefore returns frequently enough to the event loop to not block
other requests to our server.

.. _aiosqlite: https://github.com/omnilib/aiosqlite

