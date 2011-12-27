.. include:: common.txt

.. _composite-types:

Basic composite types
*********************
In the :doc:`previous chapter<functions>`, we saw how to combine the traditional
control flow idioms seen in :doc:`chapter 3<control>` to create blocks of
reusable code, called *functions*.  In this chapter, we'll see how to combine
the basic data types seen in :ref:`chapter 2<primitive-types>`, in order to
create complex data types.

The problem
===========
Go comes with some *primitive* types: ints, floats, complex numbers, booleans,
and strings. Fair enough, but you'll notice as you start to write complex
programs that you'll need more advanced forms of data.

Here's an example:

So in the previous chapter, we wrote the ``MAX(A, B int)`` function to find out
the bigger value between two integers A, and B. Now, suppose that A, and B are
actually the ages of persons whose names are respectively Tom and Bob. We need
our program to output the name of the older person, and by how many years he is
older.

Sure, we still can use the ``MAX(A, B int)`` function by giving it as input
parameters, the ages of Tom and Bob. But it'd be nicer to write a function
``Older`` for real **people**. And not for integers.

*People*! I said people! Thus the need to *create* a custom type to *represent*
people, and this type will be used for the input parameters of the ``Older``
function we'd like to write.

Structs
=======
In Go, as in C and other languages, we can declare new types that act as
*containers* for attributes or fields of some other types. For example, we can
create a custom type called ``person`` to represent a real-world person, and
that will contain the person's *name* **and** his *age*.

.. graphviz::

    digraph person {
        rankdir=LR;
        graph [bgcolor=transparent, resolution=96, fontsize="10" ];
        edge [arrowsize=.5, arrowtail="dot", color="#FF6600"];
        node [shape=record, fontsize=8, height=.1, penwidth=.4]
        person[label="<f0>Tom|<f1>25"];
        node[shape=plaintext];
        pName [label="P.name"];
        pAge [label="P.age"];
        pName->person:f0;
        pAge->person:f1;
    }


A type like this is called a ``struct`` (a structure)

How to declare a struct in Go?
------------------------------
.. code-block:: go
    :linenos:

    type person struct{
        name string //field name of type string.
        age int //field age of type int.
    }

See? It's easy. A person ``struct`` is a container of two fields: 

- The person's name: a field of type ``string``.
- The person's age: a field of type ``int``.

And we declare a ``person`` variable as we used to in :doc:`chapter 2<basic>`.

Thus, writing things like:

.. code-block:: go
    :linenos:

    type person struct{
        name string
        age int
    }

    var P person // tom is a variable of type person

You access and assign a struct's field with the *dot nottation*. That is, if
``P`` is a variable of type ``person``, then you can access its fields like
this:

.. code-block:: go
    :linenos:

    P.name = "Tom" //assign "Tom" to P's name field.
    P.age = 25 //set P's age to 25 (an int)
    fmt.Println("The person's name is %s", P.name) //Access P's name field.

There's even two short assignment statement syntaxes:

1. By providing the values for each (and every) field in order:

.. code-block:: go
    :linenos:

    //declare a variable P of type person and initialize its fields
    //name to "Tom" and age to 25.
    P := person{"Tom", 25}

2. By using field:value initializers for any number of fields in any order:

.. code-block:: go
    :linenos:

    //declare a variable P of type person and initialize its fields
    //name to "Tom" and age to 25.
    //Order, here, doesn't count since we specify the fields.
    P := person{age:24, name:"Tom"}


Good. Let's write our ``Older`` function that takes two input parameters of type
``person``, and returns the older one *and* the difference in their ages.

.. code-block:: go
    :linenos:

    package main
    import "fmt"

    // We declare our new type
    type person struct{
        name string
        age int
    }


    //return the older person of p1 and p2
    //and the difference in their ages
    func Older(p1, p2 person) (person, int){
        if p1.age>p2.age { //compare p1 and p2's ages
            return p1, p1.age-p2.age
        }
        return p2, p2.age-p1.age
    }

    func main(){
        var tom person

        tom.name, tom.age = "Tom", 18

        //Look how to declare and initialize easily
        bob := person{age:25, name:"Bob"} //specify the fields and their values
        paul := person{"Paul", 43} //specify values of fields in their order

        tb_Older, tb_diff := Older(tom, bob)
        tp_Older, tp_diff := Older(tom, paul)
        bp_Older, bp_diff := Older(bob, paul)

        fmt.Printf("Of %s and %s, %s is older by %d years\n", 
                    tom.name, bob.name, tb_Older.name, tb_diff)

        fmt.Printf("Of %s and %s, %s is older by %d years\n", 
                    tom.name, paul.name, tp_Older.name, tp_diff)

        fmt.Printf("Of %s and %s, %s is older by %d years\n", 
                    bob.name, paul.name, bp_Older.name, bp_diff)
    }

Output:

.. container:: output

    | Of Tom and Bob, Bob is older by 7 years
    | Of Tom and Paul, Paul is older by 25 years
    | Of Bob and Paul, Paul is older by 18 years

And that's about it! ``struct`` types are handy, easy to declare and to use!

Ready for another problem?

.. _Older100:

Now, suppose that we'd like to retrieve the older of, not 2, but 10 persons!
Would you write a ``Older10`` function that would take 10 input ``person``\s?
Sure you can! But, seriously, that would be one ugly, very ugly function! Don't
write it, just picture it in your brain. Still not convinced?  Ok, imagine a
``Older100`` function with **100** input parameters! Convinced, now?

Ok, let's see how to solve these kind of problems using arrays.

.. _arrays:

Arrays
======
An array is a *block* of **n** *consecutive* objects of the same **type**. That
is a *set* of indexed objects, where you can say: *object number 1*, *object
number 2*, etc... 

In fact, in Go as in C and other languages,  the indexing starts from 0, so you
actually say: *object number 0*, *object number 1*... *object number n-1*

.. graphviz::

    digraph array_s {
        rankdir=LR;
        graph [bgcolor=transparent, resolution=96, fontsize="10" ];
        edge [arrowsize=.5, arrowtail="dot", color="#555555"];
        node [shape=record, fontsize=8, height=.1, penwidth=.4]
        array[label="{<f0>A[0]|<f1>[A]1|<f2>A[2]|<f4>A[3]|...|<fn>A[n-1]}"];
    }


Here's how to declare an array of 10 ``int``\s, for example:

.. code-block:: go
    :linenos:
    
    //declare an array A of 10 ints
    var A [10]int

And here's how to declare an array of 10 ``person``\s:

.. code-block:: go
    :linenos:
    
    //declare an array A of 10 persons
    var people [10]person


You can access the x :sup:`th` element of an array ``A``, with the syntax:
**A[x]**.  So you say: A[0], A[1] and so on...

For example, the name of the 3rd person is:

.. code-block:: go
    :linenos:
    
    //declare an array A of 10 persons
    var people [10]person
    
    //Remember indices start from 0!
    third_name := people[2].name //use the dot to access the third 'name'.
    //or more verbosely:
    third_person := people[2]
    thirdname := third_person.name 

.. graphviz::

    digraph array_s {
        //rankdir=LR;
        graph [bgcolor=transparent, resolution=96, fontsize="10" ];
        edge [arrowsize=.5, arrowhead="dot", color="#555555"];
        node [shape=record, fontsize=8, height=.1, penwidth=.4]
        people[label="<f0>people[0]|<f1>[people1]|<f2>people[2]|<f3>people[3]|...|<fn>people[n-1]"];
        p2[shape=circle, penwidth=1, color=crimson, fontcolor=black, label=<
        <TABLE BORDER="0" CELLBORDER="1" CELLSPACING="0" CELLPADDING="4" VALIGN="MIDDLE">
            <TR>
                <TD BGCOLOR="lightyellow">Tom</TD>
                <TD BGCOLOR="lightblue">25</TD>
            </TR>
        </TABLE>>]
        p2->people:f2 [label=" Zoom on people[2]", fontsize=6, color=crimson, fontcolor=crimson]
    }

The idea now is to write a function ``Older10`` that takes an array of 10
``person``\s as its input, and returns the older ``person`` in this array.

.. code-block:: go
    :linenos:

    package main
    import "fmt"

    //Our struct
    type person struct{
        name string
        age int
    }

    //return the older person in a group of 10 persons
    func Older10(people [10]person) person{
        older := people[0] //the first one is the older for now
        //loop through the array and check if we could find an older person.
        for i:= 1; i<10; i++{ //we skipped the first here
            if people[i].age > older.age{ //compare the current's person age with the older one
                older = people[i] //if people[i] is older, replace the value of older
            }
        }
        return older
    }

    func main(){
        // declare an example array variable of 10 person called: array
        var array [10]person

        // initialize some of the elements of the array, the others ar by default
        // set to person{"", 0} 
        array[1] = person{"Paul", 23};
        array[2] = person{"Jim", 24};
        array[3] = person{"Sam", 84};
        array[4] = person{"Rob", 54};
        array[8] = person{"Karl", 19};

        older := Older10(array) //call the function by passing it our array

        fmt.Println("The older of the group is: ", older.name)
    }

Output:

.. container:: output

    | The older of the group is:  Sam


We could have declared and initialized the variable ``array`` of the ``main()``
function above in a single shot like this:

.. code-block:: go
    :linenos:

    //declare and initialize an array A of 10 person
    array := [10]person  {
        person{"", 0},
        person{"Paul", 23},
        person{"Jim", 24},
        person{"Sam", 84},
        person{"Rob", 54},
        person{"", 0},
        person{"", 0},
        person{"", 0},
        person{"Karl", 10},
        person{"", 0}}

This means that to initialize an array, you put its elements between two braces,
and you separate them with commas. 

We can even omit the size of the array, and Go will count the number of element
given in the initialization. So we could have written:

.. code-block:: go
    :linenos:

    //declare and initialize an array of 10 person, but let the compiler guess the size.
    array := [...]person  { //omit the size, and put ... instead.
        person{"", 0},
        person{"Paul", 23},
        person{"Jim", 24},
        person{"Sam", 84},
        person{"Rob", 54},
        person{"", 0},
        person{"", 0},
        person{"", 0},
        person{"Karl", 10},
        person{"", 0}}

.. _arrays-note:

.. note::
    **However**, the **size of an array is an important part of its
    definition**.  They don't *grow* and they don't *shrink*. You can't, for
    example, have an array of 9 elements, and use it with a function that
    expects an array of 10 elements as an input. Simply because they are of
    *different* types. **Rememeber this.**

Some things to keep in mind too:

- Arrays are values. Assigning one array to another, copies all the elements.
  That is: if ``A``, and ``B`` are arrays of the same type, and we write: ``B =
  A``, then ``B[0] == A[0]``, ``B[1] == A[1]``... ``B[n-1] == A[n-1]``.

- When passing an array to a function, it is copied. i.e. The function recieves
  a copy of that array, and *not* a reference (pointer) to it.

- Go comes with a built-in function ``len`` that returns variables sizes. In the
  previous example: ``len(array) == 10``.

Multi-dimensional arrays
========================
You can think to yourself: *"Hey! I want an array of arrays!"*. Yes, that's
possible, and often used actually.

You can declare a 2-dimensional array like this:

.. code-block:: go
    :linenos:

    //declare and initialize an array of 2 arrays of 4 ints
    a := [2][4]int {[4]int{1,2,3,4}, [4]int{5,6,7,8}}

This is an array of (2 arrays (of 4 ``int``)). You can think of it as a matrix,
or a table of two lines each made of 4 columns.

.. graphviz::

    digraph array_2 {
        rankdir=LR;
        graph [bgcolor=transparent, resolution=96, fontsize="10" ];
        edge [arrowsize=.5, arrowtail="dot", color="#ff6600"];
        node [shape=record, fontsize=8, height=.1, penwidth=.4]
        array[label="{<f00>1|<f01>2|<f02>3|<f03>4}|{<f10>5|<f11>6|<f12>7|<f13>8}"];
        node[shape="plaintext", fixedsize="true"]
        exnode00 [label="A[0,0]"];
        exnode01 [label="A[0,1]"];
        exnode11 [label="A[1,1]"];
        exnode13 [label="A[1,3]"];
        exnode00->array:f00;
        exnode01->array:f01;
        exnode11->array:f11;
        exnode13->array:f13;
    }

The previous declaration may be simplified, using the fact that the compiler can
count arrays' elements for us, like this:

.. code-block:: go
    :linenos:

    //simplify the previous declaration, with the '...' syntax
    a := [2][4]int {[...]int{1,2,3,4}, [...]int{5,6,7,8}}

Guess what? Since Go is about cleaner code, you can even simplify:

.. code-block:: go
    :linenos:

    //über simpifikation!
    a := [2][4]int {{1,2,3,4}, {5,6,7,8}}

Cool, eh? Now, we can combine multiple fields of different types to create a
``struct`` and we can have ``array``\s of, as many as we want, of objects of the
same type, and pass it as a *single block* to our functions. But this is just
the begining! In the next chapter, we will see how to create and use some
advanced composite data types with pointers.
