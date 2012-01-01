.. include:: common.txt

Advanced composite types: Slices
********************************
We saw in the previous chapter how to combine some fields of different types in
a container called a ``struct`` to create a new type that we can use in our
functions and programs. We also saw how to gather **n** objects of the same type
into a single array that we can pass as a single value to a function.

This is fine, but do you remember :ref:`the note we wrote about
arrays<arrays-note>`?  Yes, that one about the fact that they don't grow or
shrink, that an array of 10 elements of a given type is totally different of an
array of, say,  9 elements of the same type.

This kind of *inflexibility* about arrays leads us to this chapter's subject.

What is a slice?
================
You'd love to have arrays without specifying sizes, right?
Well, that is possible in a manner of speaking, we call 'dynamic arrays' in Go *slices*.

A slice is not actually a 'dynamic array' but rather a **reference**
(think pointer) to a sub-array of a given array (anywhere from empty to all of it).
You can declare a slice just like you declare an array, omitting the size.

.. code-block:: go
    :linenos:

    //declare a slice of ints
    var array []int // Notice how we didn't specify a size.

As we see in the above snippet, we can declare a slice literal just like an
array literal, except that we leave out the size number.

.. code-block:: go
    :linenos:

    //declare a slice of ints
    slice := []byte {'a', 'b', 'c', 'd'} // A slice of bytes (current size is 4).

We can create slices by *slicing* an array or even an existing slice.
That is taking a portion (a 'slice') of it, using this syntax ``array[i:j]``.
Our created slice will contains elements of the array that start at index **i** and
end before **j**. (``array[j]`` not included in the slice)

.. code-block:: go
    :linenos:

    // Declare an array of 10 bytes (ASCII characters). Remember: byte is uint8.
    var array [10]byte {'a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j'}

    // Declare a and b as slice of bytes
    var a_slice,b_slice []byte

    // Make the slice "a_slice" refer to a "portion" of "array".
    a_slice = array[2:5]
    // a_slice is now a portion of array that contains: array[2], array[3] and array[4]

    // Make the slice "b_slice" refer to another "portion" of "array".
    b_slice = array[3:5]
    // b_slice is now a portion of array that contains: array[3], array[4]

.. graphviz::

    digraph slice_from_array {

        //rankdir=LR;
        graph [bgcolor=transparent, resolution=96, fontsize="10" ];
        edge [arrowsize=.5, arrowhead="dot", arrowtail="dot", color="#555555"];
        node [shape=record, fontsize=8, height=.1, penwidth=.4]
        ar[label="<f0>a|<f1>b|<f2>c|<f3>d|<f4>e|<f5>f|<f6>g|<f7>h|<f8>i|<f9>j"];
        a[shape=record, label="<f0>c|<f1>d|<f2>e", penwidth=1, color=crimson, fontcolor=crimson, style=filled, fillcolor=lightpink];
        b[shape=record, label="<f0>d|<f1>e", penwidth=1, color=steelblue4, fontcolor=steelblue4, style=filled, fillcolor=lightblue];
        edge [color=crimson];
        a:f0->ar:f2;
        a:f2->ar:f4;
        edge [color=steelblue4];
        b:f0->ar:f3;
        b:f1->ar:f4;
        //legende
        edge [arrowsize=.8, arrowhead="vee", color="#333333", penwidth=.8];
        array[label="ar", shape=circle, style=filled, color="#333333", fontcolor=white];
        sa[label="a", shape=circle, style=filled, color="#333333", fontcolor=white];
        sb[label="b", shape=circle, style=filled, color="#333333", fontcolor=white];
        array->ar;
        sa->a;
        sb->b;
        {rank=same; ar array}
        {rank=same; a sa}
        {rank=sink; b sb}
    }

Slice shorthands
----------------
- When slicing, first index defaults to **0**, thus ``ar[:n]`` means the same as ``ar[0:n]``.
- Second index defaults to len(array/slice), thus ``ar[n:]`` means the same as ``ar[n:len(ar)]``.
- To create a slice from an entire array, we use ``ar[:]`` which means the same as ``ar[0:len(ar)]`` due to the defaults mentioned above.

Ok, let's go over some more examples to make this even clearer.
Read slowly and pause to think about each line so that you fully grasp the
concepts.

.. code-block:: go
    :linenos:

    // Declare an array of 10 bytes (ASCII characters). Remember: byte is uint8
    var array [10]byte {'a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j'}
    //declare a and b as slice of bytes
    var a_slice,b_slice []byte

    // Examples of slicing
    a_slice = array[4:8]
    // means: a_slice contains elements of array from array[4] to array[7]: e,f,g,h
    a_slice = array[6:7]
    // means: a_slice contains ONE element of array and that is array[6]: g

    // Examples of shorthands
    a_slice = array[:3] // means: a_slice = array[0:3] thus: a contains: a,b,c
    a_slice = array[5:] // means: a_slice = array[5:9] thus: a contains: f,g,h,i,j
    a_slice = array[:] // means: a_slice = array[0:9] thus: a contains all array elements.

    // Slice of a slice
    a_slice = array[3:7] // means: a_slice contains elements form 3 to 6 of array: d,e,f,g
    b_slice = a_slice[1:3] //means: b_slice contains elements a[1], a[2] i.e the chars e,f
    b_slice = a_slice[:3] //means: b_slice contains elements a[0], a[1], a[2] i.e the chars: d,e,f
    b_slice : a_slice[:] //mens: b_slice contains all elements of slice a: d,e,f,g

Does it make more sense now? Fine!

*Is that all there is? What's so interesting about slices?*

Lets show you more examples below for the yummy ``slice``\d fruity goodness.

Slices and functions
====================
Say we need to find the biggest value in an array of 10 elements. Good, we know
how to do it. Now, imagine that our program has the task of finding the biggest
value in many arrays of different sizes! Aha! See? One function is not enough for
that, because, remember, the types: ``[n]int`` and ``[m]int`` are **different**!
They can not be used as an input of a single function.

Slices fix this quite easily! In fact, we need only write a function that
accepts a slice of integers as an input parameter, and create slices of arrays.

Let's see the code.

.. code-block:: go
    :linenos:

    package main
    import "fmt"

    // Return the biggest value in a slice of ints.
    func Max(slice []int) int{ // The input parameter is a slice of ints.
        max := slice[0] // The first element is the max for now.
        for index := 1; index < len(slice); index++ {
            if slice[index]>max{ // We found a bigger value in our slice.
                max = slice[index]
            }
        }
        return max
    }

    func main() {
        // Declare three arrays of different sizes, to test the function Max.
        A1 := [10]int {1,2,3,4,5,6,7,8,9}
        A2 := [4]int {1,2,3,4}
        A3 := [1]int {1}

        // Declare a slice of ints.
        var slice []int

        slice = A1[:] // Take all A1 elements.
        fmt.Println("The biggest value of A1 is", Max(slice))
        slice = A2[:] // Take all A2 elements.
        fmt.Println("The biggest value of A2 is", Max(slice))
        slice = A3[:] // Take all A3 elements.
        fmt.Println("The biggest value of A3 is", Max(slice))
    }

Output:

.. container:: output

    | The biggest value of A1 is 9
    | The biggest value of A2 is 4
    | The biggest value of A3 is 1

You see? Using a slice as the input parameter of our function made it
*reusable* and we didn't had to rewrite the same function for arrays of
different sizes. So remember this: whenever you think of writing a function that
takes an array as its input, think again, and use a *slice* instead.

Let's see other advantages of slices.

Slices are references
=====================
We stated earlier that slices are *references* to some underlying array (or
another slice).
What exactly does that mean?
It means that slicing an array or another slice doesn't simply copy some of the
array's elements into the new slice, it instead make the new slice *point* to
the element of this sliced array similar to the way pointers operate.

In other words: changing an element's value in a slice will actually change the
value of the element in the underlying array to which the slice points.
This changed value will be visible in all slices and slices of slices containing
the array element.

Let's see an example:

.. code-block:: go
    :linenos:

    package main
    import "fmt"

    func PrintByteSlice(name string, slice []byte){
        fmt.Printf("%s is : [", name)
        for index :=0; index < len(slice)-1; index ++{
            fmt.Printf("%q,", slice[i])
        }
        fmt.Printf("%q]\n", slice[len(slice)-1])
    }

    func main(){
        // Declare an array of 10 bytes.
        A := [10]byte {'a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j'}

        //declare some slices.
        slice1 := A[3:7] // Slice1 == {'d', 'e', 'f', 'g'}
        slice2 := A[5:] // Slice2 == {'f', 'g', 'h', 'i', 'j'}
        slice3 := slice1[:2] // Slice3 == {'d', 'e'}

        // Let's print the current content of A and the slices.
        fmt.Println("\nFirst content of A and the slices")
        PrintByteSlice("A", A[:])
        PrintByteSlice("slice1", slice1)
        PrintByteSlice("slice2", slice2)
        PrintByteSlice("slice3", slice3)

        // Let's change the 'e' in A to 'E'.
        A[4] = 'E'
        fmt.Println("\nContent of A and the slices, after changing 'e' to 'E' in array A")
        PrintByteSlice("A", A[:])
        PrintByteSlice("slice1", slice1)
        PrintByteSlice("slice2", slice2)
        PrintByteSlice("slice3", slice3)

        // Let's change the 'g' in slice2 to 'G'.
        slice2[1] = 'G' // Remember that 1 is actually the 2nd element in slice2!
        fmt.Println("\nContent of A and the slices, after changing 'g' to 'G' in slice2")
        PrintByteSlice("A", A[:])
        PrintByteSlice("slice1", slice1)
        PrintByteSlice("slice2", slice2)
        PrintByteSlice("slice3", slice3)
    }

Output:

.. container:: output

    | First content of A and the slices
    | A is : ['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j']
    | slice1 is : ['d', 'e', 'f', 'g']
    | slice2 is : ['f', 'g', 'h', 'i', 'j']
    | slice3 is : ['d', 'e']

    | Content of A and the slices, after changing 'e' to 'E' in array A
    | A is : ['a', 'b', 'c', 'd', 'E', 'f', 'g', 'h', 'i', 'j']
    | slice1 is : ['d', 'E', 'f', 'g']
    | slice2 is : ['f', 'g', 'h', 'i', 'j']
    | slice3 is : ['d', 'E']

    | Content of A and the slices, after changing 'g' to 'G' in slice2
    | A is : ['a', 'b', 'c', 'd', 'E', 'f', 'G', 'h', 'i', 'j']
    | slice1 is : ['d', 'E', 'f', 'G']
    | slice2 is : ['f', 'G', 'h', 'i', 'j']
    | slice3 is : ['d', 'E']


Let's talk about the ``PrintByteSlice`` function a little bit. If you analyze
its code, you'll see that it loops until before ``len(slice)-1`` to print the
element ``slice[i]`` followed by a comma. At the end of the loop, it prints
``slice[len(slice)-1]`` (the last item) with a closing bracket instead of
a comma.

``PrintByteSlice`` works with slices of bytes, of any size, and can be used to
print arrays content also, by using a slice that contains all of its elements as
an input parameter. This proves the utility of the advice before: always think
of *slices* when you're about to write functions *for arrays*.

Until now, we've learned that slices are truly useful as input parameters for
functions which expect *blocks* of a variable size of consecutive elements of
the same type. We have learned also that slices are references to portions of
an array or another slice.

That is not all. Here's an awesome feature of slices.

Slices are resizable
====================
One of our concerns with arrays is that they're inflexible. Once you declare an
array, its size can't change. It won't shrink nor grow. How unfortunate!

Slices can grow or shrink.

How so? Conceptually, a *slice* is a ``struct`` of three fields:

* A *pointer* to the first element of the array *where the slice begins*.
* A *length* which is an **int** representing the total number of *elements in the slice*.
* A *capacity* which is an **int** representing the number of *available elements*.

.. code-block:: go
    :linenos:

    type Slice struct {
        base *elementType //the pointer
        len int //length
        cap int //capacity
    }

Schematically, since you *love* my diagrams.

.. graphviz::

    digraph slice_internal {

        //rankdir=LR;
        graph [bgcolor=transparent, resolution=96, fontsize="10" ];
        edge [arrowsize=.5, arrowtail="dot", color="#555555"];
        node [fontsize=8, height=.1, penwidth=.4]
        a [shape=plaintext, label=<
        <table cellspacing="0" border="0" cellborder="1" cellpadding="8">
            <tr>
                <td>a</td>
                <td>b</td>
                <td>c</td>
                <td>d</td>
                <td bgcolor="lightblue" port="f4">e</td>
                <td bgcolor="lightblue">f</td>
                <td bgcolor="lightblue">g</td>
                <td bgcolor="lightblue">h</td>
                <td bgcolor="steelblue">i</td>
                <td bgcolor="steelblue">j</td>
                <td border="0">A [10]byte</td>
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
                <td border="0">8</td>
                <td border="0">9</td>
                <td border="0"></td>
            </tr>
        </table>>]
        s [shape=plaintext, label=<
        <table cellspacing="0" border="0" cellborder="1" cellpadding="8">
            <tr>
                <td port="f0">base</td>
                <td>len==4</td>
                <td>cap==6</td>
                <td border="0" ALIGN="LEFT">s := A[4:8]<br/>s == {'e', 'f', 'g', 'h'}</td>
            </tr>
        </table>>]
        s:f0:s->a:f4:n
        {rank=sink; a}
    }


.. code-block:: go
    :linenos:

    // Declare an array of 10 bytes.
    array := [10]byte {'a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j'}
    slice := A[4:8]

The picture above is a representation of a slice that starts from the 5th
element of the array. It contains exactly 4 elements (its length) and can
contain up to 6 elements on the same underlying array (its capacity) starting
from the first element of the slice.

.. admonition:: Philosophy

    Simply said: Slicing does not make a copy of the elements, it just creates a
    new structure holding a different pointer, length, and capacity.

.. _function-make:

To make things even easier, Go comes with a built-in function called make, which
has the signature: ``func make([]T, len, cap) []T``.

Where ``T`` stands for the element type of the slice to be created. The make
function takes a *type*, a *length*, and an *optional capacity*.
When called, make allocates an array and returns a slice that refers to that
array.

The optional capacity parameter, when omitted, defaults to the specified length.
These parameters can be inspected for a given slice, using the built-in
functions:
- ``len(slice)`` which returns a slice's length, and
- ``cap(slice)`` which returns a slice's capacity.

Example:

.. code-block:: go
    :linenos:

    var slice1, slice2 []int
    slice1 = make([]int, 4, 4) // slice1 is []int {0, 0, 0, 0}
    slice2 = make([]int``,`` 4) // slice2 is []int {0, 0, 0, 0}
    // Note: cap == len == 4 for slice2
    // cap(slice2) == len(slice2) == 4

The zero value of a slice is ``nil``.
The ``len`` and ``cap`` functions will both return **0** for a ``nil`` slice.

.. code-block:: go
    :linenos:

    package main

    import "fmt"

    func main() {
        var slice []int
        fmt.Println("Before calling make")
        if slice==nil {fmt.Println("slice == nil")}
        fmt.Println("len(slice) == ", len(slice))
        fmt.Println("cap(slice) == ", cap(slice))

        // Let's allocate the underlying array:
        fmt.Println("After calling make")
        slice = make([]int, 4)
        fmt.Println("slice == ", slice)
        fmt.Println("len(slice) == ", len(slice))
        fmt.Println("cap(slice) == ", cap(slice))

        // Let's change things:
        fmt.Println("Let's change some of its elements: slice[1], slice[3] = 2, 3")
        slice[1], slice[3] = 2, 3
        fmt.Println("slice == ", slice)
    }

Output:

.. container:: output

    | Before calling make
    | slice == nil
    | len(slice) ==  0
    | cap(slice) ==  0
    | After calling make
    | slice ==  [0 0 0 0]
    | len(slice) ==  4
    | cap(slice) ==  4
    | Let's change some of its elements: slice[1], slice[3] = 2, 3
    | slice ==  [0 2 0 3]

We may find ourselves wondering "Why use ``make`` instead of ``new`` that we discussed
when we studied :ref:`pointers<function-new>`?"

Let's discuss the reason why we use ``make`` and not ``new`` to create a new
slice:

Recall that ``new(T)`` allocates memory to store a variable of type ``T`` and
returns a pointer to it. Thus ``new(T)`` returns ``*T``, right?

So for example if we had used ``new([]int)`` to create a slice of integers,
``new`` will allocate the internal structure discussed above and that will
consist of a pointer, and two integers (length and capacity).
``new`` will then return us a pointer of the type ``*[]int`` -- which...
let's face it... is not what we *need*!

What we do *need* is a function which:

- Returns the actual structure not a pointer to it. i.e. returns ``[]int`` not
  ``*[]int``.
- Allocates the underlying array that will hold the slice's elements.

Do you see the difference? ``make`` doesn't merely allocate enough space to
*host* the structure of a given slice (the pointer, the length and the capacity)
only, it allocates both the structure *and* the underlying array.

Back to our resizability of slices. As you may guess, shrinking a slice is
really easy! All we have to do is reslice it and assign the new slice to it.

.. insert example of shrinking a slice here -- Shrinkage!!! Cold water Example!!!

Now, let's see how -- using only what we have learned until now -- we grow
a slice. A very simple, straight forward way of growing a slice might be:

1. make a new slice
2. copy the elements of our slice to the one we created in 1
3. make our slice refer to the slice created in 1

.. code-block:: go
    :linenos:
    :emphasize-lines: 16, 17, 18

    package main
    import "fmt"

    func PrintIntSlice(name string, slice []int){
        fmt.Printf("%s == [", name)
        for index := 0; index < len(slice)-1; index++ {
            fmt.Printf("%d,", slice[index])
        }
        fmt.Printf("%d]\n", slice[len(slice)-1])
    }


    func GrowIntSlice(slice []int, add int) []int {
        new_capacity := cap(slice)+add
        new_slice := make([]int, len(slice), new_capacity)
        for index := 0; index < len(slice); index++ {
            new_slice[index] = slice[index]
        }
        return new_slice
    }

    func main(){
        slice := []int {0, 1, 2, 3}

        fmt.Println("Before calling GrowIntSlice")
        PrintIntSlice("slice", slice)
        fmt.Println("len(slice) == ", len(slice))
        fmt.Println("cap(slice) == ", cap(slice))

        // Let's call GrowIntSlice
        slice = GrowIntSlice(slice, 3) //add 3 elements to the slice

        fmt.Println("After calling GrowIntSlice")
        PrintIntSlice("slice", slice)
        fmt.Println("len(slice) == ", len(slice))
        fmt.Println("cap(slice) == ", cap(slice))

        // Let's two elements to the slice
        // So we reslice the slice to add 2 to its original length
        slice = slice[:len(slice)+2] // We can do this because cap(slice) == 7
        slice[4], slice[5] = 4, 5
       	fmt.Println("After adding new elements: slice[4], slice[5] = 4, 5")
        PrintIntSlice("slice", slice)
        fmt.Println("len(slice) == ", len(slice))
        fmt.Println("cap(slice) == ", cap(slice))
    }

Output:

.. container:: output

    | Before calling GrowIntSlice
    | slice == [0,1,2,3]
    | len(slice) ==  4
    | cap(slice) ==  4
    | After calling GrowIntSlice
    | slice == [0,1,2,3]
    | len(slice) ==  4
    | cap(slice) ==  7
    | After adding new elements: slice[4], slice[5] = 4, 5
    | slice == [0,1,2,3,4,5]
    | len(slice) ==  6
    | cap(slice) ==  7

Let's discuss this program a little bit.

First, the ``PrintIntSlice`` function is similar to the ``PrintByteSlice``
function we saw earlier. Except that we improved it to support ``nil`` slices.

Then the most important one is ``GrowIntSlice`` which takes two input
parameters: a ``[]int`` slice and an ``int`` ``add`` that represents the amount
of elements to add to the capacity.

GrowIntSlice first ``make``\s a new slice with a ``new_capacity`` that is equal
the original slice's capacity plus the ``add`` parameter.

Then it copies elements from ``slice`` to ``new_slice`` in the highlighted loop
in lines 16, 17 and 18.

And finally, it returns the ``new_slice`` as an output.

The approach you may be used to or that first comes to mind is often
implemented as a built-in Go function provided for you already.
This is most definitely one such example. There is a built-in Go function
called ``copy`` which takes two arguments: source and destination, and copies
elements from the source to the destination and returns the number of elements
copied.

The signature of ``copy`` is: ``func copy(dst, src []T) int``

So we can replace the lines 16, 17, 18 with this very simple one:

.. code-block:: go
    :linenos:

    // Instead of lines 16, 17, 18 of the previous source:
    copy(new_slice, slice) // Copy elements from slice to new_slice

You know what? Let's write another example and use the ``copy`` function in it.

.. code-block:: go
    :linenos:

    package main
    import "fmt"

    func PrintByteSlice(name string, slice []byte){
        fmt.Printf("%s is : [", name)
        for index := 0; index < len(slice)-1; index++ {
            fmt.Printf("%q,", slice[index])
        }
        fmt.Printf("%q]\n", slice[len(slice)-1])
    }

    func Append(slice, data[]byte) []byte {
        length := len(slice)
        if length + len(data) > cap(slice) {  // Reallocate
            // Allocate enough space to hold both slice and data
            newSlice := make([]byte, l+len(data))
            // The copy function is predeclared and works for any slice type.
            copy(newSlice, slice)
            slice = newSlice
        }
        slice = slice[0:length+len(data)]

        copy(slice[length:length+len(data)], data)
        return slice
    }

    func main() {
        hello := []byte {'H', 'e', 'l', 'l', 'o'}
        world := []byte {' ', 'W', 'o', 'r', 'l', 'd'}
        fmt.Println("Before calling Append")
        PrintByteSlice("hello", hello)
        PrintByteSlice("world", world)

        fmt.Println("After calling Append")
        Append(hello, world)
        PrintByteSlice("hello", hello)
        PrintByteSlice("world", world)
    }

Output:

.. container:: output

    | Before calling Append
    | hello is : ['H','e','l','l','o']
    | world is : ['W','o','r','l','d']
    | After calling Append
    | hello is : ['H','e','l','l','o','W','o','r','l','d']
    | world is : ['W','o','r','l','d']

Appending to a slice is a very common operation and as such Go has a built-in
function called ``append`` that we will see in details very soon.

Phew, that was a long chapter, eh? We'll stop here for now, go out, have a nice
day, and see you tomorrow for another interesting composite type. If you are
completely enthralled thus far by this exquisite book then do yourself a favor
before continuing and go get a tea or coffee and take a nice 10 minute break.

