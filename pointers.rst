.. include:: common.txt

Pointers and memory
*******************
Most people think of pointers as an *advanced* topic. And honestly, the more you
think of them as advanced, the more you'll fear them and the less you'll
understand such a simple thing. You are not an idiot, let me prove that for you.

When you declare a variable in your program, you know that this variable
occupies a place in your memory (RAM), right? Because when you access it, when
you change its value, it must exist somewhere!

And this *somewhere* is the variable's *adress* in memory.

.. graphviz::

    digraph ram {
        rankdir=LR;
        graph [bgcolor=transparent, resolution=96, fontsize="10"];
        edge [arrowsize=.5, arrowhead="vee", color="#ff6600", penwidth=.4];
        node [shape=record, fontsize=8, height=.2, penwidth=.4]
        ram[label="<fs>...|<f0>i|<f1>ok|<f2>hello|<f3>...|<f4>f|<fe>..."];
        node[shape="oval", fixedsize="false"]
        exnode0 [label="var i int"];
        exnode1 [label="var ok bool"];
        exnode2 [label="var hello string"];
        exnode4 [label="var f float32"];
        exnode0->ram:f0;
        exnode1->ram:f1;
        exnode2->ram:f2;
        exnode4->ram:f4;
        RAM[label="This is your RAM", shape=plaintext, fontcolor=blue];
        RAM->ram:n [color=blue, style=dotted];
    }

Now, when you buy your RAM, it's nowadays sold in GB (Giga Bytes), that is how
many *bytes* are available to be used by your program and its variables. And you
guess, that the string "Hello, world" doesn't occupy the same amount of RAM as a
simple integer.

For example, the variable ``i`` can occupy 4 bytes of your RAM starting from the
100th byte to the 103th, and the string ``"Hello world"`` can occupy more ram
starting from the 105th byte to the 116th byte.

The first byte's number of the space occupied by the variable is called its
adress.

And we use the ``&`` operator to find out the adress of a given variable.

.. code-block:: go
    :linenos:

    package main

    import "fmt"

    var (
        i int = 9
        hello string = "Hello world"
        pi float32 = 3.14
        c complex64 = 3+5i
    )

    func main() {
        fmt.Println("Hexadecimal adress of i is: ", &i)
        fmt.Println("Hexadecimal adress of hello is: ", &hello)
        fmt.Println("Hexadecimal adress of pi is: ", &pi)
        fmt.Println("Hexadecimal adress of c is: ", &c)
    }

Output:

.. container:: output

    | Hexadecimal adress of i is:  0x4b8018
    | Hexadecimal adress of hello is:  0x4b8108
    | Hexadecimal adress of pi is:  0x4b801c
    | Hexadecimal adress of c is:  0x4b80b8

You see? Each variable has its own adress, and you can guess that the adresses
of variables depend on the amount of RAM occupied by some other variables.

The variables that can *store* other variables' adresses are called *pointers*
to these variables.

How to declare a pointer?
=========================
For any given type ``T``, there is an associated type ``*T`` for pointers
to variables of type ``T``.

Example:

.. code-block:: go
    :linenos:

    var i int
    var hello string
    var p *int //p is of type *int, so p is a pointer to variables of type int

    //we can assign to p the adress of i like this:
    p = &i //now p points to i (i.e. p stores the adress of i
    hello_ptr := &hello //hello_ptr is a pointer variable of type *string and it points hello


How to access the value of a pointed-to variable?
=================================================

If ``Ptr`` is a pointer to ``Var`` then ``Ptr == &Var``. And ``*Ptr == Var``.

In other words: you precede a variable with the ``&`` operator to obtain its
adress (a pointer to it), and you precede a pointer by the ``*`` operator to
obtain the value of the variable it points to. And this is called:
*dereferencing*.

.. admonition:: Remember

    * ``&`` is called the adress operator.
    * ``*`` is called the dereferencing operator.

.. code-block:: go
    :linenos:

    package main
    import "fmt"

    func main() {
        hello := "Hello, mina-san!"
        //declare a hello_ptr pointer variable to strings
        var hello_ptr *string
        // make it point our hello variable. i.e. assign its adress to it
        hello_ptr = &hello
        // and int variable and a pointer to it
        i := 6
        i_ptr := &i //i_ptr is of type *int

        fmt.Println("The string hello is: ", hello)
        fmt.Println("The string pointed to by hello_ptr is: ", *hello_ptr)
        fmt.Println("The value of i is: ", i)
        fmt.Println("The value pointed to by i_ptr is: ", *i_ptr)
    }

Output:

.. container:: output

    | The string hello is:  Hello, mina-san!
    | The string pointed to by hello_ptr is:  Hello, mina-san!
    | The value of i is:  6
    | The value pointed to by i_ptr is:  6

.. graphviz::

    digraph ram {
        rankdir=LR;
        graph [bgcolor=transparent, resolution=96, fontsize="10" ];
        edge [arrowsize=.5, arrowtail="dot", color="#ff6600", penwidth=.4];
        node [shape=box, fontsize=8, height=.1, penwidth=.4]
        i [label="var i int"];
        i_ptr [label="var i_ptr *int", shape="oval"];
        hello [label="var hello string"];
        hello_ptr [label="var hello_ptr *string", shape="oval"];
        i_ptr->i;
        hello_ptr->hello;
    }


And that is all! Pointers are variables that store other variables adresses, and
if a variable is of type ``int``, its pointer will be of type ``*int``, if it is
of type ``float32``, its pointer is of type ``*float32`` and so on...

Why do we need pointers?
========================
Since we can access variables, assign values to them using only their names, one
might wonder why and how pointer are useful.
This is a very legitimate question, and you'll see the answer is quite simple.

Suppose that your program as it runs needs some to store some results in some
new variables not already declared. How would it do that?
You'll say: I'll prepare some variables just in case. But wait, what if the
number of new variables differs on each execution of your program?

The answer is *runtime allocation*: your program will *allocate* some memory to
store some data while it is runing.

.. _function-new:

Go has a built-in allocation function called ``new`` that allocates enough
memory to store a variable of a given type, and after it allocates the memory,
it gives you a pointer to it.

You use it like this: ``new(Type)`` where ``Type`` is the type of the variable
you want to use.

Here's an example to explain the ``new`` function.

.. code-block:: go
    :linenos:

    package main
    import "fmt"

    func main() {
        sum := 0
        var  double_sum *int //a pointer to int
        for i:=0; i<10; i++{
            sum += i 
        }
        double_sum = new(int) //allocate memory for an int and make double_sum point to it
        *double_sum = sum*2 //use the allocated memory, by dereferencing double_sum
        fmt.Println("The sum of numbers from 0 to 10 is: ", sum)
        fmt.Println("The double of this sum is: ", *double_sum)
    }

.. container:: output

    | The sum of numbers from 0 to 10 is:  45
    | The double of this sum is:  90

Another good reason, that we will see when we will study *functions* is that
passing parameters by reference can be useful in many cases. But that's a story
for another day. Until then, I hope that you have assimilated the notion of
pointers, but don't worry, we will work with them gradually in the next
chapters, and you'll see that the fear some people have about them is
unjustified.
