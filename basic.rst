.. include:: common.txt

The basic things
****************
In the previous chapter, we saw that Go programs are organized using *packages*
and that Go natively supports UTF-8 for strings and identifiers. In this chapter
we will see how to declare and use variables and constants and the different Go
built-in types.


How to declare a variable?
==========================
There are several ways to declare a variable in Go.

The basic form is:

.. code-block:: go

    // declare a variable named "variable_name" of type "type"
    var variable_name type

You can declare several variables of the same type in a single line, by
separating them with commas.

.. code-block:: go

    // declare variables var1, var2, and var3 all of type type
    var var1, var2, var3 type

And you can initialize a variable when declaring it too

.. code-block:: go

    /* declare a variable named "variable_name" of type "type" and initialize it
        to value*/
        var variable_name type = value

You can even initialize many variables that are declared in the same statement

.. code-block:: go

    /* declare a var1, var2, var3 of type "type" and initialize them to value1,
    value2, and value3 respectively*/
    var var1, var2, var3 type = value1, value2, value3

Guess what? You can omit the type and it will be inferred from the initializers

.. code-block:: go

    /* declare and initialize var1, var2 and var3 and initialize them
    respectively to value1, value2, and value3. /
    var var1, var2, var3 = value1, value2, value3

Even shorter, inside a function body (let me repeat that: only inside a function
body) you can even drop the keyword ``var`` and use the ``:=`` instead of ``=``

.. code-block:: go

    // omit var and type, and use ':=' instead of '=' inside the function body
    func test(){
        var1, var2, var3 := value1, value2, value3
    }

Don't worry. It's actually easy. The examples with the builtin types, below,
will illustrate both of these forms. Just remember that, unlike the C way, in Go
the type is put at the end of the declaration and -should I repeat it?- **that
the := operator can only be used inside a function body**.

.. _primitive-types:

The builtin types
=================

Boolean
-------
For boolean truth values, Go has the type ``bool`` (like the C++ one) that takes
one of the values: ``true`` or ``false``.

.. code-block:: go
    :linenos:

    //Example snippet
    var active bool //basic form
    var enabled, disabled = true, false //type omitted, variables initialized
    func test(){
        var available bool //general form
        valid := false //type and var omitted, and variable initialized
        available = true //normal assignation
    }


Numeric types
-------------
For integer values, signed and unsigned, Go has ``int`` and ``uint`` both having
the appropriate length for your machine (32 or 64 bits) But there's also
explicit sized ints: ``int8``, ``int16``, ``int32``, ``int64`` and ``byte``,
``uint8``, ``uint16``, ``uint32``, ``uint64``. With ``byte`` being an alias for
``uint8``.

For floating point values, we have ``float32`` and ``float64``.

Wait that's not all, Go has native support for complex numbers too! In fact, you
can use  ``complex64`` for numbers with 32 bits for the real part and 32 bits
for the imaginary part, and there is ``complex128`` for numbers with 64 bits for
the real part and 64 bits for the imaginary part.

Table of numeric types
^^^^^^^^^^^^^^^^^^^^^^

From the `Go Programming Language Specification`_

+----------+----------------------------------------------------------------------------------------+
|Type      | Values                                                                                 |
+==========+========================================================================================+
|uint8     | the set of all unsigned  8-bit integers (0 to 255)                                     |
+----------+----------------------------------------------------------------------------------------+
|uint16    | the set of all unsigned 16-bit integers (0 to 65535)                                   |
+----------+----------------------------------------------------------------------------------------+
|uint32    | the set of all unsigned 32-bit integers (0 to 4294967295)                              |
+----------+----------------------------------------------------------------------------------------+
|uint64    | the set of all unsigned 64-bit integers (0 to 18446744073709551615)                    |
+----------+----------------------------------------------------------------------------------------+
|                                                                                                   |
+----------+----------------------------------------------------------------------------------------+
|int8      | the set of all signed  8-bit integers (-128 to 127)                                    |
+----------+----------------------------------------------------------------------------------------+
|int16     | the set of all signed 16-bit integers (-32768 to 32767)                                |
+----------+----------------------------------------------------------------------------------------+
|int32     | the set of all signed 32-bit integers (-2147483648 to 2147483647)                      |
+----------+----------------------------------------------------------------------------------------+
|int64     | the set of all signed 64-bit integers (-9223372036854775808 to 9223372036854775807)    |
+----------+----------------------------------------------------------------------------------------+
|                                                                                                   |
+----------+----------------------------------------------------------------------------------------+
|float32   | the set of all IEEE-754 32-bit floating-point numbers                                  |
+----------+----------------------------------------------------------------------------------------+
|float64   | the set of all IEEE-754 64-bit floating-point numbers                                  |
+----------+----------------------------------------------------------------------------------------+
|                                                                                                   |
+----------+----------------------------------------------------------------------------------------+
|complex64 | the set of all complex numbers with float32 real and imaginary parts                   |
+----------+----------------------------------------------------------------------------------------+
|complex128| the set of all complex numbers with float64 real and imaginary parts                   |
+----------+----------------------------------------------------------------------------------------+
|                                                                                                   |
+----------+----------------------------------------------------------------------------------------+
| byte        familiar alias for uint8                                                              |
+----------+----------------------------------------------------------------------------------------+


.. code-block:: go
    :linenos:

    //Example snippet
    var i int32 //basic form with a int32
    var x, y, z = 1, 2, 3 //type omitted, variables initialized
    func test(){
        var pi  float32 //basic form
        one, two, three := 1, 2, 3 //type and var omitted, variables initialized
        c := 10+3i // a complex number, type infered and keyword 'var' omitted.
        pi = 3.14 // normal assignation
    }

Strings
-------
As seen in the previous chapter, strings are in UTF-8 and they are enclosed
between two double quotes (") and their type is -you bet!- ``string``.

.. code-block:: go
    :linenos:

    //Example snippet
    var french_hello string //basic form
    var empty_string string = "" // here empty_string (like french_hello) equals ""
    func test(){
        no, yes, maybe := "no", "yes", "maybe" //var and type omitted
        japanese_hello := "Ohaiou"  //type inferred, var keyword omitted
        french_hello = "Bonjour" //normal assignation
    }

Constants
=========
In Go, constants are -uh- constant values created at compile time, and they
can be: numbers, boolean or strings.

The syntax to declare a constant is:

.. code-block:: go

    const constant_name = value

Some examples:

.. code-block:: go
    :linenos:

    //example snippet
    const i = 100
    const pi = 3.14
    const prefix = "go_"

Facilities
==========

Grouping declarations
---------------------
Multiple ``var``, ``const`` and ``import`` declarations can be grouped using
parenthesis.

So instead of writing:

.. code-block:: go
    :linenos:

    //example snippet
    import "fmt"
    import "os"

    const i = 100
    const pi = 3.14
    const prefix = "go_"

    var i int
    var pi = float32
    var prefix string

You can write:

.. code-block:: go
    :linenos:

    //example snippet with grouping
    import(
        "fmt"
        "os"
    )

    const(
        i = 100
        pi = 3.14
        prefix = "go_"
    )

    var(
        i int
        pi = float32
        prefix string
    )


*Of course*, you group consts with consts, vars with vars and imports with imports
but you can not mix them in the same group!

iota and enumerations
---------------------
Go provides the keyword ``iota`` that can be used when declaring enumerated
constants, This keyword yelds an incremented value by 1, starting from 0, each
time it is used.

Example:

.. code-block:: go
    :linenos:

    //example snippet
    const(
        x = iota //x == 0
        y = iota //y == 1
        z = iota //z == 2
        w // implicitely w == iota, therefore: w == 3
    )


Well, that's it for this chapter. I told you, it won't be hard. In fact, Go
eases variable declarations a lot. You'd almost feel like coding with a
scripting language like python --and it's even better.

.. external links and footnotes:

.. _Go Programming Language Specification: http://golang.com/doc/go_spec.html#Numeric_types
