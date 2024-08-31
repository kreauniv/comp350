JSON
====

The `JavaScript Object Notation <JSON_>`_ (acronymized as JSON_) has in the
recent years become the data stream format of choice for communication between
servers and systems that use the services provided by the servers such as
Browsers and other servers. JSON_ came out of a need to simplify communication
formats at a time when the complexity of XML based formats such as SOAP ended
up being over-design for not too complex use cases. Getting comfortable
representation data in JSON and reading such representations will be useful
when working with online services, often referred to as "software as a
service".

.. _JSON: https://www.json.org/json-en.html

.. index::
   pair: recursive dictionary; JSON
   single: serialization

The heart of the idea underlying JSON and XML-based formats such as HTML/XHTML
is the notion of the **recursive dictionary**. This structure is common in many
"serialized" data representations -- i.e. data structures represented as a
sequence of bytes on disk or in flight on a network -- of complex objects.

In Haskell type notation, one may describe the JSON_ type of value as follows --

.. code:: haskell

    data JSON = Number | String | List JSON | Dict String JSON

1. Numbers are ordinary decimal numbers or floating point numbers in scientific
   notation.

2. Strings are character sequences enclosed in ``"`` characters.

3. Lists (or arrays) are represented by comma-separated JSON terms surrounded
   by square brackets -- like in ``[1.2, "hello", {"dog":"bowow"}]``.

4. Dictionaries (a.k.a. "objects") are represented as key-value pairs where
   keys can only be strings but values can be any JSON term -- like in
   ``{"dog": "bowow", "cat": "meow", "lengths": [5,4]}``.

Since JSON is a representation of data as a sequence of bytes, do see the JSON_
documentation for the details of how numbers and strings are notated. In
particular, the number notation provides for signed and unsigned integers (such
as ``3`` and ``-22``), decimal numbers with fixed number of places after the
decimal point (such as ``-3.7`` and ``0.3344234``), as well as "floating point"
numbers expressed in scientific notation (such as ``314.159e-2``). Strings are
notated as a sequence of characters surrounded by ``"`` double-quote characters,
where the double-quote characters themselves are not a part of the string.

.. admonition:: **Exercise**

    Go through the JSON_ specification page and observe how the choice to use
    the double quote character brings forth special conditions on how to deal
    with striings that themselves contain double quote and other unicode
    characters. Can you read the "railroad diagrams" there? Express in your
    own railroad diagram 

JSON is a flexible serialization format for some kinds of data. Its structure
is designed to map conveniently to common data structures used in programming
applications. However, it is not always clear which particular way data "ought
to be" represented in JSON. The software/system designer needs to take various
constraints into account in order to choose an appropriate representation for the
data.

JSON representation in various programming languages --

1. In JavaScript, a JSON form is valid syntax for a Javascript value. This
   means you can copy-paste a JSON form into a value position in a JavaScript
   program without change (almost, but the differences are usually very
   subtle). Not all Javascript values can be expressed as JSON though, such as
   functions. To handle the subtleties, Javascript provides the
   ``JSON.parse(str)`` function which parses a string containing a JSON form
   and returns it as a Javascript value. To represent a Javascript object in
   JSON, there is ``JSON.stringify(value)``. (Not all types of Javascript
   values can be represented in JSON.)

2. In Python too, a JSON form is valid syntax for a Python value (again,
   almost). Python's dictionaries are more general that what JSON and
   Javascript permit through -- since they can hold arbitrary comparable Python
   objects as keys and not only strings. To handle the subtleties, the Python
   library ``json`` can be used to convert between strings and Python values
   using ``json.dumps()`` and ``json.loads()``.

3. The JSON form cannot be directly used in Julia since Julia does not have a
   literal syntax for dictionaries, unlike Python and Javascript and they
   need to be constructed by calling its ``Dict`` constructor. Its `JSON.jl`_
   package features ``JSON.json(val)`` and ``JSON.parse(str)`` functions for
   translating values to and fro.

3. Similar to Python and Javascript above, many languages provide "parse"
   and "stringify" functions to convert to/from JSON.

.. _JSON.jl: https://github.com/JuliaIO/JSON.jl

Example: A simple table
-----------------------

Consider a simple table recording temperatures at a place on various days. It
would be easy to think of such a data chunk as a two-column table whose rows
map to each reading taken. If we try to serialize this data on the wire using
JSON though, we're faced with a multitude of choices, and we find ourselves not
really knowing what's "the best one" without having to think of particular
circumstances to perform such an evaluation.

That --- the choices we make taking into consideration various constraints ---
is what we call "design" in this course. Below are some possible choices for
such a table of values.

1. ::

    ["2025-05-01", 23.5, "2025-05-02", 24.1, ...]

2. ::
    
    ["2025-05-01", "23.5", "2025-05-02", "24.1", ...]

3. ::

    [["2025-05-01", 23.5], ["2025-05-02", 24.1], ...]

4. ::

    [{"date": "2024-05-01", "temp": 23.5}, ...]

5. ::

    [1714501800, 23.5, 1714588200, 24.1, ...]

6. ::

    {"date": ["2024-05-01", "2024-05-02", ...],
     "temp": [23.5, 24.1, ...]}

7. ::

    {"date": {"format":"YYYY-MM-DD",
              "values": ["2024-05-01", "2024-05-02", ...]},
     "temp": {"units": "C",
              "values": [23.5, 24.1, ...]}}


None of these can really be declared as "wrong", but each might be a suitable
representation to use depending on the constraints under which you need a
representation for such data.

.. admonition:: **Exercise**

    For each of the representations listed above, come up with one situation in
    which that representation might be considered to be more suitable than some
    (or all) of the others. Think about what some of the
    advantages/disadvantages are of each of the above representations.
    Consider, for instance, ease of programming, ease of communicating
    intention with other team members/customers and ease of documentation.

This choice partly arises from the flexibility offered by the data structure
we called a "recursive dictionary". This structure is so flexible that the PDF
file format is essentially one big recursive dictionary.

Some sample JSON structures in real applications
------------------------------------------------

The Vega_ visualization library represents its interactive visualizations
using a language representable as a JSON object. For example, here is a
stacked chart in Vega_ - https://vega.github.io/vega/examples/stacked-area-chart/

.. _Vega: https://vega.github.io/vega/

Observe various JSON representations used by Facebook in its `Graph API`_ to
access aspects of data they store on your behalf --
https://developers.facebook.com/docs/graph-api/overview/

.. _Graph API: https://developers.facebook.com/docs/graph-api/overview/

Below is a chat history represented as JSON and passed to OpenAI_ to get GPT to
produce the next response appropriate in the thread ---

.. _OpenAI: https://platform.openai.com/docs/guides/text-generation

.. code:: json

  {
    "model": "gpt-4o-mini",
    "messages": [
      {
        "role": "system",
        "content": "You are a helpful assistant."
      },
      {
        "role": "user",
        "content": "Who won the world series in 2020?"
      },
      {
        "role": "assistant",
        "content": "The Los Angeles Dodgers won the World Series in 2020."
      },
      {
        "role": "user",
        "content": "Where was it played?"
      }
    ]
  }

`GitHub.com`_ exposes some of its functionality via "API"s which you can talk
to by sending JSON forms and receiving replies in JSON forms. For example, you
can programmatically retrieve github "issues" as shown here -
https://docs.github.com/en/rest/issues/issues?apiVersion=2022-11-28#list-issues-assigned-to-the-authenticated-user

.. _GitHub.com: https://github.com


