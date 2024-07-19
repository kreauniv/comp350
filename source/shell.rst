The "shell" a.k.a. the "command line"
=====================================

The **shell** has been given that name in line with the notion of the operating
system **kernel** which serves as its core. The shell then offers a means of
interacting with the kernel to use its services, such as managing the **file
system**.

For this course, I urge you to become familiar with doing much, if not all, of
your activities via the shell on your laptop. If you're using MacOS, the
"Terminal" application offers such a shell. If you're using Windows, please
install "WSL" ("Windows Subsystem for Linux" with Ubuntu as the OS choice) and
use that in place of the traditional Windows "command line".

This document offers a "cheat sheet" that you can refer to for common
operations you need to perform. I'll assume that you're familiar with basic
computer organization terms such as "file" and "folder" (a.k.a. "directory").

Basics and Guidelines
---------------------

There are 3 main concepts you need to be familiar with to use the shell
effectively.

Running a command
~~~~~~~~~~~~~~~~~

A "command" is (typically) a verb instruction given to the shell that identifies
an operation to perform and takes arguments that control that operation. A command
has the simple form of verb and arguments separated by one or more spaces. The number
of space characters separating arguments does not matter. Here are some such commands.

* ``ls -l mydir`` - lists the contents of the ``mydir`` directory in "long form".
* ``cd mydir`` - changes the present working directory to ``mydir``.
* ``cat textfile.txt`` - dumps ``textfile.txt`` to the terminal.

All shell commands send their output to a stream called "standard output" (or
"stdout" for short) and perhaps another (depending on the case) called
"standard error" (or "stderr" for short). The shell programs also take input
from a "standard input" (or "stdin" for short). So the general structure of a
command invocation is --

.. code:: 

   verb-or-command arg1 arg2 ... argN

Most shells (such as ``bash`` on Ubuntu and ``zsh`` on MacOS), will interpret
some characters in the arguments as special. For example, ``ls prefix*`` will
list all the files that start with ``prefix``, taking the ``*`` to match
anything that follows the prefix string. So what do you do if your file name
itself has one of these characters? In such cases, you can strict quote the
file name using single quotes, like this -- ``ls 'prefix*'``. Similar to ``*``,
the ``?`` character will match any single character, so ``ls '*.??'`` will list
all files that have two character "extensions" (the portion after the ".").

.. admonition:: **Note**: The special characters ``*`` and ``?`` are called
   "wild card" characters because, as in the UNO game, they can be used in
   place of arbitrary text.

.. admonition:: **Another note**: If you have two files named
   ``assignment-first.txt`` and ``assignment-second.txt`` in the current
   directory and you've typed ``ls assignment-*`` expecting to see both of
   them, know that doing so is exactly equivalent to having typed ``ls
   assignment-first.txt assignment-second.txt`` by hand! i.e. The wild card
   characters are expanded first and passed as arguments to the ``ls`` program
   *before* running the program.

Redirecting the output of a command
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

We saw that commands send their output to their *stdout*. If we want to store
whatever is sent to the command's *stdout* to a file for later use, we can use
the "redirection" operator ``>`` like this -- ``command arg1 arg2 ... argN >
filename.ext``. Note that the file name and extension are completely arbitrary
from the system's perspective. Different programs may interpret extensions
according to their own conventions. It is common for text files to have the
".txt" extension and programming language source code files are also identified
by their extension by editors.

For example, ``ls -l > /tmp/listing.txt`` will list the files in the current
directory in long form and store ("redirect") that output to the file at the
absolute path ``/tmp/listing.txt``. The ``/tmp`` folder, by convention, is
commonly used for temporary files and this directory is emptied every time you
reboot your machine.

.. admonition:: **Note**: The ``>`` redirection operator will only redirect
   *stdout* and not *stderr*.

Connecting programs via pipes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Since shell programs take their input from a *stdin* and output their results
to an *stdout*, we can now think about combining programs by connecting the
*stdout* of one command/program to the *stdin* of another command/program to
further process the output of the first program.

This is called "piping" and is done using the ``|`` operator. 

Shell programs in unix operating systems are designed to do one focused task
each that work usually by taking input from *stdin* and sending output to
*stdout*. So accomplishing more complex tasks that involve more than one such
operation is usually done via piping, or if piping is not feasible for some
reason, via temporary files on the file system.

Typically, programs work with lines of text one line at a time as well. This
means if you organize the output of a program as lines of text, you can 
bring many tools to bear on that representation right within the shell.

The piping syntax is simple --

.. code:: bash

    <command1> <argA1> <argA2> .. <argAM> | <command2> <argB1> <argB2> ... <argBN>

For example, ``ls -l | grep '[.]js$'`` will take the long form listing of the
current directory and pass that list to ``grep 'js$'`` which will read each
line in its input and only send to its *stdout* only those lines that have '.js'
as the last two characters on the line.

Notice how ``ls`` itself doesn't need to do any such filtering and ``grep``
doesn't need to know anything about line-by-line listing of filenames and yet
we can put these together to "list only the javascript source files in the
current directory", for that is the meaning we attach to files with names that
have ".js" as their extensions.

Obviously, you can repeat such a pipe operation like -- ``cmd1 | cmd2 | cmd3 ...``
and the combination will do the obvious thing.

.. admonition:: **Note**: If the commands are all producing their outputs one
   line at a time and consuming one line of input at a time, then if any of the
   commands deep down the "pipeline" fails at a point, the whole pipeline will
   fail without forcing the first command to run to completion. i.e. The whole
   pipe will fail early, and this is an incredibly useful property. This also
   means that ``cmd2``, ``cmd3`` etc. will start their processing even before
   ``cmd1`` has completed generating all its output. This magic is orchestrated
   by the kernel facility called "process".

Useful conventions of shell programs
------------------------------------

You'd have seen argument of the form ``-l`` that kind of stand out from other
"normal" arguments like file names. These are called "flags" or "switches" and
give special instruction to the command. In the case of ``ls``, adding a ``-l``
flag tells it to output lots of details about the files in the specified
directory (or the current directory).

Some commands also have long descriptive flags that make such commands more
readable and easier to debug. One such common conventional flag is ``--help``.
These long flags always start with two hyphen characters and may take an
additional associated value (depending on the situation) like ``--flag=value``
or equivalently ``--flag value``.

``--help``
~~~~~~~~~~

If you want some quick details of what a command does, you can often invoke it
with the ``--help`` flag to get a short help. Sometimes, the short flag ``-h``
also works but ``--help`` is more common.

``man``
~~~~~~~

To get detailed help about a program's function, you can consult its "manual
pages" using the ``man`` command. For example, ``man ls`` will bring up the
manual page for the ``ls`` command giving details of all its parameters and
what they are for.

.. admonition:: **Note**: You are NOT expected to know by heart what the flags
   of various commands are and what they do. You can always look them up using
   ``man`` or ``<cmd> --help`` when you need to. **In fact, you should expect
   to do a LOT more reading of manuals than writing of code!!**, at least in the
   early stages of mastery.

Google
~~~~~~

Of course, all such help is available via Google, so you can always google for
help. In fact, all the man pages are also available on the internet and if you
type "man <something>" in google, you'll be taken to the appropriate linux man
pages (usually, if you're lucky, these days).

The caveat is that there are many predatory sites that have "SEO"d their way to
the top of the google search list for such programmer help and which more often
are interested in engaging you on their site for advertisement revenue than
actually helping you with what you want. For this reason, once you find out about
a command using Google, I recommend you stick to referring to its ``man`` page
for the details.

Stackoverflow
~~~~~~~~~~~~~

Once you've gained some fluency with the command line or some basics of a
programming language and find yourself occasionally stuck on some task, you can
also ask `stackoverflow.com <https://stackoverflow.com>`_. You may find this useful
only from about a month into this course.

Mozilla Developer Network (MDN)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

For **all** details on **HTML**, **CSS** and **Javascript** functions and
objects related details, use the "Mozilla Developer Network" or "MDN" for
short. These are excellent, descriptive and non predatory pages that give you
precise definitions with examples to help understand how to use a particular
feature. In other words, "MDN" is your ``man`` for those three areas.

To search MDN specifically, you can always add the word "mdn" to any search you
do in google that pertains to those topics and an MDN page will usually be the
first link you get. For example, to learn how to specify argument of
``document.querySelector`` Javascript function, you can google for
``querySelector mdn``.

MDN is also a "wiki", meaning you can contribute to it. So if you notice any
errors, you can create an account and submit fixes to them.

Cheat sheet
-----------

Here, if I give a word in ``ALLCAPS``, it is a placeholder for an argument
you have to supply. Paths (file and directory names) can either be relative
to the current directory or refer to an absolute location on the file system.
Absolute paths start with ``/``.

Working with folders (a.k.a. directories) and files
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

``mkdir DIRNAME`` or ``md DIRNAME`` -- "make directory" creates a new empty
directory with the given name within the current directory.

``cd DIRNAME`` -- "change directory" changes the "present working directory"
(PWD) to the given directory. To change the PWD to the parent, use ``cd ..``.

``pwd`` -- Outputs the full (absolute) path of the "present working directory".

``rm FILENAME`` -- "remove" the specified file. Note that there is no "undo"
for this destructive operation.

``ls OPT_PATH`` or ``ls -l OPT_PATH`` -- "list files" in the given directory,
either just the file names or some elaborate details if ``-l`` flag is given.

``cat FILENAME1 FILENAME2 ...`` -- Reads the given files in the given order and
dumps ("conCATenates") their contents to the *stdout*. So if you want to join
two files into a new file, you can use redirection like this -- ``cat FILE1
FILE2 > OUTFILE``.

Processing line-by-line formatted data
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

``sort`` -- sorts the contents sent to its *stdin* line by line in
lexicographical order. There are other controls for the sort operation, which
you can learn using ``man sort``.

``cut -f N`` -- can extract the ``N``-th "field" from each line of input,
discarding everything else. Use ``man cut`` to learn how fields are determined
and how to select different delimiters for the fields.

``wc`` -- short for "word count", it counts the number of characters, words and
lines in its *stdin* and prints out three numbers to its *stdout*. By passing
appropriate flags, you can select which of these numbers you want. Learn about
these flags using, you guessed it, ``man wc``. For example, to just know the number
of files in the current directory, you can do ``ls | wc -l``.

``head FILENAME`` -- shows only the first few lines of the file, or if the file
name is not given, the first few lines that get sent to its *stdin*. See ``man head``
to learn how to control the number of lines shown.

``tail FILENAME`` -- shows the last few lines of the file, or if the file name
is not given, the last few lines of the content sent to its *stdin* before it
is closed. See ``man tail`` to see how to control how many lines you want to
see. ``tail`` is a useful program to track the progress of long running
computations where log data is sent to a log file. You can do ``tail -f
FILENAME`` ("-f" for "follow") to ask ``tail`` to stay alive and update as new
lines get written to the file by the process.

``grep 'REGEXP_PATTERN' FILENAME`` -- searches the lines in the given
``FILENAME`` or if the file name is omitted, searches its *stdin* for the given
pattern. If a line contains the pattern, the line is output in its entirety,
otherwise the line is omitted from the output that ``grep`` sends to its
*stdout*. ``grep`` is an odd name for a tool that looks for patterns. It stands
for "GNU regular expression parser". "Regular expressions" constitute a popular
sub-language to express simple (and even some complex) textual patterns. To
learn how to construct and use regular expressions, see ``man grep``, where
you'll also learn about other options to control ``grep``'s output. For
searching through source code, you can use a drop-in replacement program that's
much more convenient for that purpose called ``ripgrep`` (abbreviated ``rg``).

Utilities
~~~~~~~~~

``echo <arg1> <arg2> ...`` -- prints out the given arguments on the same line,
separated by spaces. This is useful if you want to show the value of a "shell
variable" like ``$PATH``. You can see the current set of search paths using
``echo $PATH``.

``less FILENAME`` -- ``less`` is a "pager" program that lets you scroll through
the contents of a large text file using the cursor keys, and also lets you
search for specific patterns in the file. If the file name is not specified, it
works on the contents sent to its *stdin*. So ``less FILENAME`` is quite
equivalent to ``cat FILENAME | less``. This program is **so extremely** useful
that many tools such as ``man`` and ``git`` automatically send their output to
it. Learn how to jump through the file being paged using ``man less``. One of
the useful things you can do when in the ``less`` pager is to **search** for
text by first typing the ``/`` character followed by what you want to find. You
can also jump to specific lines by first typing the number followed by the
character ``G`` (for "go to").

.. admonition:: *History note* - why is this program called "less" you ask? The
   original unix pager program was called "more", which would show a page of
   content and wait for you to hit the space bar to show the next page (hence
   "pager"). When GNU/Linux was written, they couldn't use the same program
   name for copyright reasons, so they named it "less" because "less is more"
   :P Unix history is full of such delightful/groanful word play.

``curl URL`` -- Downloads the given URL and sends the result to its *stdout*.
``curl`` is a very powerful program with very many options to control the kind
of request sent, kind of data and headers to send, etc. It's all documented in
``man curl``.

``touch FILENAME`` -- does not modify the file at all, except for changing its
"last update time stamp" to the current time. If the file with that name doesn't
exist, it creates an empty file with that name.

``vi`` or ``vim`` -- is an interactive text editor that can be found
pre-installed on all Linux systems. So it is useful to be familiar with some
basics. Firstly, always use ``vim`` instead of ``vi`` 'cos its "improved vi".
This editor has two "modes" -- the "insert mode" where you type into the text
file, and the "command mode" where you can issue edit commands. At launch,
you'll be put in the "command mode". You can enter the "insert mode" by
pressing the "i" key. Then type away like in a normal editor. When you're ready
to save and quit, switch to the command mode by pressing the ESC key. Then to
write the text to a file, type ``:w FILENAME<enter>``. To quit, you type
``:q``. This much should help you not be puzzled when you're on the occasion
thrown into the ``vim`` editor. These commands are the same for both ``vi`` and
``vim``.

(MacOS only) ``pbcopy`` -- takes all the input given to its *stdin* and copies
it to the clipboard so you can paste it anywhere you want using Cmd-P. So, for
example, you can copy a list of files in the current directory using ``ls | pbcopy``.

(MacOS only) ``pbpaste`` -- copies the contents of the clipboard to its
*stdout* so you can further process it by piping it to other programs.

