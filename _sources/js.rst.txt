
.. _js-crash-course:

A crash course in for JS
========================

(for python familiars)

Here is a subset of Javascript you should know about. There are more features,
but since most of the value of Javascript lies in the DOM APIs that are exposed
to it within the browser, you can do pretty much whatever you need with the
following "language features".

**Strings** - Strings are notated ``"some character sequence"``. If
the ``"`` character needs to feature in the string, you can prefix it with
the ``\`` escape character like ``"some \"character\" sequence"``.

**Numbers** - Pretty much the same as python. Covers signed/unsigned integers,
numbers with fixed decimal places and "floating point" numbers. All numbers in
JS are treated as double precision floating point values. So there is no separate
"integer" type, so to speak.

**Arrays** - Notated ``[val1,val2,...]`` -- i.e. comma separated sequence of
values enclosed in ``[]``.

**Objects** - These are key-value associations where the keys in a particular
object are expected to be unique strings and the values can be any Javascript value
(including functions, objects, arrays, etc.)

**Expressions** - Javascript expressions are like math expressions that are
expected to evaluate to some Javascript value. 

**Special values** - ``null`` and ``undefined`` are special values to indicate
things like "not available" or "not present" in some circumstances. For example,
if you access a key of an object that does not have the key, then it will
give you ``undefined`` as the value. Many functions in the DOM API return ``null``
to indicate "such a thing does not exist".

**Functions** - In javascript, you write named functions like this --

.. code:: js

    // This is a comment line.
    function fname(arg1, arg2, ...) {
        let <varname> = <value>; // Introduces new local variable.
        <statement1>;
        <statement2>;
        for (let i = 0; i < 100; ++i) {
            <statement3>;
            <statement4>;
        }
        for (let val of iterable-collection) {
            <something-using-val>;
        }
        for (let key in iterable-collection) {
            <something-using-key>;
            
        if (<condition>) {
            <statement5>;
        } else { // The else part is optional
            <statement6>;
        }
        switch (<value>) {
            case <const1>: {
                <branch-statement1>;
                <branch-statemnt2>;
                break;
            }
            case <const2>: {
                <branch-statement3>;
                <branch-statement4>;
            }
            ...
            default {
                <branch-statement5>;
                <branch-statement6>;
            }
        }
        try {
            <statement7>;
            <statement8>;
        } catch (e) {
            <statement9>;
        } finally {
            <statement10>;
        }
        throw <error-value>;
        return <result-value>;
    }

The important thing to keep in mind is that unlike Python, Javascript ``function ... {}``
expressions are **expressions** and not statements --- i.e. they evaluate to
function values. So you can do the following as well --

.. code:: js

    let pyth = function (x, y) { return Math.sqrt(x * x + y * y); };

This means you can make lists of functions, dictionaries of functions
and so on.

Javascript functions are also proper closures. They any variable names not
mentioned in  local ``let`` bindings or in argument list but are present in the
enclosing context will be "closed over". **This is critical for its role in
manipulating the DOM, so pay attention**. For example, consider the following
---

.. code:: js

    function counter(n) {
        return function () {
            let m = n;
            n += 1;
            return m;
        };
    }

    let countup1 = counter(5);
    let countup2 = counter(10);
    console.log(countup1()); // Prints 5
    console.log(countup2()); // Prints 10
    console.log(countup1()); // Prints 6
    console.log(countup2()); // Prints 11

If comparing to python, you **DO NOT** need declarations like ``global`` and
``nonlocal`` to get this kind of behaviour. It is the way things work already
in JS.

**Anonymous functions** - while ``function {...}`` can be used in to make
anonymous function values, another notation is available for these that is
simpler for small functions. For example, a function that squares a number can
be expressed as ``(x) => x*x``. If the body needs to be more complicated, you
can use ``(x,y,z) => {....; return <resultval>;}``. Again, this is an
expression that produces a function value and you can therefore use it wherever
a value is required.

**Dynamic typing** - Javascript is usually called a "dynamically typed" language
(like Python). What it means in this context is that a particular variable is not
required to always hold values of a certain type, but values carry information
about their own types. So ... untyped variables and typed values.

**Classes and inheritance** (NO NEED) - Javascript has what is called a
"prototype based object system" and provides a ``class`` based notation around
it. You can do a lot with JS without using this part of the language using only
the above described facets and so I will not be getting into it. Write to me if
you want to learn this and I'll see if I can hold a separate session for those
interested.

.. admonition:: **The above is a SANE subset of JS**

    Sticking to the above subset of the Javascript language will help
    you avoid common pitfalls and bugs that you'll otherwise make in your
    code.













