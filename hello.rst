.. include:: common.txt

Konnichiwa, World!
******************
Before starting to write Go applications like a wizard, we should start with the
very basic ingredients. You just can't craft an airplane when you don't know
what a wing is! So in this chapter we're going to learn the very basic syntax
idioms to get our feet wet.

The program
===========
This is a tradition. And we do respect some traditions [#f1]_. The very first program
one should write when learning a programming language is a small one that
outputs the sentence: ``Hello World``. 

Ready? Go!

.. code-block:: go
    :linenos:

    package main

    import "fmt"

    func main() { 
        fmt.Printf("Hello, world; καλημ ́ρα κóσμ or こんにちは世界\n")
    }

Output:

.. container:: output

    | Hello, world; καλημ ́ρα κóσμ or こんにちは世界

The details
===========

The notion of packages
----------------------
A Go program is constructed as a "package", which may use facilities from other
packages.

The statement ``package <something>`` (in our example: ``<something>`` is main)
tells which package this file is part of. 

In order to print on screen the text "Hello, world..." we use a function called
``Printf`` that comes from the ``fmt`` package that we imported using the
statement ``import "fmt"`` on the 3rd line.

.. note::
    Every standalone Go program contains a package called main
    and its main function, after initialization, is where execution starts.
    The main.main function takes no arguments and returns no value.

This is actually the same idea as in perl packages and python modules. And as
you may see, the big advantage here is: modularity and reusability.

By *modularity*, I mean that you can divide your program into sevral pieces each
one answering a specific need. And by *reusability*, I mean that packages that
you write, or that already are brought to your by Go itself, can be used many
times without rewriting the functionalities they provide each and every time you
need them.

For now, you can just think of packages from the perspective of their *rasion
d'être* and advantages. Later we will see how to create our own packages.

The main function
-----------------
On line 5, we declare our main function with the keyword ``func`` and its body
is enclosed between ``{`` and ``}`` just like in C, C++, Java and other
languages. 

Notice that our ``main()`` function takes no arguments. But we will see when we
will study functions, that in Go, they can take arguments, and return no, or one,
or many values!

So on line 6, we called the function ``Printf`` that is defined in the ``fmt``
packages that we imported in line 3.

The ``dot notation`` is what is used in Go to refer to data, or functions
located in imported packages. Again, this is not a Go's invention, since this
technique is used in sevral other languages such as Python:
``<module_name>.<data_or_function>``

UTF-8 Everywhere!
-----------------
Notice that the string we printed using ``Printf`` contains non-ASCII
characters (greek and japanese). In fact, Go, natively supports UTF-8 for string
*and* identifiers.

Conclusion
==========
Go uses packages (like python's modules) to organize code. The function
``main.main`` (function main located the in main package) is the entry-point of
every standalone Go program. Go uses UTF-8 for strings and identifiers and is
not limited to the old ASCII character set.


.. external links and footnotes:

.. [#f1] It is said that the first "hello, world" program was written by Brian
    Kernighan in his tutorial "Programming in C: A Tutorial" at Bell Labs in
    1974. And there is a "hello, world" example in the seminal book "The C
    programming Language" much often refered to as "K&R" by Brian Kernighan and
    Dennis Ritchie.
