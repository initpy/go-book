.. include:: common.txt

Basic Control flow
******************
In the previous chapter, we learned how to declare variables and constants.
We also saw the different basic, builtin types that come with Go. In this
chapter, we will see the basic control constructions, or how to *reason about*
this data. But before that, let's talk about comments and semicolons.

Comments
========
You probably noticed in the example snippets given in previous chapters, some
comments. Didn't you?
Go has the same commenting conventions as in C++. That is:

* Single line comments: Everything from ``//`` until the end of the line is a
  comment.
* bloc comments: Everything betwwen ``/*`` and ``*/`` is a comment.

.. code-block:: go
    :linenos:

    //Single line comment, this is ignored by the compiler.
    /* Everything from the beginning of this line until the keyword 'var' on
    line 4 is a comment and ignored by the compiler. */
    var(
        integer int
        pi = float32
        prefix string
    )

Semicolons
==========
If you've programmed with C, or C++, or Pascal, you might wonder why, in the
previous examples, there weren't any semicolons.
In fact, you don't need them in Go. Go automatically inserts semicolons on every
line end that looks like the end of a statement.

And this makes the code a lot cleaner, and easy on the eyes.

The only place you will typically see semicolons is separating the clauses of
for loops and the like; they are not necessary after every statement.

Note that you can use semicolons to separate several statements written on a
single line.

The one surprise is that it's important to put the opening brace of a construct
such as an if statement on the same line as the if; if you don't, there are
situations that may not compile or may give the wrong result [#f1]_

Ok let's begin, now.

The if statement
================
Perhaps the most well-known statement type in imperative programming. And it can
be summarized like this: if condition met, do this, else do that.

In Go, you don't need parenthesis for the condition.

.. code-block:: go
    :linenos:

    if x > 10 {
        fmt.Println("x is greater than 10")
    } else {
        fmt.Println("x is less than 10")
    }

You can also have a leading initial short statement before the condition too.

.. code-block:: go
    :linenos:

    // Compute the value of x, and then compare it to 10.
    if x := computed_value(); x > 10 {
        fmt.Println("x is greater than 10")
    } else {
        fmt.Println("x is less than 10")
    }

You can combine multiple if/else statements.

.. code-block:: go
    :linenos:

    if integer == 3 {
        fmt.Println("The integer is equal to 3")
    } else if integer < 3 {
        fmt.Println("The integer is less than 3")
    } else {
        fmt.Println("The integer is greater than 3")
    }

The for statement
=================
The ``for`` statement is used to iterate and loop.
Its general syntax is:

.. code-block:: go
    :linenos:

    for expression1; expression2; expression3{
        ...
    }

Gramatically, ``expression1``, ``expression2``, and ``expression3`` are
expressions, where ``expression1``, and ``expression3`` are used for
assignements or function calls (``expression1`` before starting the loop, and
``expression3`` at the end of every iteration) and ``expression2`` is used to
determine whether to continue or stop the iterations.

An example would be better than the previous paragraph, right? Here we go!

.. code-block:: go
    :linenos:

    package main
    import "fmt"

    func main(){
        sum := 0;
        for index:=0; index < 10 ; index++ {
            sum += index
        }
        fmt.Println("sum is equal to ", sum)
    }

In the code above we initalize our variable ``sum`` to 0. The ``for`` loop by
is begun by initializing the variable ``index`` to 0 (``index:=0``).

Next the ``for`` loop's body is executed (``sum += i``) *while* the condition
``index < 10`` is true.

At the end of each iteration the ``for`` loop will execute the ``index++``
expression (eg. increments ``index``).

You might guess that the output from the previous program would be:

.. container:: output

    | sum is equal to  45

And you would be correct!
Because the sum is 0+1+2+3+4+5+6+7+8+9 which equals 45.

**Question:**

  Is it possible to combine lines 5 and 6 into a single one, in the previous
  program? How?

Actually ``expression1``, ``expression2``, and ``expression3`` are all optional.
This means you can omit, one, two or all of them in a ``for`` loop. If you omit
``expression1`` or ``expression3`` it means they simply won't be executed. If
you omit ``expression2`` that'd mean that it is always true and that the loop
will iterate forever -- unless a ``break`` statement is encountered.

Since these expressions may be omitted, you can have something like:

.. code-block:: go
    :linenos:

    //expression1 and expression3 are omitted here
    sum := 1
    for ; sum < 1000;  {
        sum += sum
    }

Which can be simplified by removing the semicolons:

.. code-block:: go
    :linenos:

    //expression1 and expression3 are omitted here, and semicolons gone.
    sum := 1
    for sum < 1000 {
        sum += sum
    }

Or when even expression2 is omitted also, we can have something like:

.. code-block:: go
    :linenos:

    //infinite loop, no semicolons at all.
    for {
        fmt.Println("I loop for ever!")
    }

break and continue
==================

With ``break`` you can stop loops early. i.e. finish the loop even if the
condition expressed by ``expression2`` is still true.

.. code-block:: go
    :linenos:

    for index := 10; index>0; index-- {
        if index < 5{
            break
        }
        fmt.Println(index)
    }

The previous snippet says: loop from 10 to 0 printing the numbers (line 6); but
if i<5 then break (stop) the loop!
Hence, the program will print the numbers: 10, 9, 8, 7, 6, 5.

``continue``, on the other hand will just break the current iteration and
*skips* to the next one.

.. code-block:: go
    :linenos:

    for index := 10; index > 0; index-- {
        if index == 5 {
            continue
        }
        fmt.Println(index)
    }

Here the program will print all the numbers from 10 down to 1, except 5! Because
on line 3, the condition ``if index == 5`` will be ``true`` so ``continue``
will be executed and the ``fmt.Println(index) (for index == 5)`` won't be run.

The switch statement
====================
Sometimes, you'll need to write a complicated chain of ``if``/``else`` tests.
And the code becomes ugly and hard to read and maintain. The ``switch``
statement, makes it look nicer and easier to read.

Go's switch is more flexible than its C equivalent, because the case expression
need not to be constants or even integers.

The general form of a switch statement is

.. code-block:: go
    :linenos:

    // General form of a switch statement:
    switch sExpr {
        case expr1:
            some instructions
        case expr2:
            some other instructions
        case expr3:
            some other instructions
        default:
            other code
    }

The type of ``sExpr`` and the expressions ``expr1``, ``expr2``, ``expr3``...
type should match. i.e. if ``var`` is of type ``int`` the cases expressions should
be of type ``int`` also!

Of course, you can have as many cases you want.

Multiple match expressions can be used, each separated by commas in the case
statement.

If there is no ``sExpr`` at all, then by default it's *bool* and so should be
the cases expressions.

If more than one case statements match, then the first in lexical order is
executed.

The *default* case is optional and can be anywhere within the switch block,
not necessarily the last one.

Unlike certain other languages, each case block is independent and code does not
"fall through" (each of the case code blocks are like independent if-else-if
code blocks. There is a fallthrough statement that you can explicitly use to
obtain fall through behavior if desired.)

Some switch examples:
---------------------

.. code-block:: go
    :linenos:

    // Simple switch statement example:
    i := 10
    switch i {
        case 1:
            fmt.Println("i is equal to 1")
        case 2, 3, 4:
            fmt.Println("i is equal to 2, 3 or 4")
        case 10:
            fmt.Println("i is equal to 10")
        default:
            fmt.Println("All I know is that i is an integer")
    }

In this snippet, we initialized ``index`` to 10, but in practice, you should
think of ``index`` as a computed value (result of a function or some other
calculus before the switch statement)

Notice that in line 6, we grouped some expressions (2, 3, and 4) in a single
case statement.

.. code-block:: go
    :linenos:

    // Switch example without sExpr
    index := 10
    switch {
        case index < 10:
            fmt.Println("The index is less than 10")
        case index > 10, index < 0:
            fmt.Println("The index is either bigger than 10 or less than 0")
        case index == 10:
            fmt.Println("The index is equal to 10")
        default:
            fmt.Println("This won't be printed anyway")
    }

Now, in this example, we omitted ``sExpr`` of the general form. So the cases
expressions should be of type *bool*. And so they are! (They're comparisons that
return either ``true`` or ``false``)

**Question**:

  Can you tell why the ``default`` case on line 10 will never be reached?


.. code-block:: go
    :linenos:

    //switch example with fallthrough
    integer := 6
    switch integer {
    case 4:
        fmt.Println("The integer was <= 4")
        fallthrough
    case 5:
        fmt.Println("The integer was <= 5")
        fallthrough
    case 6:
        fmt.Println("The integer was <= 6")
        fallthrough
    case 7:
        fmt.Println("The integer was <= 7")
        fallthrough
    case 8:
        fmt.Println("The integer was <= 8")
        fallthrough
    default:
        fmt.Println("default case")
    }

In this example, the case on line 10 matches, but since there is
``fallthrough``, the code for ``case 7:`` and so on will also be executed
(just like a C switch that doesn't use the ``break`` keyword).


.. [#f1] http://golang.org/doc/go_tutorial.html#tmp_33
