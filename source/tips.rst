Some utility tips to help you
=============================

Code formatting
---------------

Different programmers format code differently when they're starting out. Once
you begin working in a team and reading another person's code becomes
important, the team typically adopts common formatting conventions.

There is a broad consensus in the python community on how to format python
code. Thankfully you **don't** have to learn these conventions, because there
is tool that will automatically format your code for you - `black`_. So install
``black`` using ``pip install black`` and run it on your python files before
you commit them to git --- like ``black *.py``. Your team-mates will thank you.

.. _black: https://black.readthedocs.io/en/stable/index.html

If this sounds like a chore, you ought to have guessed the next step -- it must
be *automated*! The package `pre-commit`_ can be used to setup what are called
"git pre-commit hooks" that can be used to perform certain normalizing
operations on your project files before committing to git (yes, even before
pushing). Such "hooks" are not only used to format code, but they can also run
your tests before committing. Go through the `pre-commit`_ documentation to see
how to set it up for automatic python code formatting using ``black`` (if you
want to automate it, that is).

.. _pre-commit: https://pre-commit.com/

Even better, figure out how to automatically run ``black`` on your python file
**every time you save it** in your editor. This varies from editor to editor.

In the Javascript world, the corresponding tool is `prettier`_.

**Important**: Don't fiddle with the default configurations of these tools.
They're defaults for a good reason and instead adapt your style to the output
of these tools gradually.

.. _prettier: https://prettier.io/

