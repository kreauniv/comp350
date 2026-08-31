On factoring programs
=====================

Much of program and system design involves identifying how to divide up the
program (or system) into parts that can be understood in isolation and can be
composed to make the whole. We'll call this "factoring" a program/system.
It also goes by the name "decomposition".

You may have encountered the related term "refactoring" more commonly used to
refer to restructuring a program without altering any functionality. There is
no clear definition of "refactoring" practiced in the industry, but we'll use it
to refer to "changing the way the program or system has been factored" towards
some maintenance goal such as improved comprehensibility, testability or
flexibility.

A broad framework
-----------------

When faced with designing code for a task, it is useful to think about and
articulate four aspects -

1. Stuff
2. Structure
3. Properties
4. Behaviour

``pyls`` example
~~~~~~~~~~~~~~~~

For the simple ``pyls`` program you initially worked on, the "primary stuff"
consists of files or folders identified by name, the "structure" consists of a
list of files, and the "properties" include their names, modification times and
whether they are files or folders. We have additional "auxiliary stuff" such as
the "file system" which we can ask to fetch properties of files/folders
identified by path. We also have the terminal to which we can print our
results. You could say that using these to achieve our goal of listing the
contents of a directory constitutes the "behaviour" we need to implement.

It helps to have these find direct representation in your program. For example,
we represented the "stuff and their properties" like this --

.. code:: python

   class FileInfo(NamedTuple):
      filename: str
      modtime: datetime
      isfile: bool
      size: int

The behaviour we sought was to take given options such as directory to scan
and format of presentation to produce a "one item per line" output on the
terminal. The structure in this case is nearly trivial - simply a list
of files - i.e. :code:`list[FileInfo]`.

.. code:: python

   def fetch_file_info(dir: str, filename: str) -> list[FileInfo]:
      # ... Uses the os to fetch file details.

Since the Terminal accepts strings to be printed out, we transformed
each file into a string so we can send it out --

.. code:: python

   class Options(NamedTuple):
      dirname: str
      dir_indicator: bool
      longform: bool

   def get_options() -> Options:
      # Get options from command line

   def present_file(f: FileInfo, dir_indicator: bool, longform: bool) -> str:
      # ... Construct a string.

Once we have all of these implemented, we merely needed to put together
everything to have what we wanted --

.. code:: python

   def main():
      options = get_options()
      filenames = os.listdir(options.dirname)
      fileinfos = [fetch_file_info(options.dirname, filename) for filename in filenames]
      lines = [present_file(f, options.dir_indicator, options.longform) for f in fileinfos]
      for line in lines:
         print(line)


Focusing on data transformation
-------------------------------

Doing I/O in a program is one of the aspects to pay close attention to --
because you do not want to mix I/O with other computation your program
needs to do. So in general, it helps to isolate all I/O in a portion
of your code and keep the rest of your program structured using ordinary
functions that perform data transformation.

The type notation of Haskell is a useful way to work out how to compose
the program. Again, in the ``pyls`` case, we could've expressed our program
decomposition as the following sequence of "factoring" --

.. code:: hs

   main :: IO ()
   -----
   main is composed of
       get_file_infos :: IO (List FileInfo)
       print_file_infos :: List FileInfo -> IO ()
   -----
   get_file_infos is composed of
      get_options :: IO Options
      dir_name :: Options -> String
      dir_contents :: String -> IO (List FileInfo)
   print_file_infos is composed of
      present_file_info :: Options -> FileInfo -> String
      print_line :: String -> IO ()
   ----
   dir_contents is composed of
      list_dir :: String -> IO (List String)
      fetch_file_info :: String -> String -> IO FileInfo

   
For a given function F to be "factored" into functions A and B,
it is important that the result type of A matches an argument type of B,
so that F can be expressed as a composition of A and B as ``B . A``.
In the above case, our main can then be composed from the above parts
perhaps like this --

.. code:: hs
   
   main = do
      opts <- get_options
      items <- list_dir (dir_name opts)
      infos <- sequence [fetch_file_info (dir_name opts) item | item <- items]
      sequence_ [print_line (present_file_info opts info) | info <- infos]

Notice how this is similar to the structure of our python version, suggesting
that the decomposition (a.k.a. factoring) process is reusable across
programming languages.

We'll now look at some factoring techniques.

Dependency injection
--------------------

To generalize a procedure or a function over some particular aspect of its
behaviour, that behaviour can be encapsulated into a function/procedure and
passed as an argument. In more complex cases, this argument can be an object or
even a class (in languages where classes are also objects). Because we're
lifting the dependency on specifics of the object out into the argument and
"injecting" it back in when we're calling our function, this technique is
called "dependency injection".

Traditional OOP languages like Java tend to use this method extensively to simplify
code and to effect separation of concerns. Therefore they also tend to have
easier ways to perform such dependency injection. However the idea is as
simple as we've stated it.

In our ``pyls`` case, for example, if we wished to abstract over the 
"printing out of the results", we could model that behaviour as an argument
that is a function which performs the printing job. That way, we can pass
any other implementation for that behaviour if we so desired, such as
writing the string to a log file instead of the terminal.

.. code:: python

    def flexi_show_file_info(fs: list[FileInfo], output_line = print):
        # ...
        for line in lines:
            output_line(line)
        # ...

    # If we wanted to write to another file, we can now call it
    # as ...

    with open("anotherfile.txt", "w") as f:
        print_to_file = lambda s: print(s, file=f)
        # Inject the dependency into flexi_show_file_info
        flexi_show_file_info(fs, output_line = print_to_file)

Parsing
-------

The broad category of activities comprising the interpretation of user
input into appropriate data structures for processing can be referred
to as "parsing". In the simplest of cases, we have the user giving
us textual input, in which case it would be literally parsing of
some language. In other cases, the user may be performing an activity
that mixes modes like mouse input and key presses which need to be
captured. For example, the process of dragging an object and dropping
it into a target will need to be captured as an operation that focuses
on which object was dragged and where it was dropped, as opposed to
the mechanics of how the user went about it.

In all cases, the end result of the "parsing" step is the creation of
a data structure that represents the operation intended by the user.
Again, in our ``pyls`` example, we parsed the command line arguments
into an ``Options`` structure that looked like this --

.. code:: python

    class Options(NamedTuple):
        dirname: str
        longform: bool
        dir_indicator: bool

In our case, the user could've given this information in a plethora
of ways --

.. code:: bash

    # Argument patterns
    "somedir" --longform --formatted
    "somedir" -l -F
    "somedir" -lF
    "somedir" -Fl
    "somedir" -l
    "somedir" -l --formatted
    "somedir" -F --longform
    "somedir"

And in all those cases, we normalize the input into a single ``Options``
object. By the time the object is created, default choices for unspecified
arguments have been made, order or values provided have been accounted for,
short and long forms have been normalized, and so on. The rest of the program
now only has to work with ``Options`` without worrying about how we got it.

That's the benefit of treating input processing as "parsing" -- by the time the
``Options`` object is created, the rest of the program can assume that any
validation checks on the input that are necessary have been applied and no
further checks are needed. 

Generalization
--------------

In many cases, we may write functions for doing something specific, but then
realize that if one aspect of it can be hidden, the function can be used for
more purposes -- i.e. becomes more generic. This can be achieved by splitting
out that portion into a separate function and injecting it as a dependency.
Though this also looks like dependency injection, we call it out as a 
"generalization" step because the intention is to render the function be
usable in potentially unrelated contexts.

This is kind of how we got the generic functions like ``map``, ``reduce``,
``filter`` and ``foldl``.

**Note**: In Comp308, you'll learn about **beta-abstraction** which formalizes
this notion of generalization.

As with most organization principles, it is possible to make code too general.
It takes judgement to strike a good balance and you'll develop that as you
program more.

Configuration
-------------

To make a module more flexible or usable in different deployment scenarios,
we may choose to load constants from "configuration files". Common configuration
file formats are ini_, json_, toml_, yaml_.

.. _ini: https://en.wikipedia.org/wiki/INI_file
.. _json: https://en.wikipedia.org/wiki/JSON
.. _yaml: https://yaml.org/
.. _toml: https://toml.io/en/

Typical uses of config information are to help the program better deal with
differences in installed setups such as database connections, folder preferences,
log file information, log levels required, API access tokens and so on.

The high configurability requirements of deep-learning libraries gave rise
to the gin-config_ package which automates the task of parsing the config
information and setting appropriate constructor arguments.

.. _gin-config: https://github.com/google/gin-config

As seen with gin-config_, such configurability is done only for entire programs
usually and not for individual modules -- i.e. there is one configuration file
for the entire program. Furthermore, libraries don't usually use configuration
files and will expect all input to be provided via structures or function
arguments. That enables libraries/packages to be used in environments the
authors didn't foresee -- for example, in environments where there may not be
permissions to access specific folders or the file system may be restricted in
other ways.

Contracting at the boundaries
-----------------------------

While functions may have type declarations restricting the types of arguments
they may receive at call points, type declarations alone do not sufficiently
describe the arguments, which are often additionally constrained. For example,
though ``math.sqrt`` in python might be declared with the type ``float -> float``,
the argument is constrained to be ``>= 0`` and ``math.sqrt`` applied to a
negative number will raise an error.

Input contracts refer to such constraints on the arguments of functions
and can be easily expressed in code using "assertions". In python, the
assert_ statement provides this facility. When making assertions, it is
useful to provide an appropriate string that includes additional information
so that the way to fix the failing assertion is easy to identify.

.. _assert: https://docs.python.org/3.9/reference/simple_stmts.html#the-assert-statement

Similar to input contracts, every function will also make claims about its
result, assuming it was called with valid input arguments. These are "output
contracts". For example, ``math.sqrt`` will guarantee that the result, when
squared, will be close to the argument to a good approximation. A function
often does not explicitly state output contracts, except in languages
that support such contracts. It is, however, a valuable practice to document
them either as assertions at return points or in the function's docstring.

Since a module or class is expected to provide some coherent functionality,
they will also place similar contracts at their boundaries. Usually, they will
require that any data structures defined by them be created only using the
functions within the module, so that "invariants" about them can be ensured. An
"invariant" is a property (i.e. a predicate) of an object that does not change
across a particular operation. Some invariants are always expected to hold (for
example, a list may be expected to always be in sorted order) while other
invariants may apply specifically to some operations. In any case, it is
important to document any such invariants as they provide valuable hints to the
programmer using the module's facilities.

In summary, contracts are most usefully specified at "boundaries".

Parse. Don't validate.
----------------------

In some code, you may see functions that check some property of one or more
data structures to "validate" whether they're suitable for use with some other
functions. For example, if a function expects a monetary debit value, a
validation function may check that the value is above zero and only uses
two decimal places of precision. 

The principle asks that instead of writing such validation functions, you wrap
the validated values in appropriate objects so that the proof that they've been
validated is carried along with the values wherever they're used. This helps
all functions that reference the values be assured that they're dealing with
valid values and don't themselves need to redundantly check the same conditions
over and over again.

In the ``pyls`` example, we wrapped the information about a file in a ``FileInfo``
object so that functions which used it can assume that it is about a file and
doesn't need to check that its string argument is indeed a file that's present
in the file system.

For more on this topic, see Alexis King's article `Parse. Don't validate.`_.

.. _Parse. Don't validate.: https://lexi-lambda.github.io/blog/2019/11/05/parse-don-t-validate/

Layering
--------

Designing the dependency relationships between components in such a way that
components are partitioned into sets where each set is though of as a "layer",
with higher layers being dependent only on lower layers is called "layering".
There are other overloaded meanings, but this is the most common "layer
architecture" that is also visually depicted as boxes sitting on boxes.

Usually a layer's job is to translate instructions it receives from layers
above into instructions for layers below. Here we use "instruction" in the
general sense of the word, including programmer designed data structures
specified for communication between layers.

The most famous example of the layer architecture is the `OSI 7-layer network model`_.

.. _OSI 7-layer network model: https://en.wikipedia.org/wiki/OSI_protocols

In graphics systems, it is common to translate drawing instructions called
"scene graphs" into lower level commands to a driver such as OpenGL which
itself translates it yet again into "GPU assembly language" called "shaders"
(ex: vertex shaders, fragment shaders, geometry shaders) and sends to the
graphics cards for the actual rendering.

.. note:: **The primary constraint** for such a layer architecture to work is
   that a layer should only be dependent on the layer or layers immediately
   below it in the hierarchy. It must **not** be dependent on a layer two steps
   below and must **definitely not** be dependent on any layer above it.
   This way, a layer serves as an abstraction facility for the layers above it.

Functions within a module may also fall into a layer structure, though not
necessarily so. If they do, then it might be better to split the layers off
into their own modules. If modules are organized in a layer structure, then
lower layers may be organized as sub-modules with nested names. The main
organizing principle though would be in the contracts between the modules.

Substitution principle
----------------------

In programming languages that have a notion of subtyping, the design of the
types should be in such a way that whenever a function expects an argument
to be of type A, a value of type B that is a subtype of A is also considered
legitimate and the function must work correctly in that case as well.

i.e. a B type value can be used in place of an A type value wherever an A type
value is appropriate.

This is also referred to as Liskov Substitution Principle in honour of Barbara
Liskov who coined it in the context of object-oriented programs. Today,
OOP languages are not the only ones with sub-typing and the principle applies
more broadly to those contexts also.




