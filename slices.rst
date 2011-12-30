.. include:: common.txt

Advanced composite types: Slices
********************************
We saw in the previous chapter how to combine some fields of different types in
a container called a ``struct`` to create a new type that we can use in our
functions and programs. We also saw how to gather *n* objects of the same type
in a single array that we can pass as a single value to a function.

This is fine, but do you remember :ref:`the note we wrote about
arrays<arrays-note>`?  Yes, that one about the fact that they don't grow or
shrink, that an array of 10 elements of a given type is totally different of an
array of, say,  9 elements of the same type.

This kind of *inflexibility* about arrays leads us to this chapter's subject.

What is a slice?
================
You'd love to have arrays without specifying sizes, right? Well, that's kind of
possible, but they're not arrays but slices.

A slice is a **reference** to a section of an array. You can declare a slice
like you used to declare an array, but without the size.

.. code-block:: go
    :linenos:

    //declare a slice of ints
    var a []int //notice how we didn't specify a size.

we can declare a slice literal like an array literal, except we leave out the
size.

.. code-block:: go
    :linenos:

    //declare a slice of ints
    s := []byte {'a', 'b', 'c', 'd'} //a slice of bytes (current size is 4)

We can create slices by *slicing* an array or slice. That is taking a portion
(a slice) of it, using this syntax array[i:j].
Our created slice will contains elements of the array that start at index i and
end before j. (array[j] not included in the slice)

.. code-block:: go
    :linenos:

    //declare an array of 10 bytes (ASCII characters). Remember: byte is uint8
    var ar [10]byte {'a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j'}
    //declare a and b as slice of bytes
    var a,b []byte 
    //make the slice "a" refer to a "portion" of "ar".
    a = ar[2:5] //a is a portion of ar that contains: ar[2], ar[3] and ar[4]
    //make the slice "b" refer to another "portion" of "ar".
    b = ar[3:5] //b is a portion of ar that contains: ar[3], ar[4]

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
- When slicing, first index defaults to 0: ar[:n] means the same as ar[0:n].
- Second index defaults to len(array/slice): ar[n:] means the same as ar[n:len(ar)].
- Thus to create a slice from an array: ar[:] means the same as ar[0:len(ar)].

Ok, some more examples to make it even clearer:

.. code-block:: go
    :linenos:

    //declare an array of 10 bytes (ASCII characters). Remember: byte is uint8
    var ar [10]byte {'a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j'}
    //declare a and b as slice of bytes
    var a,b []byte 
    //examples of slicing
    a = ar[4:8] // means: a contains elements of ar from ar[4] to ar[7]: e,f,g,h
    a = ar[6:7] // means: a contains ONE element of ar and that is ar[6]: g
    //examples of shorthands
    a = ar[:3] // means: a = ar[0:3] thus: a contains: a,b,c
    a = ar[5:] // means: a = ar[5:9] thus: a contains: f,g,h,i,j
    a = ar[:] //means: a = ar[0:9] thus: a contains all ar elements.
    //slice of a slice
    a = ar[3:7] // means: a contains elements form 3 to 6 of ar: d,e,f,g
    b = a[1:3] //means: b contains elements a[1], a[2] i.e the chars e,f
    b = a[:3] //means: b contains elements a[0], a[1], a[2] i.e the chars: d,e,f
    b : a[:] //mens: b contains all elements of slice a: d,e,f,g

Does it make more sense now? Fine!

*Is that all there is? What's so interesting about slices?* Let me show you an
example. 

Slices and functions
====================
Say we need to find the biggest value in an array of 10 elements. Good, we know
how to do it. Now, imagine that our program has the task of finding the biggest
value in many arrays of different sizes! Aha! See? One function is not enough for
that, because, remember, the types: ``[n]int`` and ``[m]int`` are **different**!
They can not be used as an input of a single function.

Slices fix this so easily! In fact, you just have to write a function that
accepts a slice of ints as an input parameter, and create slices of your arrays.
Let's see the code.

.. code-block:: go
    :linenos:

    package main
    import "fmt"

    //return the biggest value in a slice of ints
    func Max(s []int) int{ //the input parameter is a slice of ints
        max := s[0] //the first element is the max for now
        for i:=1; i<len(s); i++{ 
            if s[i]>max{ //we found a bigger value in our slice
                max = s[i] 
            }
        }
        return max
    }

    func main(){
        //declare three arrays of different sizes, to test the function Max
        A1 := [10]int {1,2,3,4,5,6,7,8,9}
        A2 := [4]int {1,2,3,4}
        A3 := [1]int {1}

        //declare a slice of ints
        var slice []int

        slice = A1[:] //take all A1 elements
        fmt.Println("The biggest value of A1 is", Max(slice))
        slice = A2[:] //take all A2 elements
        fmt.Println("The biggest value of A2 is", Max(slice))
        slice = A3[:] //take all A3 elements
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
takes an array as its input, think again, and use a slice instead.

Let's see other advantages of slices.

Slices are references
=====================
So we said that slices are *references* to some underlying array (or slice).
What does that mean? It means that slicing an array or another slice doesn't
copy some of its elements into the new slice, it just make the new slice *point*
to the element of this sliced array.

In other words: changing an element's value in a slice will actually change the
value of the element in the underlying array. And this change will be visible in
all the slices, and slices of slices... referring to this array.

Let's see an example:

.. code-block:: go
    :linenos:

    package main
    import "fmt"

    func PrintByteSlice(name string, slice []byte){
        fmt.Printf("%s is : [", name)
        for i:=0; i<len(slice)-1; i++{
            fmt.Printf("%q,", slice[i]) 
        } 
        fmt.Printf("%q]\n", slice[len(slice)-1])
    }

    func main(){
        //declare an array of 10 bytes.
        A := [10]byte {'a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j'}

        //declare some slices
        slice1 := A[3:7] // slice1 == {'d', 'e', 'f', 'g'}
        slice2 := A[5:] //slice2 == {'f', 'g', 'h', 'i', 'j'}
        slice3 := slice1[:2] //slice3 == {'d', 'e'}

        //let's print the current content of A and the slices
        fmt.Println("\nFirst content of A and the slices")
        PrintByteSlice("A", A[:])
        PrintByteSlice("slice1", slice1)
        PrintByteSlice("slice2", slice2)
        PrintByteSlice("slice3", slice3)

        //let's change the 'e' in A to 'E'
        A[4] = 'E'
        fmt.Println("\nContent of A and the slices, after changing 'e' to 'E' in array A")
        PrintByteSlice("A", A[:])
        PrintByteSlice("slice1", slice1)
        PrintByteSlice("slice2", slice2)
        PrintByteSlice("slice3", slice3)

        //Let's change the 'g' in slice2 to 'G'
        slice2[1] = 'G' //it's the 2nd element in slice2!
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
element ``slice[i]`` and follow it with a comma. And at the end of the loop, it
prints ``slice[len(slice)-1]`` (the last item) with a closing bracket.

``PrintByteSlice`` works with slices of bytes, of any size, and can be used to
print arrays content too, by using a slice that contains all of its elements as
an input parameter. This proves the utility of the advice before: always think
of slices when you're about to write functions for arrays.

So until now, we've learned that slices are really neat as input parameters for
functions that expect *blocks* of variable size of consecutive elements of the
same type. And that slices are references. 

That is not all. Here's an awesome feature of slices.

Slices are resizable
====================
One of our concerns with arrays is that they're inflexible. Once you declare an
array, its size can't change. It won't shrink nor grow. How unfortunate!

Slices can grow or shrink.

How so? Conceptually, a slice is struct of three fields:

* A pointer to the first element of the array where the slice begins.
* A length: an int that represents the number of elements in the slice.
* A Capacity: an int that represents the number of available elements.

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

    //declare an array of 10 bytes.
    A := [10]byte {'a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j'}
    s := A[4:8]
    
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

Where T stands for the element type of the slice to be created. The make
function takes a type, a length, and an optional capacity. When called, make
allocates an array and returns a slice that refers to that array.

The optional capacity parameter, when omitted, defaults to the specified length.
These parameters can be inspected for a given slice, using the built-in
functions: ``len(slice)`` which returns the slice's length, and ``cap(slice)``
which returns its capacity.

Example:

.. code-block:: go
    :linenos:
    
    var slice1, slice2 []int
    slice1 = make([]int, 4, 4)
    //so slice1 == []int {0, 0, 0, 0}
    slice2 = make([]int``,`` 4) //cap == len == 4
    //so slice2 == []int {0, 0, 0, 0}
    //cap(slice2) == len(slice2) == 4

The zero value of a slice is ``nil``. The ``len`` and ``cap`` functions will
both return 0 for a nil slice.

.. code-block:: go
    :linenos:
    
    package main

    import "fmt"

    func main() {
        var s []int
        fmt.Println("Before calling make")
        if s==nil {fmt.Println("s == nil")}
        fmt.Println("len(s) == ", len(s))
        fmt.Println("cap(s) == ", cap(s))
        //let's allocate the underlying array
        fmt.Println("After calling make")
        s = make([]int, 4)
        fmt.Println("s == ", s)
        fmt.Println("len(s) == ", len(s))
        fmt.Println("cap(s) == ", cap(s))
        //let's change things
        fmt.Println("Let's change some of its elements: s[1], s[3] = 2, 3")
        s[1], s[3] = 2, 3
        fmt.Println("s == ", s)
    }

Output:

.. container:: output

    | Before calling make
    | s == nil
    | len(s) ==  0
    | cap(s) ==  0
    | After calling make
    | s ==  [0 0 0 0]
    | len(s) ==  4
    | cap(s) ==  4
    | Let's change some of its elements: s[1], s[3] = 2, 3
    | s ==  [0 2 0 3]

Smart will already ask: "Why use ``make`` instead of ``new`` that we discussed
when we studied :ref:`pointers<function-new>`?" -- And smarter ones already know
why :)

Let's say you're smart. Here's the reason why we use ``make`` and not ``new`` to
create a new slice:

Rememeber? ``new(T)`` allocates memory to store a variable of type ``T`` and
returns a pointer to it. Thus ``new(T)`` returns ``*T``. Right?

So if we had used ``new([]int)`` for example to create a slice of ints, this
will just allocate the internal structure discussed above and that will consist
of a pointer, and two ints (length and capacity). And will return us a pointer
of the type ``*[]int`` --and... that's not what we *need*!

What we do need is a function that:

- Returns the actual structure not a pointer to it. i.e. returns ``[]int`` not
  ``*[]int``.  
- Allocate the underlying array that will hold the slice's elements.

Do you see the difference? ``make`` doesn't allocate enough space to *host* a
structure of a slice (the pointer, the length and the capacity) only, it
allocates the structure *and* the underlying array.

Back to our resizability of slices. As you may guess, shrinking a slice is
really easy! All we have to do is reslice it and assign the new slice to it. 

.. insert example of shrinking a slice here

Now, let's see how -using only what we've learnt until now- we can make a slice
grow. A very simple way of doing this is:

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
        for i:=0; i<len(slice)-1; i++{
            fmt.Printf("%d,", slice[i]) 
        } 
        fmt.Printf("%d]\n", slice[len(slice)-1])
    }


    func GrowIntSlice(slice []int, add int) []int{
        new_capacity := cap(slice)+add
        new_slice := make([]int, len(slice), new_capacity)
        for i:=0; i<len(slice); i++{
            new_slice[i] = slice[i]
        }
        return new_slice
    }

    func main(){
        s := []int {0, 1, 2, 3}

        fmt.Println("Before calling GrowIntSlice")
        PrintIntSlice("s", s)
        fmt.Println("len(s) == ", len(s))
        fmt.Println("cap(s) == ", cap(s))

        //let's call GrowIntSlice
        s = GrowIntSlice(s, 3) //add 3 elements to the slice

        fmt.Println("After calling GrowIntSlice")
        PrintIntSlice("s", s)
        fmt.Println("len(s) == ", len(s))
        fmt.Println("cap(s) == ", cap(s))
	
        //Let's two elements to the slice
        //so we reslice the slice to add 2 to its original length
        s = s[:len(s)+2] //We can do this because cap(s) == 7
        s[4], s[5] = 4, 5	
       	fmt.Println("After adding new elements: s[4], s[5] = 4, 5")
        PrintIntSlice("s", s)
        fmt.Println("len(s) == ", len(s))
        fmt.Println("cap(s) == ", cap(s))
    }

Output:

.. container:: output

    | Before calling GrowIntSlice
    | s == [0,1,2,3]
    | len(s) ==  4
    | cap(s) ==  4
    | After calling GrowIntSlice
    | s == [0,1,2,3]
    | len(s) ==  4
    | cap(s) ==  7
    | After adding new elements: s[4], s[5] = 4, 5
    | s == [0,1,2,3,4,5]
    | len(s) ==  6
    | cap(s) ==  7

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

We could have totally removed this loop actually with a built-in Go function
called ``copy`` that takes two arguments: source and destination, and copies
elements from source to destination and returns the number of the copied
elements.

Its signature is: ``func copy(dst, src []T) int``

So we can replace the lines 16, 17, 18 with this very simple one:

.. code-block:: go
    :linenos:
    
    //instead of lines 16, 17, 18 of the previous source:
    copy(new_slice, slice) //copy elements from slice to new_slice

Know what? Let's write another example and use the ``copy`` function in it.

.. code-block:: go
    :linenos:
    
    package main
    import "fmt"

    func PrintByteSlice(name string, slice []byte){
        fmt.Printf("%s is : [", name)
        for i:=0; i<len(slice)-1; i++{
            fmt.Printf("%q,", slice[i]) 
        } 
        fmt.Printf("%q]\n", slice[len(slice)-1])
    }

    func Append(slice, data[]byte) []byte {
        l := len(slice)
        if l + len(data) > cap(slice) {  // reallocate
            // Allocate enough space to hold both slice and data
            newSlice := make([]byte, l+len(data))
            // The copy function is predeclared and works for any slice type.
            copy(newSlice, slice)
            slice = newSlice
        }
        slice = slice[0:l+len(data)]

        copy(slice[l:l+len(data)], data)
        return slice
    }

    func main(){

        h := []byte {'H', 'e', 'l', 'l', 'o'}
        w := []byte {' ', 'W', 'o', 'r', 'l', 'd'}
        fmt.Println("Before calling Append")
        PrintByteSlice("h", h)
        PrintByteSlice("w", w)

        fmt.Println("After calling Append")
        Append(h, w)
        PrintByteSlice("h", h)
        PrintByteSlice("w", w)
    }

Output:

.. container:: output

    | Before calling Append
    | h is : ['H','e','l','l','o']
    | w is : ['W','o','r','l','d']
    | After calling Append
    | h is : ['H','e','l','l','o','W','o','r','l','d']
    | w is : ['W','o','r','l','d']

Appending to a slice is such a common operation that Go has a built-in function
called ``append`` that we will see in details very soon.

That was a long chapter, wasn't it? We'll stop here for now, go out, have a nice
day, and see you tomorrow for another interesting composite type.
