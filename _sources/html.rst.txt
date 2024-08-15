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
no constraint of uniformity of type for all members of a collection. The lure
of JSON for many data exchange tasks that require flexible representation is
that it maps more or less directly to data structures available in many common
programming languages. All a language's library needs to support is strings,
numbers, arrays and dictionaries (hashtables/maps).

Similar to that, there is an "object model" corresponding to the structure of a
HTML document --- an in-program representation of the document that can be used
to access and manipulate parts of the document programmatically. This is the
DOM_. Such documents are not direct maps to existing data structures in
languages, but have some extra semantics to them such as "tag names",
"attributes", "ids", "classes" and such due to which languages provide
libraries to work with such documents. In particular, because Javascript was
designed to manipulate HTML documents, this facility is readily available for
use in-browser via the ``document`` object.

.. _DOM: https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model

1. Individual tags such as ``<h1>..</h1>``, ``<p>..</p>`` are called "HTML
   elements" and are mapped to the Javascript class named `HTMLElement`, which
   inherits from a more generic ``Element`` which inherits from ``Node``. The
   term of "node" comes from thinking of these elements and their parent-child
   relationships forming a "tree of nodes". All ``HTMLElement`` objects are
   also of type ``Node`` but not vice versa.

   **Note**: A "node" that isn't a "html element" is the "text node" which
   holds essentially a string of characters and can have no "child nodes" and
   is hence a "leaf node" in the "DOM tree". Also, ``HTMLElement`` type nodes
   are a special set of tags specified with particular semantics in the HTML
   specification. Most modern browsers though generalize this idea to also
   include any tag names you may define yourself within a HTML document.

2. An element may have a number of attributes with string values. For example,
   a "paragraph element" with the id ``my-para`` would look like ``<p
   id="my-para">...</p>`` in the HTML file. If the corresponding
   ``HTMLElement`` object were in the JS variable named ``el``, then you can
   retrieve the value of the ``id`` attribute using ``el.getAttribute("id")``
   and you'll get a string value if the attribute were present (See
   getAttribute_). The object then also lets you modify the attribute of the
   element using ``el.setAttribute("id", "new-value")`` (See setAttribute_).
   If you merely want to check whether an element has an attribute with
   a particular name, you can use ``el.hasAttribute("<attrname>")`` which
   evaluates to a boolean indicating the presence of such an attribute.
   **Note:** Element attribute names are expected to be unique. Multiple
   occurrences of the same attribute name in a HTML element's text should
   be considered an error irrespective of how browsers treat them.

3. You can retrieve the immediate children of a given ``Element`` using
   ``el.children`` which will be an iterable ``HTMLCollection`` object holding
   the child **elements** (not all **nodes**, only **elements**) that you can
   treat conceptually as a list of elements. Note that ``el.tagName`` gives you
   the name of the tag corresponding to that element.

CSS selectors
-------------

The top level ``head`` and ``body`` elements can be accessed in JS simply
as ``document.head`` and ``document.body``. So with just that much machinery,
it would be possible to step down and walk the tree of elements to reach
whatever element you need to for manipulations. For example, here is such a
"walker" function that visits all nodes in the sub-tree under ``document.body``
and calls a given function on the element.

.. code:: js

    function walk_domtree(root, fn) {
        fn(root);
        for (let n in root.children) {
            walk_domtree(n, fn);
        }
    }

You can use the above function, for example, to remove the "class" attribute
of all the nodes in the DOM (just for fun, not that you want to do it).

.. code:: js

    walk_domtree(document.body, (n) => n.removeAttribute("class"))

Doing that is highly repetitive from a programming perspective and therefore
there is a mini language called `CSS selectors`_ that you can use to address
an element within the document by specifying some properties and asking the
``document`` to search for those properties. The two primary methods you
can use to query the document using CSS selectors are ``el.querySelector("..")``
and ``el.querySelectorAll("...")``. The former will give you the first
element that matches the selector specification and the latter will give
you a (conceptual) list of all selectors that match the given specification.

.. admonition:: **Exercise**

    Open a web page like, say, https://kreauniv.github.io/comp350 ), right
    click on some element that catches your attention and choose "inspect".
    This will take you to the dev console and highlight the specific node in
    the DOM tree corresponding to the element you selected. (See how the
    browser maintains a map between the visual location of a node and it's
    position in the tree for it to be able to do this?) Now right click on the
    element and choose "Set as global variable". This will make a new global
    variable like ``temp1`` and set it to refer to that particular
    ``HTMLElement``. You can inspect its properties and methods by typing
    ``temp1.`` and scrolling through the menu of candidate properties and
    methods that show up. Use the functions discussed above on this element and
    see what happens in the browser's display. Note down your observations.

.. _css: https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_selectors
.. _getAttribute: https://developer.mozilla.org/en-US/docs/Web/API/Element/getAttribute
.. _setAttribute: https://developer.mozilla.org/en-US/docs/Web/API/Element/setAttribute
.. _removeAttribute: https://developer.mozilla.org/en-US/docs/Web/API/Element/removeAttribute
.. _hasAttribute: https://developer.mozilla.org/en-US/docs/Web/API/Element/hasAttribute

As mentioned above, when you're programming an interactive web page by
manipulating the DOM tree, you do not have the luxury of asking the user to
tell you which node they want manipulated, because .... well, they have no
idea.

So you need a way to be able to specify using a declaration like "an element
which ...". For example, "an element which has 'DIV' as its tag name", "an
element whose ``id`` attribute is ``header``" and so on. `CSS selectors`_
provide a mini language to describe elements in this manner. While the language
has many more aspects, the following are sufficient for you to know and handle
more than 80% of use cases in most web pages (easily more).

1. "an element whose tagname is ``tag``" is specified as ``tag``. So
   ``document.querySelectorAll("tag")`` will return a list (technically an
   "iterable") of elements in the current HTML document which have the tag name
   "tag". (Substitute "tag" with "div", "p", "section", "h1", etc.)

2. "an element whose id is ``elemid``" is specified as ``#elemid``. So
   ``document.querySelector("#elemid")`` will return the element whose id is
   ``elemid``. While the specification places no constraint that element ids
   must be unique, in which case you can retrieve all of them using
   ``querySelectorAll``, it is a good idea to obey that constraint to kep
   programming simple. Note that this places no constraint on the tag name.
   To combine this with a specific tag name, use ``tagname#elemid``.

3. "an element with a ``class`` attribute whose value contains the word
   ``some_css_class``" is specified as ``.some_css_class``. Again, like the
   case with ``#elemid``, this will match against any tag that has such an
   attribute (like ``<div class="footer floatright">...</div>`` will match
   ``.footer`` and ``.floatright``). To constrain that to a specific tag name,
   you can use ``tagname.some_css_class``. So the ``div`` just mentioned can be
   selected using ``document.querySelector("div.footer")``.

4. "a ``span`` element that is a descendant of a ``div`` element" is expressed
   as ``div span``. So ``document.querySelectorAll("div span")`` will let you
   walk through all the ``span`` element anywhere in the tree of any ``div``
   element in the document. To restrict this to "a ``span`` that's an immediate
   child of a ``div`` element", you write ``div > span``.

Note that you can combine the above in the following ways ---

1. ``div#elemid``
2. ``div.some_css_class``
3. ``div span#elemid``
4. ``div span.some_css_class``
5. ``.some_css_class descendant_tag``
6. ``#elemid descendant_tag``
7. ``#elemid > child_tag``
8. ``#elemid > child_tag.some_css_class``
9. ``#elemid.classA > child_tag.classB``
10. ... and so on.

So with just that much, the mini language of `CSS selectors`_ already offers
considerable expressive power to select and work with elements in the DOM tree.

Next we'll talk about what these "CSS classes" are all about.

CSS classes
-----------

While the HTML tag "language" is about describing document **structure**, the
CSS language is about describing its **appearance**. The history of this
language is very interesting in that, much like a lot of computing, the
earliest implementation of this idea used a LiSP dialect, but I won't go into
that any further and you can you read up via google searches if you're
interested in that.

Some HTML elements already have some appearance characteristics associated with
them. For example, text enclosed by ``<b>...</b>`` will appear in bold face,
text enclosed in ``<em>...</em>`` will be italicized (i.e. "emphasized") and
such. But in modern HTML+CSS, you can override all of that if you want to.

The way CSS is used to specify appearance of entities in the documents is through
clauses of the following form --

.. code::

    <css selector> {
        <appearance property name1>: <value2>;
        <appearance property name2>: <value2>;
        ...
    }

For example, here is a CSS specification that says "all elements with the
class "footer" must appear in bold face."

.. code::
    
    .footer {
        font-weight: bold
    }

The advantage of specifying the above using CSS is that the browser will always
maintain this condition even under changes to the DOM tree. For example, if you
add a new element to the tree using ``document.createElement``, and then set
its ``class`` attribute to ``footer`` using ``el.setAttribute("class",
"footer")``, the browser will automatically start displaying its contents in
bold because the CSS declaration says so.

You can store such "CSS rules" in a separate file named "something.css" and
apply it to a HTML document by including a ``link`` tag in the ``<head>...</head>``
part like so --

.. code::

    <head>
        ....
        <link rel="stylesheet" src="mystyles.css">
        ....
    </head>

.. note:: Some HTML tags like ``<link>`` do not require matching closing tags
   and are considered to be "self closing". Most tags aren't self closing
   though and when in doubt, check the `HTML standard`_ to clarify.

Loading Javascript code
-----------------------

So far we played with Javascript right in the console. However when you're
constructing interactive pages, this is not how you introduce Javascript code
that manipulates the DOM. You include your code within ``<script>...</script>``
tags in one of two ways --

.. code::

   <script>
       ... your JS code ...
   </script>

... or as

.. code::

   <script src="URL-to-myjscode.js"></script>

The path given for the ``src`` attribute is a URL in its own right and not
necessarily just a file name. All that is required is that when the browser
tries to fetch the URL, it receives some Javascript code to load. Otherwise it
will reject it.

URLs can take one of two forms -- "relative" or "absolute".

**Relative URLs** look like a file path without a leading "/" character. For
example, ``myfile.js``, ``js/myfile.js`` and such are relative URLs. What are
they "relative" to? They are relative to the URL of the document they appear in.
So if the HTML document was loaded from, say, ``https://somewhere.com/here/there/anywhere/file.html``,
and it features a tag like ``<script src="dir/myfile.js"></script>`` inside it,
then the full URL (a.k.a. "absolute URL") of that file is taken to be
``https://somewhere.com/here/there/anywhere/dir/myfile.js``. So the last
"file name" part of the URL is removed and replaced with the given "relative URL".

**Absolute Paths** specify URLs relative to the current "origin" of the document
it is used in. These start with a leading "/" character. For example, if
``<script src="/js/myfile.js"></script>`` is used inside the same HTML file
in the previous case, it would refer instead to ``https://somewhere.com/js/myfile.js``.

**Absolute URLs** are specified in full like ``https://.....`` including the
protocol name, the domain and any intervening paths leading to the "end point".

There is also a category of "protocol relative URLs" that start with a
``//somewhere.com/...``, which are the auto-prefixed with the protocol of the
document. If the protocol happened to be ``http``, then it is taken to be
``http://somewhere.com/...`` and if it happened to be ``https``, it is taken to
be ``https://somewhere.com/...``.

The "event loop"
----------------

When you're reading a webpage, the browser is pretty much not doing anything.
It is sitting "idle" like a car in neutral gear. This "idle state" is the
"event loop" that in principle looks something like this --

.. code::

    // This is pseudo Javascript code to illustrate the idea
    while (waitForEvent()) {
        let event = getNextEvent();
        processEventHandlers(event);
    }

When you do something on the web-page, including scrolling, mousing over
elements, clicking links or text or buttons, typing within input fields, etc.,
the browser generates "events" represented internally as ``Event`` objects. You
program interactivity in a web page by attaching "event handlers" to elements
that handle specific types of events. Javascript "event handlers" are plain JS
functions that take one argument which will be the ``Event`` object
representing the event that needs to be handled. Below is a silly event handler
that removes the "class" attribute of the tag that it is attached to when
the handler is run. See  :ref:`js-crash-course`.

.. code:: js

    function remove_class_attribute(event) {
        event.target.removeAttribute("class");
    }

You then attach such an event handler to the "click" event of an element like this -

.. code:: js

    let elem = document.querySelector("div.footer");
    elem.addEventListener("click", remove_class_attribute);

Now, when the user clicks the element, your function will run, which will result
in the element's class attribute being removed. Since functions are values, you
can combine the above two like this ---

.. code:: js

    let elem = document.querySelector("div.footer");
    elem.addEventListener("click", function (event) {
        event.target.removeAttribute("class");
    });

Note that we're using an anonymous function here. We can also use a 
named function as a value this way ---

.. code:: js

    let elem = document.querySelector("div.footer");
    elem.addEventListener("click", function remclass(event) {
        event.target.removeAttribute("class");
    });

The advantage of giving a name is that if an error occurs, the stack trace will
tell you the name of the function. The names of such "function values" have no
uniqueness constraint. Only those named functions declared at block or global
level require unique names within their block scopes.


