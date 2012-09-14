.. include:: common.txt

If you're not a programmer
**************************
This chapter is for people who have never written a single program in an
imperative programming language. If you are used to languages likes C, C++, Python, Perl,
Ruby or if you know what the words variable, constant, type,
instruction, assignation mean you may (No! You SHOULD!) skip this section entirely.


How do programs work?
=====================
In the programming world there are many families of languages: functional,
logic, imperative.

Go, like C, C++, Python, Perl and some others are said to be imperative
languages because you write down what the processor should execute. It's an
imperative way to describe tasks assigned to the computer.

An imperative program is a sequence of *instructions* that the processor
executes when the program is compiled and *transformed* to a binary form that
the processor understands.

Think of it as a list of steps to assemble or build something. We said
*"sequence"* because the **order** of these steps generally matters a LOT! You
can't paint the walls of a house when you haven't built the walls yet. See?
Order matters!

What are variables?
===================
When you solve math problems (mind you it won't be about math this time), you
say that the variable ``x`` is set to the value ``1`` for example. 

Sometimes you have to solve equations like: ``2x + 1 = 5``. Solving here means
finding out the value of the variable ``x``.

So it's safe to say that ``x`` is a container that can contain different values
of a given type.

Think of them as boxes labeled with *identifiers* that store values.

What is assignation?
====================
We assign a value to a variable to say that this variable is *equal* to this
value from the assignation act on, until another assignation changes its value.

.. code-block:: go
    :linenos:

    //assign the value 5 to the variable x
    x = 5
    // Now x is equal to 5. We write this like this: x == 5
    x = x + 3 //now, we assign to x its value plus 3
    //so now, x == 8

Assignation is the act of *storing* values inside variables. In the previous
snippet, we assigned the value ``5`` to the variable whose identifier is ``x``
(we say simply the variable ``x``) and just after, we assigned to the variable
``x`` its own value (``5`` is what we assigned to it in the previous
instruction) plus ``3``. So now, x contains the value ``5+3`` which is ``8``.

.. graphviz::

    digraph variable_use {
        //rankdir=LR;
        graph [bgcolor=transparent, resolution=96, fontsize="10"];
        edge [arrowsize=.5, arrowhead="vee", color="#ff6600", penwidth=.4];
        node [shape=box, fontsize=8, height=.2, penwidth=.4]
        value[shape="circle", label="5", fixedsize="false"]
        value2[shape="circle", label="x+3", fixedsize="false"]
        variable[label="x"];
        variable2[label="x"];
        value->variable
        value2->variable2
        {rank=sink; variable2 value2}
    }

What is a type?
===============
Each declared variable is of a given type. A type is a set of the values that
the variable can store. In other words: A type is the set of values that we can
assign to a given variable.

For example: A variable x of the type ``int``\eger can be used to store
integer values: 0, 1, 2, ...

The phrase *"Hello world"* can be assigned to variables of type ``string``.

So we actually *declare* variables of a given *type* in order to use them with
values of that type. 

What is a constant?
===================
A constant is an identifier that is assigned a given value, which can't be
changed by your program.

We *declare* a constant with a given value.

.. code-block:: go
    :linenos:

    // Pi is a constant whose value is 3.14
    const Pi = 3.14

    // Let's use the constant Pi
    var x float32 //we declare a variable x of type float32
    x = 3 * Pi //when compiled, x == 3 * 3.14

    var y float32
    x = 4 * Pi //when compiled, y == 4 * 3.14

For example, we can declare the constant ``Pi`` to be equal to ``3.14``. During
the execution of the program, the value of ``Pi`` won't and can not be changed.
There is no assignation of values to constants.

Constants are useful to make programs easy to read and to maintain. In effect,
writing ``Pi`` instead of the floating point number ``3.14`` in many spots of
your program make it easier to read.

Also, if we decide to make the ouput of our program that uses ``Pi`` more
precise, we can change the constant's declared value once to be ``3.14159``
instead of the earlier ``3.14`` and the new value will be used all along when
the program is compilled again.

Conclusion
==========
Imperative programming is easy. Easy as putting values in boxes. Once you
understand this very basic thing, complex (notice, I didn't say complicated)
things can be seen as a composition of simpler things.

Easy and fun. Fun as playing with Lego sets :)
