.. include:: common.txt

Advanced composite types: Maps
******************************
This chapter will be lighter than the :doc:`previous one<slices>`, so don't
worry, and smile for the perspective of adding a new tool to your box.

Arrays and slices are wonderful tools for collecting elements of the same type
in a *sequential* fashion.
We simply say: the first element, the second element, ... the **n** :sup:`th`
element of the array or the slice. We can access and modifiy elements using
their numerical integer indices in the ``array`` or the *slice*.

This is nice and quite useful. Now suppose that we'd like to access and modify
elements given a name of some type (a non-integer name or 'index').
For example: Say we wish to access the definition of the word "Hello" in a
dictionary or to find out the capital of "Japan".

These category of problems call for a new kind of type.
A type where we can specify a *key* from which a *value* will be stored and
retrieved. It's not about the n\ :sup:`th` element of a sequence, it's about an
*association* of things (keys) with other things (values).

This kind of types is called a *dict* in Python and *map* in Go and a *hash* in
Ruby.

How to declare a map?
=====================
The syntax of a map type is: ``map[keyType] valueType``.

For example:

.. code-block:: go
    :linenos:

    // A map that associates strings to int
    // eg. "one" --> 1, "two" --> 2...
    var numbers map[string] int //declare a map of strings to ints

You can access and assign a value to an *entry* of this map using the square
bracket syntax as in arrays or slices, but instead of an ``int`` index you use a
key of type ``keyType``.

As with slices, since maps are reference types, we can make them using the
``make`` function: ``make(map[string]float32)``.

When used with maps, ``make`` takes an optional capacity parameter.

.. code-block:: go
    :linenos:

    // A map that associates strings to int
    // eg. "one" --> 1, "two" --> 2...
    var numbers map[string] int //declare a map of strings to ints
    numbers = make(map[string]int)
    numbers["one"] = 1
    numbers["ten"] = 10
    numbers["trois"] = 3 //trois is "three" in french. I know that you know.
    //...
    fmt.Println("Trois is the french word for the number: ", numbers[trois])
    // Trois is the french word for the number: 3. Also a good time.

We now have the idea: it's like a table with two columns: in the left column we
have the key, and on the right column we have its *associated* value.

.. graphviz::

    digraph numbers_map {

        //rankdir=LR;
        graph [bgcolor=transparent, resolution=96, fontsize="10" ];
        edge [arrowsize=.5, arrowhead="dot", arrowtail="dot", color="#555555"];
        node [fontsize=8, height=.1, penwidth=.4]
        numbers [shape="plaintext", label=<
        <table cellspacing="0" border="0" cellborder="1" cellpadding="8">
            <tr>
                <td bgcolor="lightblue">Key</td>
                <td bgcolor="lightblue">Value</td>
            </tr>
            <tr>
                <td>"one"</td><td>1</td>
            </tr>
            <tr>
                <td>"ten"</td><td>10</td>
            </tr>
            <tr>
                <td>"trois"</td><td>3</td>
            </tr>
            <tr>
                <td colspan="2" border="0">numbers</td>
            </tr>
        </table>>]
    }

Some things to notice:

- There is no defined notion of *order*. We do not access a value by an index,
  but rather by a *key* instead.
- The size of a map is not fixed like in arrays. In fact, just like a *slice*,
  a map is a *reference* type.
- Doing for example ``numbers["coffees_I_had"] = 7`` will actually add an entry
  to this table, and the size of the map will be incremented by 1.
- As in slices and arrays, the built-in function ``len`` will return the number
  of keys in a map (thus the number of entries).
- Values can be changed, of course. ``numbers["coffees_I_had"] = 12`` will
  change the ``int`` value associated with the string "coffes_I_had".

Literal values of maps can be expressed using a list of colon-separated
``key:value`` pairs. Let's see an example of this:

.. code-block:: go
    :linenos:

    // A map representing the rating given to some programming languages.
    rating := map[string]float32 {"C":5, "Go":4.5, "Python":4.5, "C++":2 }
    // This is equivalent to writing more verbosely
    var rating = map[string]float32
    rating = make(map[string]float)
    rating["C"] = 5
    rating["Go"] = 4.5
    rating["Python"] = 4.5
    rating["C++"] = 2 //Linus would put 1 at most. Go ask him

Maps are references
-------------------
If you assign a map ``m`` to another map ``m1``, they will both refer to the
same underlying structure that holds the key/value pairs.
Thus, changing the value associated with a given key in ``m1`` will also change
the value of that key in ``m`` as they both reference the same underlying data:

.. code-block:: go
    :linenos:

    //let'ss ay a translation dictionary
    m = make(map[string][string])
    m["Hello"] = "Bonjour"
    m1 = m
    m1["Hello"] = "Salut"
    // Now: m["Hello"] == "Salut"

Checking existence of a key
---------------------------
**Question** What would the expression ``rating["C#"]`` return as a value, in our previous
example?

Good question! The answer is simple: the expression will return the zero value
of the value type.

The value type in our example is ``float32``, so it will return '0.00'.

.. code-block:: go
    :linenos:

    //A map representing the rating given to some programming languages.
    rating := map[string]float32 {"C":5, "Go":4.5, "Python":4.5, "C++":2 }
    csharp_rating := rating["C#"]
    //csharp_rating == 0.00

But then, if the value associated with an inexistent key is the zero of the type
value, how can we be sure that C#'s rating is actually 0.00? In other words: is
C#'s rating actually 0.00 so we can say that as a language it *stinks* or was it
that it was not rated at all?

Here comes the "comma-ok" form of accessing a key's associated value in a map.
It has this syntax: ``value, present = m[key]``.
Where ``present`` is a boolean that indicates whether the key is present in the
map.

.. code-block:: go
    :linenos:

    //A map representing the rating given to some programming languages.
    rating := map[string]float32 {"C":5, "Go":4.5, "Python":4.5, "C++":2 }
    csharp_rating, ok := rating["C#"]
    //would print: We have no rating associated with C# in the map
    if ok {
        fmt.Println("C# is in the map and its rating is ", csharp_rating)
    } else {
        fmt.Println("We have no rating associated with C# in the map")
    }

We often use ``ok`` as a variable name for boolean presence, hence the name
"comma-ok" form. But, hey! You're free to use any name as long as it's a
``bool`` type.

Deleting an entry
-----------------
To delete an entry from the map, think of an *inversed* "comma-ok" form.
In fact, you just have to assign any given value followed by comma ``false``.

.. code-block:: go
    :linenos:

    // A map representing the rating given to some programming languages.
    rating := map[string]float32 {"C":5, "Go":4.5, "Python":4.5, "C++":2 }
    map["C++"] = 1, false // We delete the entry with key "C++"
    cpp_rating, ok := rating["C++"]
    // Would print: We have no rating associated with C++ in the map
    if ok {
        fmt.Println("C++ is in the map and its rating is ", cpp_rating)
    } else {
        fmt.Println("We have no rating associated with C++ in the map")
    }

If in line 2, we had ``map["C++"] = 1, true`` the output of the if-else
statement would be: *C++ is in the map and its rating is 1*. i.e. the entry
associated with key "C++" would be kept in the map, and its value changed to 1.

Cool! We now have a sexy new type which allows us to easily add key/value pairs,
check if a given key is present, and delete any given key. Simply.

The question now is: "How do I retrieve all the elements in my map?"
More specifically, "How do I print a list of all the languages in the ``rating``
map with their respective ratings?"

The range clause
================
For maps (and arrays, and slices, and other stuff which we'll see later), Go
comes with a facinating alteration of the syntax for the "for statement".

This syntax is as follow:

.. code-block:: go
    :linenos:

    for key, value := range m {
        // In each iteration of this loop, the variables key and value are set
        // to the current key/value in the map
        ...
    }

Let's see a complete example to understand this better.

.. code-block:: go
    :linenos:

    package main
    import "fmt"

    func main(){
        // Declare a map literal
        ratings := map[string]float32 {"C":5, "Go":4.5, "Python":4.5, "C++":2 }

        // Iterate over the ratings map
        for key, value := range ratings {
            fmt.Printf("%s language is rated at %g\n", key, value)
        }
    }

Output:

.. container:: output

    | C++ language is rated at 2
    | C language is rated at 5
    | Go language is rated at 4.5
    | Python language is rated at 4.5


If we don't need the value in our for statement, we can omit it like this:

.. code-block:: go
    :linenos:

    package main
    import "fmt"

    func main(){
        // Declare a map literal.
        ratings := map[string]float32 {"C":5, "Go":4.5, "Python":4.5, "C++":2 }

        fmt.Print("We rated these languages: ")

        // Iterate over the ratings map, and print the languages names.
        for key := range ratings {
            fmt.Print(key, ",")
        }
    }

Output:

.. container:: output

    | We rated these languages: C++,C,Go,Python,

**Exercise**: Modify the program above to replace the last comma in the list by
a period. That is, output:
"We rated these languages: C++,C,Go,Python."

This "for statement" form is also available for arrays and slices where instead
of a key we have an index.

Let's rewrite a previous example using this new tool:

.. code-block:: go
    :linenos:

    package main
    import "fmt"

    // Return the biggest value in a slice of ints.
    func Max(slice []int) int { // The input parameter is a slice of ints.
        max := slice[0] //the first element is the max for now.
        for index, value := range slice { // Notice how we iterate!
            if value > max { // We found a bigger value in our slice.
                max = value
            }
        }
        return max
    }

    func main() {
        // Declare three arrays of different sizes, to test the function Max.
        A1 := [10]int {1,2,3,4,5,6,7,8,9}
        A2 := [4]int {1,2,3,4}
        A3 := [1]int {1}

        //declare a slice of ints
        var slice []int

        slice = A1[:] // Take all A1 elements.
        fmt.Println("The biggest value of A1 is", Max(slice))
        slice = A2[:] // Ttake all A2 elements.
        fmt.Println("The biggest value of A2 is", Max(slice))
        slice = A3[:] // Ttake all A3 elements.
        fmt.Println("The biggest value of A3 is", Max(slice))
    }

Output:

.. container:: output

    | The biggest value of A1 is 9
    | The biggest value of A2 is 4
    | The biggest value of A3 is 1

Look carefully at line 6. We used a ``range`` over a slice of ints. ``i`` is an
index and it goes from 0 to ``len(s)-1`` and ``value`` is an int that goes from
``s[0]`` to ``s[len(s)-1]``.

Notice also how we didn't use the index ``i`` in this loop. We didn't need it.

The blank identifier
====================
Whenever a function returns a value you don't care about, or a range returns an
index  that you don't care about, you can use the *blank identifier* _
(underscore) This predeclared identifier can be assigned any value of any type,
and it will be discarded.

We could have written the ``Max(s []int)int`` function like this:

.. code-block:: go
    :linenos:

    // Return the biggest value in a slice of ints.
    func Max(slice []int) int { // The input parameter is a slice of ints.
        max := slice[0]
        for _, value := range slice { // Notice how we use _ to "ignore" the index.
            if value > max {
                max = value
            }
        }
        return max
    }

Also, say, we have a function that returns two or more values of which some are
unimportant for us. We can "ignore" these output results using the *blank
identifier*.

.. code-block:: go
    :linenos:

    // A function that returns a bool that is set to true of Sqrt is possible
    // and false when not. And the actual square root of a float64
    func MySqrt(floater float64) (squareroot float64, ok bool) {
        if floater > 0 {
            squareroot, ok = math.Sqrt(f), true
        } else {
            squareroot, ok = 0, false
        }
        return squareroot, ok
    }
    //...
    r,_ = MySqrt(v) //retrieve the square root of v, and ignore its faisability

That's it for this chapter. We learned about *maps* ; how to make them, how to
add, change, and delete key/value pairs from them. How to check if a given key
exists in a given map. And we also learned about the *blank identifier* '``_``'
and the ``range`` clause.

Yes, the ``range`` clause, especially, is a control flow construct that should
belong in the chapter about :doc:`control flow<control>`, but we didn't yet
know about arrays, slices or maps, at that time. This is why we deferred it
until this chapter.

The next chapter will be about things that we didn't mention about functions in
the chapter about :doc:`functions <functions>` for the same reason: lack of
prior exposure, or simply because I wanted the chapter to be light so we
can make progress with advanced data structures.

Anyways, you'll see, it will be fun! See you in the next chapter! :)

