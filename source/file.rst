The "file" abstraction
======================

You're all familiar with the notion of a file stored on your computer's "disk".
From the operating system's perspective, it doesn't care about the contents
of your files and only provides a handful of operations to deal with its contents.
These are -

1. open_ a file for operations. This is presented to the programmer as an
   integer value that represents the open file "handle" and this handle needs
   to be made available for the other operations.

2. read_ one or more bytes from the "current position" (if the file is
   readable). This will advance the current position by those many bytes.
   You cannot read beyond the end of the file.

3. write_ one or more bytes at the "current position" (if the file is
   writeable). This will also advance the current position by those many bytes.

4. `seek <lseek_>`_ to a specific position within the file (if the file is seekable).

5. close_ the file.

.. _open: https://www.man7.org/linux/man-pages/man2/open.2.html
.. _read: https://www.man7.org/linux/man-pages/man2/read.2.html
.. _write: https://www.man7.org/linux/man-pages/man2/write.2.html
.. _lseek: https://www.man7.org/linux/man-pages/man2/lseek.2.html
.. _close: https://www.man7.org/linux/man-pages/man2/close.2.html

Notice that these are very generic in nature. Therefore the operating system
exposes many system facilities as such "streams of bytes" that you can
open/read/write/seek/close. [#seek]_ Some examples --

1. Want to write to your program's "stdout"? Just ``write`` bytes to handle
   number 1. To write to stderr, ``write`` to handle number 2. It's that
   simple. Want to read from "stdin", just ``read`` from handle number 0.

2. A connection to another computer (server or client) similarly provides a
   "socket_ handle" (also an integer just like a file handle) and transmitting
   or receiving bytes to/from another computer is simply a write/read on that
   handle. 

3. Older versions of Linux used to provide a special "file" named `/dev/dsp
   <dsp_>`_ which if you open and read, you'll be recording audio samples from
   your computer's microphone and which if you write to, you will be sending
   audio samples to your computer's speaker -- i.e. playing a sound.

4. Want some random bytes to use in your program? Open the "file" `/dev/random
   <random_>`_ and read how many ever bytes you want. You can't write to it or
   "seek" to a different position though.

5. Want a list of processes running on your linux machine? You can list the
   "directory" `/proc <proc_>`_ and find one directory for each process which
   further contains more info you can read using the "file" interface.

.. _socket: https://www.man7.org/linux/man-pages/man2/socket.2.html
.. _random: https://en.wikipedia.org/wiki//dev/random
.. _dsp: https://manpages.ubuntu.com/manpages/questing/man7/dsp.7.html
.. _proc: https://www.man7.org/linux/man-pages/man5/proc.5.html

This is a core example of how one abstract "interface" can have multiple
"implementations". This is such a core idea in operating systems that a
whole operating system called "Plan 9" was once built with **all** of its
facilities accessible as "files".

Having such a common interface makes it easy to use all these diverse
facilities as no new vocabulary needs to be learnt for programming them.
Of course, you need to know the specific concepts about those various
aspects to use them, but the mechanics of "how" is the same.

Interface - implementation split
--------------------------------

We call the set of functions open/read/write/seek/close the "interface" of the
"file" abstraction and each of the above examples provide "implementations" of
that interface.

**Small** interfaces that hide **a lot** of functionality are very beneficial
for programming because they simplify the mental model a programmer needs to
use and the number of words/functions they need to remember.

For a function, its "interface" consists of the specific nature of its
arguments and the return value(s) it provides. This is also referred to 
as its "type signature" or just "signature" for short.

Example - logging
~~~~~~~~~~~~~~~~~

Supposing we have a procedure that does many operations but wishes to log
intermediate steps to the terminal stdout. You might write it like this --

.. code:: python

    def my_big_procedure(x: int, name: str, values: list[int]) -> str:
        print("I'm going to do step 1")
        do_step_1(name, x)
        print("Now on to step 2")
        do_step_2(values, x)
        print("All done")
        return "done"

It is not much of a procedure, but I hope you get the idea why someone would
write something with this pattern -- this is usually called a "script".

We now notice that for the purpose of this procedure, ``print`` behaves
as though it had the signature ``print(arg: str) -> None``. So if we wanted
the procedure to be flexible enough to send those messages to any destination,
(ex: to a file, to a web browser client via a websocket, etc.), we can simply take
in the function to call as an argument like this --

.. code:: python

     def my_big_procedure(x: int, name: str, values: list[int], log: callable[[str],None] = print) -> str:
        log("I'm going to do step 1")
        do_step_1(name, x)
        log("Now on to step 2")
        do_step_2(values, x)
        log("All done")
        return "done"

Now ordinarily, you'll call this new procedure in the same way, though it
takes an extra argument, because we've given it a default. Supposing we want
to collect all the messages into a list of strings, we can do this --

.. code:: python

    messages = []
    def collect(msg: str) -> None:
        messages.append(msg)

    my_big_procedure(2, "something", [1,2,3], log=collect)
    # Now `messages` contains a list of the messages!

What we've done here is to generalize the "logging" behaviour of ``my_big_procedure``
to be customizable using a parameter. Anything that meets the "signature" of a
logging procedure can be supplied. Supposing we wish to output to ``stderr``,

.. code:: python

    def stderrlog(msg: str) -> None:
        print(msg, file=sys.stderr)

    my_big_procedure(2, "something", [1,2,3], log=stderrlog)

Objects and classes
-------------------

The "class" facility in Python is basically to provide such interfaces which
are more than just one function. Any class that "implements" the same interface
-- i.e. has the same set of methods (identified by name) all of which
have the same argument pattern and return types (i.e. the same signatures)
can be substituted for each other. We saw how ``StringIO`` implements the
same methods as a ``file`` and therefore can be used in its place.

.. admonition:: **Liskov substitution principle**

    Because this substitutability principle can be expressed as "if it looks
    like a duck and quacks like a duck, it is a duck", this is also sometimes
    referred to as "duck typing". The early computer scientist Barbara Liskov
    was the first to articulate it and so a more precise definition of it is
    attributed to her as the `Liskov substitution principle <liskov_>`_.

.. _liskov: https://en.wikipedia.org/wiki/Liskov_substitution_principle

.. [#seek] Of these, **seek** is perhaps the most specific one that usually
   applies only to files.

