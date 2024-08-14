HTML/XHTML
==========

"Markup languages" based on XML have been the staple of the world wide web
pretty much since its inception. They originated in the need to note attributes
of portions of otherwise normal text for the purpose of a program that would
interpret these attributes for appropriate display. While XML was originally
intended to describe data and documents using "tags", it was adapted for use to
describe hypermedia documents as HTML -- HyperText Markup Language.

.. index::
   single: hypermedia

The term **Hypermedia** refers to the construction, use and transmission of
documents that may refer to other documents via "URL"s (**Uniform Resource
Locator** s). Below is a simple ``HTML`` document that presents a heading
and some body text.

.. code:: 

    <!doctype html>  <!-- Indicates this is a HTML document,
                          and this block is a "comment" that is
                          discarded by the interpreter. -->
    <html>
        <head>
                <!-- contains metadata about the document -->
                <title>Sample document</title>
        </head>
        <body>
            <h1>Padma Bhushan</h1>
            <p id="padma-bhushan-desc">
                That is a title conferred on persons in India who are
                eminent in significant areas of national value such as
                <em>art</em>, <em>sciences</em> and <em>education</em>.
            </p>
        </body>
    </html>

Conceptually, HTML also presents a "recursive dictionary" like JSON.
However, it is more conducive when significant amounts of textual
content needs to be represented along with structure and presentation
style.

A HTML document consists of a ``html`` "tag". A "tag" has the form --

.. code::

    <tagname attr1="value1" attr2="value2" ...>
       ... tag body content ..
       <child_tagname attrc1="val1" attrc2="val2" ...>
       </child_tagname>
       ...
    </tagname>

Tags appear in "open-close" pairs like ``<p>...</p>`` where the tag name
is repeated in both parts to indicate that they're matched. Tags are required
to be nested correctly, so ``<p>...<b>...</p>...</b>`` would be considered
"illegal HTML" (though some programs might give it some meaning).

Tags can have string-valued attributes, textual body content and child tags.
Together they form a tree-like structure. In the case of ``HTML``, this tree is
called the **DOM-Tree** where **DOM** stands for **Document Object Model**.
As with JSON, the DOM is how the document is made available as a value within
a programming language (such as Javascript).

The ``HTML`` language, specifies a collection of known tag names and specific
intended meanings for them. One key element of basic HTML that marks the format
as a candidate for "hypermedia" is the "anchor" tag (tag name ``a``), written
as ``<a href="some-URL">Link text</a>``. The ``href`` attribute of the ``a``
tag is required to specify the URL to navigate the client to when the link text
is selected by some means such as a mouse-click.

.. admonition:: **The HTML Standard**

    The `HTML Standard`_ is worth browsing through to get a feel for all the
    considerations that need to go into specifying a format that is the
    hallmark of the modern Internet. Descriptions of the various `elements of
    HTML`_.
    

.. _HTML Standard: https://html.spec.whatwg.org/multipage/
.. _elements of HTML: https://html.spec.whatwg.org/multipage/#toc-semantics

One of the ways in which HTML differs from JSON is that HTML makes no
particular note of "numbers" as a type. Everything is a string of text and if
you want numbers, you need to store them as string values -- either as body
content or as values of attributes.

The Document Object Model - DOM
-------------------------------

The byte-stream representations of structured data we've seen in JSON and HTML
are designed with a model of the data structures to be used to work with them
in mind when processing the data in programs.

For JSON, the in-program objects we work with involve numbers, strings, array
of values, and a dictionary (a.k.a. "object" in Javascript) that maps string
keys to values. The values themselves may be of any of these types and there is
no constraint of uniformity of type for all members of a collection.

Similar to that, there is an "object model" corresponding to the structure of a
HTML document --- an in-program representation of the document that can be used to
access and manipulate parts of the document programmatically. This is the DOM_.

.. _DOM: https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model

1. Individual tags such as ``<h1>..</h1>``, ``<p>..</p>`` are called "HTML
   elements" and are mapped to the Javascript class named `HTMLElement`, which
   inherits from a more generic ``Element`` which inherits from ``Node``. The
   term of "node" comes from thinking of these elements and their parent-child
   relationships forming a "tree of nodes".

2. An element may have a number of attributes with string values. For example,
   a "paragraph element" with the id ``my-para`` would look like ``<p
   id="my-para">...</p>`` in the HTML file. If the corresponding
   ``HTMLElement`` object were in the JS variable named ``el``, then you can
   retrieve the value of the ``id`` attribute using ``el.getAttribute("id")``
   and you'll get a string value if the attribute were present (See
   getAttribute_). The object then also lets you modify the attribute of the
   element using ``el.setAttribute("id", "new-value")`` (See setAttribute_).

3. You can retrieve the immediate children of a given ``Element`` using
   ``el.children`` which will be an iterable ``HTMLCollection`` object holding
   the child **elements** (not all **nodes**, only **elements**) that you can
   treat conceptually as a list of elements. Note that ``el.tagName`` gives you
   the name of the tag corresponding to that element.

The top level ``head`` and ``body`` elements can be accessed in JS simply
as ``document.head`` and ``document.body``. So with just that much machinery,
it would be possible to step down and walk the tree of elements to reach
whatever element you need to for manipulations.

Doing that is highly repetitive from a programming perspective and therefore
there is a mini language called `CSS selectors`_ that you can use to address
an element within the document by specifying some properties and asking the
``document`` to search for those properties. The two primary methods you
can use to query the document using CSS selectors are ``el.querySelector("..")``
and ``el.querySelectorAll("...")``. The former will give you the first
element that matches the selector specification and the latter will give
you a (conceptual) list of all selectors that match the given specification.


.. _CSS selectors: https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_selectors
.. _getAttribute: https://developer.mozilla.org/en-US/docs/Web/API/Element/getAttribute
.. _setAttribute: https://developer.mozilla.org/en-US/docs/Web/API/Element/setAttribute
