.. include:: common.txt

Even more on functions
**********************
In the previous chapter, we learned how to write variadic, and recursive
functions and how to use the ``defer`` statement to defer functions calls to
just before the currently executing function returns.

In this chapter, we will learn something that might seem surprising a little
bit, which will end up being *awesome*.

Function types
==============
Do you remember when we talked about
:ref:`functions signatures <functions-signatures>`? That what matters most in a
function, beside what it actually does, is the parameters and results numbers
and types.

Well, a function type denotes the set of all functions with the same parameter
and result types.

For example: the functions with signatures: func ``SUM(int, int) int`` and
``MAX(int, int) int`` are of the same type, because their parameter count,
results count and types are the same.

Does this sound strange? One computes a sum, and the other the max, yet they are
both of the same type?

Yes, they are. Picture this with me: The numbers 1, and 23 are both of the same
type (``int``) yet their *values* are different. Paul and Sam are both of the
same "human" type, yet their "values" (beliefs, jobs, weight...) are different.

The function's body, what it actually does, is what makes the difference between
two functions of the same type. Just like how many coffees you can by with 1
dollar or 23 dollars.

Same thing for functions: What they do, their bodies, is what differentiate two
functions of the same type.

Convinced? Let's see how to declare functions types.

How to declare a function type?
-------------------------------
The syntax is simple:

``type type_name func(input1 inputType1 [, input2 inputType2 [, ...]) (result1 resultType1 [, ...])``

Some examples to illustrate this:

.. code-block:: go
    :linenos:

    // twoOne is the type of functions with two int parameters and an int result.
    // max(int, int) int and sum(int, int) int, for eg. are of this type.
    type twoOne func(int, int) int

    // slice_transform is the type of functions that takes a slice and outputs a
    // slice.
    // invert(s []int) []int and sort(s []int) []int are of this type.
    type slice_transform func(s []int) []int

    // varbytes is the type of variadic functions that takes bytes and outputs a
    // boolean.
    // redundant(...byte) bool, and contains_zero(...byte) bool are of this type
    type varbytes func(...byte) bool

Is it clear now?

*Yes, by why is this even useful or worth the hassle?* You might find yourself
asking. Well, I'm glad you asked this question! Read on, you will see the
beautiful madness to this method!

Functions are values
====================
Functions of the same type are values of this type! This means: You can declare
a variable of a given function-type and assign any function of the same type to
it. You can pass functions as parameters to other functions, you can also have
functions that return other functions as results...  And this, my friend, offers
a lot of beautiful, intellignt and powerful opportunities! Let me show you some
examples (don't worry, this doesn't result in Skynet... quite...).

**Example 1**: Write a program which when given a slice of integers will return
another slice which contains only the odd elements of the first slice (yes...
of course... by 'odd' I meant the 'weird' ones... what is wrong with you people?!?).

.. code-block:: go
    :linenos:

    package main
    import "fmt"

    type test_int func(int) bool

    // isOdd takes an ints and returns a bool set to true if the
    // int parameter is odd, or false if not.
    // isOdd is of type func(int) bool which is what test_int is declared to be.

    func isOdd(integer int) bool {
        if integer%2 == 0 {
            return false
        }
        return true
    }

    // Same comment for isEven
    func isEven(integer int) bool {
        if integer%2 == 0 {
            return true
        }
        return false
    }

    // We could've written:
    // func filter(slice []int, f func(int) bool) []int
    func filter(slice []int, f test_int) []int {
        var result []int
        for _, value := range slice {
            if f(value) {
                result = append(result, value)
            }
        }
        return result
    }

    func main(){
        slice := []int {1, 2, 3, 4, 5, 7}
        fmt.Println("slice = ", slice)
        odd := filter(slice, isOdd)
        fmt.Println("Odd elements of slice are: ", odd)
        even := filter(slice, isEven)
        fmt.Println("Even elements of slice are: ", even)
    }

Output:

.. container:: output

    | s =  [1 2 3 4 5 7]
    | Odd elements of s are:  [1 3 5 7]
    | Even elements of s are:  [2 4]

The functions ``isOdd`` and ``isEven`` are very simple.
They both take an ``int``, and return a ``bool``.
Of course, they're just little examples of functions of type ``test_int``.
One can imagine more advanced and complex functions that are of this type.

The interesting part is the ``filter`` function, which takes a slice of type
``[]int`` *and* a function of type ``test_int`` and returns a slice of type
``[]int``.

Look at how we made a call ``f(value)`` on line 30. In fact, the function
``filter`` doesn't *care* how ``f`` works, what matters for it is that ``f``
needs an ``int`` parameter, and that it returns a ``bool`` result.

And we used this fact, in our ``main`` function. We called ``filter`` using
different functions of type ``test_int``.

If we're asked to extend the program to filter all the elements of the slice
that, for example, are multiple of 3. All we would have to do is write a new
function ``is3Multiple(i int) bool`` and use it with ``filter``.

Now, if we want to filter the odd integers that are multiple of 3, we call
filter twice likes this:

``slice := filter(filter(s, isOdd), is3Multiple)``

The function ``filter`` is good at what it does. It filters a slice given a
criteria, and this criteria is expressed in the form of functions.

Cool, isn't it?

Anonymous functions
===================
In the previous example, we wrote the functions ``isOdd`` and ``isEven`` outside
of the main function. It is possible to declare *anonymous* functions in Go,
that is functions without a name, and assign them to variables.

Let's see what this means with an example:

.. code-block:: go
    :linenos:

    package main
    import "fmt"

    func main(){
        //add1 is a variable of type func(int) int
        //we don't declare the type since Go can guess it
        //by looking at the function being assigned to it.
        add1 := func(x int) int{
            return x+1
        }

        n := 6
        fmt.Println("n = ", n)
        n = add1(n)
        fmt.Println("add1(n) = ", n)
    }

Output:

.. container:: output

    | n =  6
    | add1(n) =  7

Our variable ``add1`` is assigned a *complete* function definition, minus the
name. This function is said to be "anonymous" for this reason (lack of a name).

Let's see an example using anonymous functions, and that returns functions.

Functions that return functions
===============================
Back to the filtering problem, but now, we'd like to design it differently. We
want to write a function ``filter_factory`` that given a single function ``f``
like ``isOdd``, will produce a new function that takes a slice ``s`` of ints,
and produces two slices:

* ``yes``: a slice of the elements of ``s`` for which ``f`` returns ``true``.
* ``no``: a slice of the elements of ``s`` for which ``f`` returns ``false``.

.. code-block:: go
    :linenos:

    package main
    import "fmt"

    func isOdd(integer int) bool{
        if integer%2 == 0 {
            return false
        }
        return true
    }

    func isBiggerThan4(integer int) bool{
        if integer > 4 {
            return true
        }
        return false
    }

    /*
        filter_factory
        input: a criteria function of type: func(int) bool
        output: a "spliting" function of type: func(s []int) (yes, no []int)
        --
        To wrap your head around the declaration below, consider it this way:
        //declare parameter and result types
        type test_int func (int) bool
        type slice_split func([]int) ([]int, []int)
        and then declare it like this:
        func filter_factory(f test_int) slice_split
        --
        In summary: filter_factory takes a function, and creates another one
        of a completly different type.
    */
    func filter_factory(f func(int) bool) (func (s []int) (yes, no []int)){
        return func(s []int) (yes, no []int){
            for _, value := range s{
                if f(value){
                    yes = append(yes, value)
                } else {
                    no = append(no, value)
                }
            }
            return //look, we don't have to add yes, no. They're named results.
        }
    }

    func main(){
        s := []int {1, 2, 3, 4, 5, 7}
        fmt.Println("s = ", s)
        odd_even_function := filter_factory(isOdd)
        odd, even := odd_even_function(s)
        fmt.Println("odd = ", odd)
        fmt.Println("even = ", even)

        //separate those that are bigger than 4 and those that are not.
        //this time in a more compact style.
        bigger, smaller := filter_factory(isBiggerThan4)(s)
        fmt.Println("Bigger than 4: ", bigger)
        fmt.Println("Smaller than or equal to 4: ", smaller)
    }

Output:

.. container:: output

    | s =  [1 2 3 4 5 7]
    | odd =  [1 3 5 7]
    | even =  [2 4]
    | Bigger than 4:  [5 7]
    | Smaller than or equal to 4:  [1 2 3 4]

How does this work? First, we won't discuss ``isOdd`` and ``isBiggerThan4``
they're simple and both of the same type: ``func(int) bool``. That's all that
matters for us, now.

Now the heart of the program: the ``filter_factory`` function. Like stated in
the comments, this function takes as a parameter a function ``f`` of type
``fun(int) bool`` (i.e. the same as ``isOdd`` and ``isBiggerThan4``)

filter_factory returns an anonymous function that is of type:
``func([]int)([]int, []int)`` --or in plain english: a function that takes a
slice of ints as its parameters and outputs two slices of ints.

This anonymous function, justly, uses ``f`` to decide where to *copy* each
element of its input slice. if ``f(value)`` returns ``true`` then append it to
the ``yes`` slice, else append it to the ``no`` slice.

Like the ``filter`` function in the first example, the anonymous function
doesn't *care* how ``f`` works. All it knows is that if it gives it an ``int``
it will return a ``bool``. And that's enough to decide where to append the
value; in the ``yes`` or the ``no`` slices.

Now, back to the ``main`` function and how we use ``filter_factory`` in it.

Two ways:

The detailed one: we declare a variable ``odd_even_function`` and we assign to
it the result of calling ``filter_factory`` with ``isOdd`` as its parameter. So
odd_even_function is of type ``func([]int) ([]int, []int)``.

We call ``odd_even_function`` with ``s`` as its parameters and we retrieve two
slices: ``odd`` and ``even``.

The second way is compact. The call ``filter_factory(isBiggerThan4)`` returns a
function of type ``func([]int)([]int, []int)`` so we use that directly with our
slice ``s`` and retrieve two slices: ``bigger`` and ``smaller``.

Does it make sense, now? Re-read the code if not, it's actually simple.

Functions as data
=================
Since functions have types, and can be assigned to variables, one might wonder
whether it is possible to, for example, have an array or a slice of functions?
Maybe a struct with a function field? Why not a map?

All this is possible in fact and, when used intelligently, can help you write
simple, readable and elegant code.

Let's see a silly one:

.. code-block:: go
    :linenos:

    package main
    import "fmt"

    //These consts serve as names for activities
    const(
        STUDENT = iota
        DOCTOR
        MOM
        GEEK
    )

    //we create an "alias" of byte and name it "activity"
    type activity byte

    //A person has activities (a slice of activity) and can speak.
    type person struct{
        activities []activity
        speak func()
    }

    //people is a map string:person we access persons by their names
    type people map[string] person

    //phrases is a map activity:string. Associates a phrase with an activity.
    type phrases map[activity] string

    /*
        The function compose takes a map of available phrases
        and constructs a function that prints a sentence with these
        phrases that are appropriate for a "list" of the activities
        given in the slice a
    */
    func compose(p phrases, a []activity) (func()){
        return func(){
            for key, value := range a{
                fmt.Print(p[value])
                if key == len(a)-1{
                    fmt.Println(".")
                } else {
                    fmt.Print(" and ")
                }
            }
        }
    }

    func main(){
        // ph contains some phrases per activities
        ph := phrases{
            STUDENT: "I read books",
            DOCTOR: "I fix your head",
            MOM: "Dinner is ready!",
            GEEK: "Look ma! My compiler works!"}

        // a group of persons, and their activities
        folks := people{
            "Sam": person{activities:[]activity{STUDENT}},
            "Jane": person{activities:[]activity{MOM, DOCTOR}},
            "Rob": person{activities:[]activity{GEEK, STUDENT}},
            "Tom": person{activities:[]activity{GEEK, STUDENT, DOCTOR}},
        }

        // Let's assign them their function "speak"
        // depending on their activities
        for name, value := range folks{
            f := compose(ph, value.activities)
            k := value.activities
            //update the map's entry with a different person with the
            //same activities and their function speak.
            folks[name] = person{activities:k, speak:f}
        }

        // Now that they know what to say, let's hear them.
        for name, value := range folks{
            fmt.Printf("%s says: ", name)
            value.speak()
        }
    }

Output:

.. container:: output

    | Sam says: I read books.
    | Rob says: Look ma! My compiler works! and I read books.
    | Jane says: Dinner is ready! and I fix your head.
    | Tom says: Look ma! My compiler works! and I read books and I fix your head.

It may sound funny, and probably silly, but what I wanted to show you is how
functions can be used as ``struct`` fields just as simply as classic data types.

Yes, I know what you are thinking; It would have been more elegant if the
function ``speak`` of the type ``person`` could access her ``activities`` by
itself and we won't even need the ``compose`` function then.

That's called "Object Oriented Programming" (OOP), and even though Go isn't
really an OOP language, it supports some notions of it. This will be the subject
of our next chapter. Stay tuned, you, geek! :)

