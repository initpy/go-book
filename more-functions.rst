.. include:: common.txt

.. This chapter might be better named 'Getting funky with functions' ;)

More on functions
*****************
In the previous chapters, we went from how to declare variables, pointers, how
to write basic control structures, how to combine these control structures to
write functions. Then we went back to data, and how to combine basic data types
to create composite data types.

And now, armed with this knowledge, we're ready to work on advanced aspects of
functions.

I decided to divide this task in two chapters, not because the things we're
about to study are hard, but mainly because it's easier, and more fun to work on
a few topics at a time.

Variadic functions
==================
Do you remember when I made you :ref:`cringe <Older100>` at the idea of writing
a function ``Older100`` that will not accept less than 100 ``person``\s?
``struct``\s as its input?

Well I lied a little bit, but it was a good lie. I duct-taped you to a chair
with that blindfold on with the best of intentions, I swear!
It was an excellent reason to learn about arrays, and later about slices!
Relax... it's for your own safety! ;)

The truth is: you can have functions that accept a *variable* number of input
parameters of the *same* type. They're called *variadic functions*.

These functions can be declared like this:

``func function_name(args ...inputType) (output1 OutputType1 [, output2 OutputType2 [, ...]])``

There's no difference from the way we learned to declare functions, except the
three dots (called an ellipses) thing:``...inputType`` which says to the function that it can
recieve a variable number of parameters all of them of type ``inputType``.

How does this work? Easy: In fact, the ``args`` identifier you see in the input
parameter is actually a slice which contains all of the parameters passed in (
which should all be of type ``inputType``).

A slice! Yay! So we can, in the function's body, iterate using ``range`` to get
all the parameters! Yes, that's the way you do it. You play the guitar on the
mtv... er, sorry. I got a case of *Mark Knopfler* fever.

Let's see an example:

.. code-block:: go
    :linenos:

    package main
    import "fmt"

    //Our struct representing a person
    type person struct{
        name string
        age int
    }

    //Return true, and the older person in a group of persons
    //Or false, and nil if the group is empty.
    func Older(people ...person) (bool, person){ //variadic function
        if(len(people) == 0){return false, person{}} //the group is empty
        older := people[0] //The first one is the older FOR NOW.
        //loop through the slice people
        for _, value := range people{ //we don't need the index
            if value.age > older.age{ //compare the current person's age with the older one
                older = value //if value is older, replace older
            }
        }
        return true, older
    }

    func main(){

        //two variables to be used by our program
        var(
            ok bool
            older person
        )

        // declare some persons
        paul := person{"Paul", 23};
        jim := person{"Jim", 24};
        sam := person{"Sam", 84};
        rob := person{"Rob", 54};
        karl := person{"Karl", 19};

        //who is older? Paul or Jim?
        _, older = Older(paul, jim) //notice how we used the blank identifier
        fmt.Println("The older of Paul and Jim is: ", older.name)
        //who is older? Paul, Jim or Sam?
        _, older = Older(paul, jim, sam)
        fmt.Println("The older of Paul, Jim and Sam is: ", older.name)
        //who is older? Paul, Jim , Sam or Rob?
        _, older = Older(paul, jim, sam, rob)
        fmt.Println("The older of Paul, Jim, Sam and Rob is: ", older.name)
        //who is older in a group containing only Karl?
        _, older = Older(karl)
        fmt.Println("When Karl is alone in a group, the older is: ", older.name)
        //is there an older person in an empty group?
        ok, older = Older() //this time we use the boolean variable ok
        if !ok {
            fmt.Println("In an empty group there is no older person")
        }
    }

Output:

.. container:: output

    | The older of Paul and Jim is:  Jim
    | The older of Paul, Jim and Sam is:  Sam
    | The older of Paul, Jim, Sam and Rob is:  Sam
    | When Karl is alone in a group, the older is:  Karl
    | In an empty group there is no older person

Look how we called the function ``Older`` with 2, 3, 4, 1 and even no input
parameters at all. We could have called it with 100 ``person``\s if we wanted
to.

Variadic functions are easy to write and can be handy in a lot of situations. By
the way, I promised to tell you about the built-in function called ``append``
that makes appending to slices easier! Now, I can tell you more, because, guess
what? The built-in ``append`` function is a variadic function!

The built-in append function
----------------------------
The append function has the following signature:

``func append(slice []T, elements...T) []T``.

It takes a slice of type ``[]T``, and as many other ``elements`` of the same
type ``T`` and returns a new slice of type ``[]T`` which includes the given
elements.

Example:

.. code-block:: go
    :linenos:

    package main
    import "fmt"

    func main(){
        s := []int {1, 2, 3}
        fmt.Println("At first: ")
        fmt.Println("s = ", s)
        fmt.Println("len(s) = ", len(s))
        fmt.Println("Let's append 4 to it")
        s = append(s, 4)
        fmt.Println("s = ", s)
        fmt.Println("len(s) = ", len(s))
        fmt.Println("Let's append 5 and 6 to it")
        s = append(s, 5, 6)
        fmt.Println("s = ", s)
        fmt.Println("len(s) = ", len(s))
        fmt.Println("Let's append 7, 8, and 9 to it")
        s = append(s, 7, 8, 9)
        fmt.Println("s = ", s)
        fmt.Println("len(s) = ", len(s))
    }

Output:

.. container:: output

    | At first:
    | s =  [1 2 3]
    | len(s) =  3
    | Let's append 4 to it
    | s =  [1 2 3 4]
    | len(s) =  4
    | Let's append 5 and 6 to it
    | s =  [1 2 3 4 5 6]
    | len(s) =  6
    | Let's append 7, 8, and 9 to it
    | s =  [1 2 3 4 5 6 7 8 9]
    | len(s) =  9

You can even give a variadic function a slice of type ``[]T`` instead of a list
of comma-separated parameters of type ``T``.

Example:

.. code-block:: go
    :linenos:

    //two slices of ints
    a := []int {1, 2, 3}
    b := []int {10, 11, 12}
    a = append(a, b...) // look at this syntax: the slice followed by three dots
    // a == {1, 2, 3, 10, 11, 12}

Again, just in case you glossed over the comment, notice the ellipses '``...``'
immediately following the second slice argument to the variadic function.

Using the append function to delete an element
----------------------------------------------
Suppose that we have a slice ``s`` and that we would like to delete an element
from it. There are 3 cases to consider:

* The element we'd like to delete is the first one of the slice.
* The element we'd like to delete is the last one of the slice
* The element we'd like to delete is the one at index ``i`` of the slice (where
  ``i`` is between the first and the last ones)

Let's write a function that deletes the element at a given index ``i`` in a
given slice of ``int``\s and see how the ``append`` function can help.

.. code-block:: go
    :linenos:

    package main
    import "fmt"

    func delete(i int, slice []int) []int{
        switch i {
            case 0: slice = slice[1:]
            case len(slice)-1: slice = slice[:len(slice)-1]
            default: slice = append(slice[:i], slice[i+1:]...)
        }
        return slice
    }

    func main(){
        s := []int {1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
        fmt.Println("In the beginning...")
        fmt.Println("s = ", s)
        fmt.Println("Let's delete the first element")
        s = delete(0, s)
        fmt.Println("s = ", s)
        fmt.Println("Let's delete the last element")
        s = delete(len(s)-1, s)
        fmt.Println("s = ", s)
        fmt.Println("Let's delete the 3rd element")
        s = delete(2, s)
        fmt.Println("s = ", s)
    }

Output:

.. container:: output

    | In the beginning...
    | s =  [1 2 3 4 5 6 7 8 9 10]
    | Let's delete the first element
    | s =  [2 3 4 5 6 7 8 9 10]
    | Let's delete the last element
    | s =  [2 3 4 5 6 7 8 9]
    | Let's delete the 3rd element
    | s =  [2 3 5 6 7 8 9]

The lines 6 and 7 of our program are fairly easy to understand. Aren't they?
We re-slice our slice omitting the first and the last elements respectively.

Now, the case where the element is not the first nor the last one. We assign to
our slice the result of ``append`` of the slice starting from the first up to,
but not including, the ``i`` :sup:`th` element and the slice that starts from
the element after the ``i`` :sup:`th` (i.e. i+1) up to the last one. That is, we
made our slice contain elements from the right sub-slice and the left one to the
element that we want to delete.

.. graphviz::

    digraph delete_from_slice {

        //rankdir=LR;
        graph [bgcolor=transparent, resolution=96, fontsize="10" ];
        edge [arrowsize=.5, arrowtail="dot", color="#FF6600"];
        node [fontsize=8, height=.1, penwidth=.4]
        s [shape=plaintext, label=<
        <table cellspacing="0" border="0" cellborder="1" cellpadding="8">
            <tr>
                <td bgcolor="lightpink" port="f0">2</td>
                <td bgcolor="lightpink">3</td>
                <td>4</td>
                <td bgcolor="lightblue" port="f1">5</td>
                <td bgcolor="lightblue">6</td>
                <td bgcolor="lightblue">7</td>
                <td bgcolor="lightblue">8</td>
                <td bgcolor="lightblue">9</td>
                <td border="0">s</td>
            </tr>
            <tr>
                <td border="0">0</td>
                <td border="0">1</td>
                <td border="0">2</td>
                <td border="0">3</td>
                <td border="0">4</td>
                <td border="0">5</td>
                <td border="0">6</td>
                <td border="0">7</td>
                <td border="0"></td>
            </tr>
        </table>>]
        s1 [shape=plaintext, label=<
        <table cellspacing="0" border="0" cellborder="1" cellpadding="8">
            <tr>
                <td bgcolor="lightpink" port="f0">2</td>
                <td bgcolor="lightpink">3</td>
                <td bgcolor="lightblue" port="f1">5</td>
                <td bgcolor="lightblue">6</td>
                <td bgcolor="lightblue">7</td>
                <td bgcolor="lightblue">8</td>
                <td bgcolor="lightblue">9</td>
                <td border="0">new s after deleting<br />the 3rd element</td>
            </tr>
            <tr>
                <td border="0">0</td>
                <td border="0">1</td>
                <td border="0">2</td>
                <td border="0">3</td>
                <td border="0">4</td>
                <td border="0">5</td>
                <td border="0">6</td>
                <td border="0"></td>
            </tr>
        </table>>]
        s1:f0:n->s:f0:s
        s1:f1:n->s:f1:s
        {rank=sink; s1}
    }

Simple, isn't it? Now, we actually complicated the ``delete`` function in vain.
We could have written it like this:

.. code-block:: go
    :linenos:

    //simpler delete
    func delete(i int, slice []int) []int{
        slice = append(slice[:i], slice[i+1:]...)
        return slice
    }

Yes, it will work. Go and try it. Can you figure out why?
(**Hint**: empty.  **Hint Hint**: empty.)

Recursive functions
===================
Some algorithmic problems can be thought of in a beautiful way using recursion!

A function is said to be *recursive* when it calls itself within it's own body.
For example:
the Max value in a slice is the maximum of the Max value of two sub-slices of
the same slice, one starting from 0 to the middle and the other starting from
the middle to the end of the slice.
To find out the Max in each sub-slice we use the same function,
since a sub-slice is itself a slice!

Enough talk already, let's see this wonderous beast in action!

.. code-block:: go
    :linenos:

    package main
    import "fmt"

    //Recursive Max
    func Max(slice []int) int{
        if len(slice) == 1 {
            return slice[0] //there's only one element in the slice, return it!
        }

        middle := len(slice)/2 //the middle index of the slice
        //find out the Max of each sub-slice
        m1 := Max(slice[:middle])
        m2 := Max(slice[middle:])
        //compare the Max of two sub-slices and return the bigger one.
        if m1 > m2 {
            return m1
        }
        return m2
    }

    func main(){
        s := []int {1, 2, 3, 4, 6, 8}
        fmt.Println("Max(s) = ", Max(s))
    }

Output:

.. container:: output

    | Max(s) =  8

Another example: How to invert the data in a given slice of ints?
For example, given ``s := []int {1, 2, 3, 4, 5}``, we want our program to modify
``s`` to be ``{5, 4, 3, 2, 1}``.

A recursive strategy to do this involves first swapping the first and the last
elements of ``s`` and then inverting the slice between these elements,
i.e. the one from the 2nd element to the one just before the last one.


.. graphviz::

    digraph slice_invert {

        //rankdir=LR;
        graph [bgcolor=transparent, resolution=96, fontsize="10" ];
        edge [arrowsize=.5, color="#FF6600"];
        node [fontsize=8, height=.1, penwidth=.4]
        a [shape=plaintext, label=<
        <table cellspacing="0" border="0" cellborder="1" cellpadding="8">
            <tr>
                <td border="0">step 1</td>
                <td port="f0">1</td>
                <td bgcolor="lightblue" port="f1">2</td>
                <td bgcolor="lightblue" port="f2">3</td>
                <td bgcolor="lightblue" port="f3">4</td>
                <td port="f4">5</td>
            </tr>
        </table>>]
        a:f0:n->a:f4:n [dir=both, label="swap", fontsize=8]
        a:f1:s->a:f3:s [dir=both, label="sub-slice", fontsize=8,
                        penwidth=2, color="lightblue"]

        b [shape=plaintext, label=<
        <table cellspacing="0" border="0" cellborder="1" cellpadding="8">
            <tr>
                <td border="0">step 2</td>
                <td port="f0">2</td>
                <td bgcolor="lightblue" port="f1">3</td>
                <td port="f2">4</td>
            </tr>
        </table>>]
        b:f0:n->b:f2:n [dir=both, label="swap", fontsize=8]
        b:f1:s->b:f1:s [dir=both, label="sub-slice", fontsize=8,
                        penwidth=2, color="lightblue"]

        c [shape=plaintext, label=<
        <table cellspacing="0" border="0" cellborder="1" cellpadding="8">
            <tr>
                <td border="0">step 3</td>
                <td port="f0">3</td>
            </tr>
        </table>>]
    }

Let's write it:

.. code-block:: go
    :linenos:

    package main
    import "fmt"

    func Invert(slice []int){
        l := len(slice)
        if l > 1 { //only a slice of 2 or more elements can be inverted
            slice[0], slice[l-1] = slice[l-1], slice[0] //swap first and last ones
            Invert(slice[1:l-1]) //invert the slice in between
        }
    }

    func main(){
        s := []int {1, 2, 3, 4, 5}
        fmt.Println("s = ", s)
        Invert(s)
        fmt.Println("Invert(s) = ", s)
    }

Output:

.. container:: output

    | s =  [1 2 3 4 5]
    | Invert(s) =  [5 4 2 3 1]

**Exercise** Recursive absurdity! The SUM of the ints in a given slice is equal
to the first element *plus* the SUM of the subslice starting from the second
element. Go and write this program.

And these are but a few examples, many problems can be solved using recursion.
You know, just like walking: to walk, you put one foot in front of the other and
you walk :)

The defer statement
===================
Sometimes you'd like to do *something* just before a ``return`` statement of a
function, like for example closing an opened file.
Now, suppose that our function has many spots where we call ``return``, like for
example in a ``if``/``else`` chain, or in a ``switch`` statement. In these cases
we'd do that *something* as many times as there is a ``return``.

The ``defer`` statement fixes this annoying irreverant mind-numbing repetition.

The ``defer`` statement schedules a function call (the deferred function) to be
run immediately before the function executing the ``defer`` returns.

Let's see an example to make the idea clearer:

The `os package`_ comes with functions to open, close, create... files, Go
figure!.
Being a good lil programmer means that we have to close every file we open in
our program after we've done our nefarious business with it's contents.

Now, imagine a function that opens a file, processes the data in it, but has
many return spots in its body. Without ``defer`` you'll have to *manually* close
any already opened file, just before the ``return``.

Look at this simple program from `Effective Go`_.

.. code-block:: go
    :linenos:

    package main

    import(
        "os"
        "fmt"
    )

    // Contents returns the file's contents as a string.
    func Contents(filename string) (string, os.Error) {
        f, err := os.Open(filename)
        if err != nil {
            return "", err
        }
        defer f.Close()  // f.Close will run when we're finished.

        var result []byte
        buf := make([]byte, 100)
        for {
            n, err := f.Read(buf[0:])
            result = append(result, buf[0:n]...) // append is discussed later.
            if err != nil {
                if err == os.EOF {
                    break
                }
                return "", err  // f will be closed if we return here.
            }
        }
        return string(result), nil // f will be closed if we return here.
    }

    func main(){
        c, _ := Contents("/etc/hosts")
        fmt.Println(c)
    }

Our function ``Contents`` needs a file name as its input, and it ouputs this
file's content, and an eventual error (for example in case the file was not
found).

On line 10, we open the file. If the returned error from ``os.Open`` is ``nil``
then we were unable to open the file. So we return an empty string and the same
error that ``os.Open`` returned.

Then on line 14, we ``defer`` the call to ``f.Close()`` so that the file will be
*automatically* closed when the function emits a return (2 spots in our function).

Then we declare a ``result`` slice that will contain all the bytes read from the
file. And we make a buffer ``buf`` of 100 bytes that will be used to read 100
bytes at a time from our file using the call to ``f.Read``. This buffer will be
appended to``result``  each time it's filled with ``f.Read``.

And we loop. If we get an error while reading, we check if its ``os.EOF`` (EOF
means End Of File) if so, we ``break`` from the loop, else it's another error so
we return an empty string and that error.

At the end, we return the slice ``result`` converted to a string, using a type
casting and ``nil`` which means that there was no error while retrieving the
contents of the file.

I know, this example may look a little bit hard, especially since we have never used
the ``os`` package before, but the main goal of it is to show you a use of the
``defer`` statement. And how it can be used to guarantee two things:

- Using ``defer f.Close()`` in the beginning makes sure that you will never
  forget to close the file, no matter how the function might change in the
  future, and no matter how many ``return`` spots will be added/removed from it.

- The close sits near the open, which is much clearer than placing it at the end
  of the function.

Multiple defer calls
--------------------
.. Just like procrastinating ;) 'defer' should be called 'procrastinate' :p
.. It likely wasn't due to the amount of typing involved ;) Plus we don't want
.. to be known as 'lazy programmers' :)
You can defer as many function calls as you want, and they'll be executed before
the containing function returns in a LIFO (Last In First Out) order. That is,
the last one to be deferred will be run first, and the first one will be executed
at last.

Example:

.. code-block:: go
    :linenos:

    package main
    import "fmt"

    func A(){
        fmt.Println("Running function A")
    }

    func B(){
        fmt.Println("Running function B")
    }

    func main(){
        defer A()
        defer B()
    }

Output:

.. container:: output

    | Running function B
    | Running function A


And that's it for this chapter. We learned how to write variadic, and recursive
functions, and how to defer function calls. The next chapter will be even more
interesting, we will see how functions are, actually, values!

.. external links and footnotes:

.. _os package: http://golang.org/pkg/os/index.html
.. _Effective Go: http://golang.com/doc/effective_go.html#defer
