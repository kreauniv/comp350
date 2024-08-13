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
particular note of "numnbers" as a type. Everything is a string of text and if
you want numbers, you need to store them as string values -- either as body
content or as values of attributes.


