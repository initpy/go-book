.. include:: common.txt

Functions, why and how
**********************
In the previous two chapters, we saw the basic data types, and the basic
control structures. They're basic in the sense that they are useful for some
very basic needs. And as the needs, or the problem to solve with a program
become complex, you'll feel the the need for complex data, and control
structures.

The problem
===========
Let's say that we have to write a program to retrieve the maximum value of
two integers A and B. That's easy to do with a simple *if* statement, right?

.. code-block:: go
    :linenos:

    //compare A and B and say wich is bigger
    if A>B {
        fmt.Print("The max is A")
    } else {
        fmt.Print("The max is B")
    }

Now, imagine that we have to make such a comparison many times in our program.
Writing the previous snippet every time becomes a real chore, and error-prone.
This is why we need to *encapsulate* this snippet in a code block, give it a
name and then call this code block every time we need to know the max of two
numbers.  This code block is what we call a function.

.. graphviz::

    digraph function {
        rankdir=LR;
        graph [bgcolor=transparent, resolution=96, fontsize="10"];
        node [shape=circle, fontsize=8, fixedsize=true, penwidth=.4];
        edge [arrowsize=.5, arrowtail="dot", color="#555555", penwidth=.4];
        block[label = "Function", shape="box", color=antiquewhite3, style=filled, peripheries=2];
        input1->block [color="#ff6600"];
        input2->block [color="#ff6600"];
        input3->block [color="#ff6600"];
        block->output1;
        block->output2;
        { rank=same; input1 input2 input3}
    }


A *function* is a code block that takes some items and returns some results.

If you've ever done some electronics, you'd see that functions really look like
electronic circuits or chips. Need to amplify a signal? Take an amplifier
chipset. Need to filter? Use a filter chipset and so on...

Your job as a software developer will be much easier, and far more fun once you
start to think of a complex project as a set of modules for which the input is
from the output of another module. The complexity of the solution will be
reduced, and the debugging will be made easier also.

Let's see how to apply this sort of design to the following problem:
suppose we have a function MAX(A, B) which takes two integers {A, B} and returns
a single integer output M which is the larger value of both A and B:

.. graphviz::

    digraph MAX {
        rankdir=LR;
        graph [bgcolor=transparent, resolution=96, fontsize="10" ];
        node [shape=circle, fontsize=8, fixedsize=true, penwidth=.4];
        edge [arrowsize=.5, arrowtail="dot", color="#555555", penwidth=.4];
        block[label = "MAX", shape="box", color=antiquewhite3, style=filled, peripheries=2];
        A->block [color="#ff6600"];
        B->block [color="#ff6600"];
        block->M;
    }

By reusing the function ``MAX`` we can compute the maximum value of three
values: A, B and C as follows:

.. graphviz::

    digraph MAX {
        rankdir=LR;
        graph [bgcolor=transparent, resolution=96, fontsize="10" ];
        node [shape=circle, fontsize=8, fixedsize=true, penwidth=.4];
        edge [arrowsize=.5, arrowtail="dot", color="#555555", penwidth=.4];
        max1[label = "MAX", shape="box", color=antiquewhite3, style=filled, peripheries=2];
        max2[label = "MAX", shape="box", color=antiquewhite3, style=filled, peripheries=2];
        A->max1 [color="#ff6600"];
        B->max1 [color="#ff6600"];
        max1->max2;
        C->max2 [color="#ff6600"];
        max2->X;
        { rank=same; A B C}
    }


This is a very simple example actually. In practice you probably should write a
``MAX3`` function that takes 3 input integers and returns a single integer,
instead of reusing the function ``MAX``.

Of course, you can *inject* the output of a function into another if and only if
the types match.

How to do this in Go?
=====================
The general syntax of a function is:

.. code-block:: go
    :linenos:

    func funcname(input1 type1, input2 type2) (output1 type1, output2 type2){
        //some code and processing here
        ...
        //return the results of the function
        return value1, value2
    }

The details:

* The keyword ``func`` is used to declare a function of name *funcname*.
* A function may take as many input parameters as required. Each one followed
  by its type and all separated by commas.
* A function may return many result values.
* In the previous snippet, ``result1`` and ``result2`` are called
  *named result parameters*, if you don't want to name the return parameters,
  you can instead specify only the types, separated with commas.
* If your function returns only one output value, you may omit the parenthesis
  arround the output declaration portion.
* If your function doesn't return any value, you may omit it entirely.

Some examples to better understand these rules:

A simple Max function
---------------------
.. code-block:: go
    :linenos:

    package main
    import "fmt"

    //return the maximum between two int a, and b.
    func max(a, b int) int{
        if a>b {
            return a
        }
        return b
    }

    func main(){
        x := 3
        y := 4
        z := 5


        max_xy := max(x, y) //calling max(x, y)
        max_xz := max(x, z) //calling max(x, z)

        fmt.Printf("max(%d, %d) = %d\n", x, y, max_xy)
        fmt.Printf("max(%d, %d) = %d\n", x, z, max_xz)
        fmt.Printf("max(%d, %d) = %d\n", y, z, max(y,z)) //just call it here
    }

Output:

.. container:: output

    | max(3, 4) = 4
    | max(3, 5) = 5
    | max(4, 5) = 5


Our function *MAX* takes two input parameters A and B, and returns a single
*int*.
Notice how we grouped A and B's types. We could have written: MAX(A int, B int),
however, it is shorter this way.

Notice also that we prefered not to name our output value. We instead specified
the output type (*int*).

A function with two ouput values
--------------------------------

.. code-block:: go
    :linenos:

    package main
    import "fmt"

    //return A+B and A*B in a single shot
    func SumAndProduct(A, B int) (int, int){
        return A+B, A*B
    }

    func main(){
        x := 3
        y := 4

        xPLUSy, xTIMESy := SumAndProduct(x, y)

        fmt.Printf("%d + %d = %d\n", x, y, xPLUSy)
        fmt.Printf("%d * %d = %d\n", x, y, xTIMESy)
    }

Output:

.. container:: output

    | 3 + 4 = 7
    | 3 * 4 = 12


A function with a result variable
---------------------------------
.. code-block:: go
    :linenos:

    package main

    //look how we grouped the import of packages fmt and math
    import(
        "fmt"
        "math"
    )

    //A function that returns a bool that is set to true of Sqrt is possible
    //and false when not. And the actual square root of a float64
    func MySqrt(f float64) (ok bool, s float64){
        if f>0 {
            ok, s = true, math.Sqrt(f)
        } else {
            ok, s = false, 0
        }
        return ok, s
    }

    func main(){
        for i:= -2.0; i<=10; i++{
            possible, sqroot := MySqrt(i)
            if possible{
                fmt.Printf("The square root of %f is %f\n", i, sqroot)
            } else {
                fmt.Printf("Sorry, no square root for %f\n", i)
            }
        }
    }

Outputs:

.. container:: output

    | Sorry, no square root for -2.000000
    | Sorry, no square root for -1.000000
    | Sorry, no square root for 0.000000
    | The square root of 1.000000 is 1.000000
    | The square root of 2.000000 is 1.414214
    | The square root of 3.000000 is 1.732051
    | The square root of 4.000000 is 2.000000
    | The square root of 5.000000 is 2.236068
    | The square root of 6.000000 is 2.449490
    | The square root of 7.000000 is 2.645751
    | The square root of 8.000000 is 2.828427
    | The square root of 9.000000 is 3.000000
    | The square root of 10.000000 is 3.162278


Here we *import* the package "math" in order to use the *Sqrt* (Square root)
function it provides. We then write our own function ``MySqrt`` which returns
two values: the first is boolean (indicates whether a square root is
possible) and the second is the actual sqaure root of the input parameter ``f``.

Notice how we use the parameters ``ok`` and ``s`` as actual variables in the
function's body.

Since result variables are initialized to "zero" (0, 0.00, false...) according
to its type, we can rewrite the previous example as follows:

.. code-block:: go
    :linenos:

    import "math"

    //return A+B and A*B in a single shot
    func MySqrt(f float64) (ok bool, s float64){
        if f>0 {
            ok, s = true, math.Sqrt(f)
        }
        return ok, s
    }

The empty return
----------------
When using named output variables, if the function executes a return statement
with no arguments, the current values of the result parameters are used as the
returned values.

Thus, we can rewrite the previous example as follow:

.. code-block:: go
    :linenos:

    import "math"

    //return A+B and A*B in a single shot
    func MySqrt(f float64) (ok bool, s float64){
        if f>0 {
            ok, s = true, math.Sqrt(f)
        }
        return //omitting the output named variables, but keeping "return"
    }

.. _value-reference:

Parameters by value, and by reference
=====================================
When passing an input parameter to a function, it actually recieves a *copy* of
this parameter. So, if the function changes the value of this parameter, the
original variable's value won't be changed, because the function works on a
*copy* of the original variable's value.

An example to verify the previous paragraph:

.. code-block:: go
    :linenos:

    package main
    import "fmt"

    //simple function that returns 1 + its input parameter
    func add1(a int) int{
        a = a+1 // we change the value of a, by adding 1 to it
        return a //return the new value
    }

    func main(){
        x := 3

        fmt.Println("x = ", x) //should print "x = 3"

        x1 := add1(x) //calling add1(x)

        fmt.Println("x+1 = ", x1) //should print "x+1 = 4"
        fmt.Println("x = ", x) //will print "x = 3"
    }

.. container:: output

    | x =  3
    | x+1 =  4
    | x =  3

You see? The value of ``x`` wasn't changed by the call of the function ``add1``
even though we had an explicit ``a = a+1`` instruction on line 6.

The reason is simple: when we called the function ``add1``, it recieved a *copy*
of the variable ``x`` and not ``x`` itself, hence it changed the copy's value,
not the value of ``x`` itself.

I know the question you have in mind:
*"What if I wanted the value of x to be changed by calling the function?"*

And here comes the utility of pointers: Instead of writing a function that has
an ``int`` input parameter, we should give it a pointer to the variable we want
to change! This way, the function has *access* to the original variable and it
can change it.

Let's try this.

.. code-block:: go
    :linenos:

    package main
    import "fmt"

    //simple function that returns 1 + its input parameter
    func add1(a *int) int{ //notice that we give it a pointer to an int!
        *a = *a+1 // we dereference and change the value pointed by a
        return *a //return the new value
    }

    func main(){
        x := 3

        fmt.Println("x = ", x) //should print "x = 3"

        x1 := add1(&x) //calling add1(&x) by passing the adress of x to it

        fmt.Println("x+1 = ", x1) //should print "x+1 = 4"
        fmt.Println("x = ", x) //will print "x = 4"
    }

.. container:: output

    | x =  3
    | x+1 =  4
    | x =  4

Now, we have changed the value of ``x``!

How is passing a reference to functions is useful? You may ask.

* The first reason is that passing a reference makes function *cooperation* on
  a single variable possible. In other words, if we want to apply several
  functions on a given variable, they will all be able to change it.

* A pointer is cheap. Cheap in terms of memory usage. We can have functions that
  operate on a big data value for example. Copying this data will, of course,
  require memory on each call as it is copied for use by the function.
  In this way, a pointer takes much less memory.
  Yo! Remember, it's just an adress! :)

.. _functions-signatures:

Functions signatures
====================
Like with people, names don't matter that much. Do they? Well, yes, let's face
it, they do matter, but what matters most is what they do, what they need, what
they produce (Yeah you! Get back to work!).

In our previous example, we could have called our function **add_one** instead of
**add1**, and that would still work as long as we call it by its name.
What really matters in our function is:

1. Its input parameters: how much? which types?
2. Its output parameters: how much, which type?
3. Its body code: What does the function do?

We can rewrite the function's body to work differently, and the main program
would continue to compile and run without problems. But we can't change the
input and output parameters in the function's declaration and still use its old
parameters in the main program.

In other words, of the 3 elements we listed above, what matters even more is:
what input parameters does the function expect, and what output parameters does
it return.

These two elements are what we call a *function signature*, for which we use
this formalism:

``func (input1 type1 [, input2 type2 [, ...]]) (output1 OutputType1 [, output2 OutputType2 [,...]])``

Or optionally with the function's name:

``func function_name (input1 type1 [, input2 type2 [, ...]]) (output1 OutputType1 [, output2 OutputType2 [,...]])``

Examples of signatures
----------------------

.. code-block:: go
    :linenos:

    //The signature of a function that takes an int and returns and int
    func (int x) x int

    //takes two float and returns a bool
    func (float32, float32) bool

    // Takes a string returns nothing
    func (string)

    // Takes nothing returns nothing
    func()

