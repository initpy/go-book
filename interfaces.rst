.. include:: common.txt

Interfaces: the awesomesauce of Go
**********************************
Let's recapitulate a little bit. We saw basic data types, and using them, we
learned how to create composite data types. We also saw basic control
structures. We then combined all of this to write functions.

We next discovered that functions are actually a kind of data, they are
themselves values and have types. We went on to learn about methods.
We used methods to write functions that act on data, and then moved on to data
types that *implement* a functionality.

In this chapter, we will Go to the next realm.  We will now learn to use the
innate spiritual ability of objects to actually *do* something.

Enough spirituality, let's make this real.

What is an interface?
=====================
Stated simply, interfaces are *sets of methods*.
We use an interface to specify a behavior of a given object.

For example, in the previous chapter, we saw that both ``Student`` and
``Employee`` can ``SayHi``, they do it differently, but it doesn't matter. They
both *know* how to say "Hi".

Let's extrapolate: ``Student`` and ``Employee`` implement another method
``Sing`` and ``Employee`` implements ``SpendSalary`` while ``Student``
implements ``BorrowMoney``.

So: ``Student`` implements: ``SayHi``, ``Sing`` and ``BorrowMoney`` and
``Employee`` implements: ``SayHi``, ``Sing`` and ``SpendSalary``.

These sets of methods are *interfaces* that ``Student`` and ``Employee``
*satisfy*. For example: both ``Student`` and ``Employee`` satisfy the interface
that contains: ``SayHi`` and ``Sing``. But ``Employee`` doesn't satisfy the
interafeca that is composed of ``SayHi``, ``Sing`` and ``BorrowMoney`` because
``Employee`` doesn't implement ``BorrowMoney``.

The interface type
==================
An *interface type* is the specification of a set of methods implemented by some
other objects. We use the following syntax:

.. code-block:: go
    :linenos:

    type Human struct {
        name string
        age int
        phone string
    }

    type Student struct {
        Human //an anonymous field of type Human
        school string
        loan float32
    }

    type Employee struct {
        Human //an anonymous field of type Human
        company string
        money float32
    }

    // A human likes to stay... err... *say* hi
    func (h *Human) SayHi() {
        fmt.Printf("Hi, I am %s you can call me on %s\n", h.name, h.phone)
    }

    // A human can sing a song, preferrably to a familiar tune!
    func (h *Human) Sing(lyrics string) {
        fmt.Println("La la, la la la, la la la la la...", lyrics)
    }

    // A Human man likes to guzzle his beer!
    func (h *Human) Guzzle(beerStein string) {
        fmt.Println("Guzzle Guzzle Guzzle...", beerStein)
    }

    // Employee's method for saying hi overrides a normal Human's one
    func (e *Employee) SayHi() {
        fmt.Printf("Hi, I am %s, I work at %s. Call me on %s\n", e.name,
            e.company, e.phone) //Yes you can split into 2 lines here.
    }

    // A Student borrows some money
    func (s *Student) BorrowMoney(amount float32) {
        loan += amount // (again and again and...)
    }

    // An Employee spends some of his salary
    func (e *Employee) SpendSalary(amount float32) {
        e.money -= amount // More vodka please!!! Get me through the day!
    }

    // INTERFACES
    type Men interface {
        SayHi()
        Sing(lyrics string)
        Guzzle(beerStein string)
    }

    type YoungChap interface {
        SayHi()
        Sing(song string)
        BorrowMoney(amount float32)
    }

    type ElderlyGent interface {
        SayHi()
        Sing(song string)
        SpendSalary(amount float32)
    }

As you can see, an interface can be satisfied (or implemented) by an arbitrary
number of types. Here, the interface ``Men`` is implemented by both ``Student``
and ``Employee``.

Also, a type can implement an arbitrary number of interfaces, here, ``Student``
implements (or satisfies) both ``Men`` and ``YoungChap`` interfaces, and
``Employee`` satisfies ``Men`` and ``ElderlyGent``.

And finally, every type implements the *empty interface* and that contains, you
guessed it: *no methods*. We declare it as ``interface{}``
(Again, notice that there are no methods in it.)

You may find yourself now saying:

  "*Cool, but what's the point in defining interfaces types?
  Is it just to describe how types behave, and what they have in common?*"

But wait! There's more! ---Of course there is more!

Oh, by the way, did I mention that the empty interface has no methods yet?

Interface values
================
Since interfaces are themselves types, you may find yourself wonderng what
exactly are the values of an interface type.

Tada! Here comes the wonderful news: If you declare a variable of an interface,
it may store any value type that implements the methods declared by the
interface!

That is, if we declare ``m`` of interface ``Men``, it may store a value of
type ``Student`` or ``Employee``, or even... (gasp) ``Human``!
This is because of the fact that they *all* implement methods specified by the
``Men`` interface.

Reason with me now: if ``m`` can store values of these different types, we can
easily declare a slice of type ``Men`` that will contain heterogeneous values.
This was not even possible with slices of classical types!

An example should help to clarify what we are saying here:

.. code-block:: go
    :linenos:

    package main
    import "fmt"

    type Human struct {
        name string
        age int
        phone string
    }

    type Student struct {
        Human //an anonymous field of type Human
        school string
        loan float32
    }

    type Employee struct {
        Human //an anonymous field of type Human
        company string
        money float32
    }

    //A human method to say hi
    func (h Human) SayHi() {
        fmt.Printf("Hi, I am %s you can call me on %s\n", h.name, h.phone)
    }

    //A human can sing a song
    func (h Human) Sing(lyrics string) {
        fmt.Println("La la la la...", lyrics)
    }

    //Employee's method overrides Human's one
    func (e Employee) SayHi() {
        fmt.Printf("Hi, I am %s, I work at %s. Call me on %s\n", e.name,
            e.company, e.phone) //Yes you can split into 2 lines here.
    }

    // Interface Men is implemented by Human, Student and Employee
    // because it contains methods implemented by them.
    type Men interface {
        SayHi()
        Sing(lyrics string)
    }

    func main() {
        mike := Student{Human{"Mike", 25, "222-222-XXX"}, "MIT", 0.00}
        paul := Student{Human{"Paul", 26, "111-222-XXX"}, "Harvard", 100}
        sam := Employee{Human{"Sam", 36, "444-222-XXX"}, "Golang Inc.", 1000}
        Tom := Employee{Human{"Sam", 36, "444-222-XXX"}, "Things Ltd.", 5000}

        //a variable of the interface type Men
        var i Men

        //i can store a Student
        i = mike
        fmt.Println("This is Mike, a Student:")
        i.SayHi()
        i.Sing("November rain")

        //i can store an Employee too
        i = Tom
        fmt.Println("This is Tom, an Employee:")
        i.SayHi()
        i.Sing("Born to be wild")

        //a slice of Men
        fmt.Println("Let's use a slice of Men and see what happens")
        x := make([]Men, 3)
        //These elements are of different types that satisfy the Men interface
        x[0], x[1], x[2] = paul, sam, mike

        for _, value := range x{
            value.SayHi()
        }
    }

Output:

.. container:: output

    | This is Mike, a Student:
    | Hi, I am Mike you can call me on 222-222-XXX
    | La la la la... November rain
    | This is Tom, an Employee:
    | Hi, I am Sam, I work at Things Ltd.. Call me on 444-222-XXX
    | La la la la... Born to be wild
    | Let's use a slice of Men and see what happens
    | Hi, I am Paul you can call me on 111-222-XXX
    | Hi, I am Sam, I work at Golang Inc.. Call me on 444-222-XXX
    | Hi, I am Mike you can call me on 222-222-XXX

As you may have noticed, interfaces types are abstract in that they don't
themselves implement a given and precise data structure or method.
They simply say: "if something can do *this*, it may be used *here*".

Notice that these types make *no mention* of any interface.
Their implementation doesn't explicitly mention a given interface.

Similarly, an interface doesn't specify or even care which types implement it.
Look how ``Men`` made no mention of types ``Student`` or ``Employee``.
All that matters for an interface is that if a type implements the methods it
declares then it can reference values of that type.

The case of the empty interface
===============================
The empty interface ``interface{}`` doesn't contain even a single method,
hence every type implements *all* 0 of its methods.

The empty interface, is not useful in *describing* a behavior (clearly, it is
an entity of few words), but rather is extremely useful when we need to store
*any* type value (since all types implement the empty interface).

.. code-block:: go
    :linenos:

    // a is an empty interface variable
    var a interface{}
    var i int = 5
    s := "Hello world"
    // These are legal statements
    a = i
    a = s

A function that takes a parameter that is of type ``interface{}`` in actuality
accepts any type. If a function returns a result of type ``interface{}`` then
we expect it to be able to return values of any type.

How is this even useful? Read on, you'll see! :sub:`(pixies)`

Functions with interface parameters
===================================
The examples above showed us how an interface variable can store any value of
any type which satisfies the interface, and gave us an idea of how we can
construct containers of heterogeneous data (different types).

In the same train of thought, we may consider functions (including methods) that
accept interfaces as parameters in order to be used with any type that satisfies
the given interfaces.

For example: By now, you already know that fmt.Print is a variadic function,
right? It accepts any number of parameters. But did you notice that sometimes,
we used ``string``\s, and ``int``\s, and ``float``\s with it?

In fact, if you look into the `fmt package`_, documentation you will find a
definition of an interface type called `Stringer`_

.. code-block:: go
    :linenos:

    //The Stringer interface found in fmt package
    type Stringer interface {
         String() string
    }

Every type that implements the ``Stringer`` interface can be passed to
``fmt.Print`` which will print out the output of the type's method ``String()``.

Let's try this out:

.. code-block:: go
    :linenos:

    package main
    import (
        "fmt"
        "strconv" //for conversions to and from string
    )

    type Human struct {
        name string
        age int
        phone string
    }

    //Returns a nice string representing a Human
    //With this method, Human implements fmt.Stringer
    func (h Human) String() string {
        //We called strconv.Itoa on h.age to make a string out of it.
        //Also, thank you, UNICODE!
        return "❰"+h.name+" - "+strconv.Itoa(h.age)+" years -  ✆ " +h.phone+"❱"
    }

    func main() {
        Bob := Human{"Bob", 39, "000-7777-XXX"}
        fmt.Println("This Human is : ", Bob)
    }

Output:

.. container:: output

    | This Human is : ❰Bob - 39 years -  ✆ 000-7777-XXX❱

Look at how we used ``fmt.Print``, we gave it ``Bob``, a variable of type
``Human``, and it printed it so nice and purty like!
All we had to do is make the ``Human`` type implement a simple method
``String()`` that returns a string, which means that we made ``Human`` satisfy
the interface ``fmt.Stringer`` and thus were able to pass it to ``fmt.Print``.

Recall the :ref:`colored boxes example<colored-boxes-example>`?
We had a type named ``Color`` which implemented a ``String()`` method.
Let's again go back to that program, and instead of calling ``fmt.Println`` with
the result of this method, we will call it directly with the color.
You'll see it will work as expected.

.. code-block:: go
    :linenos:

    //These two lines do the same thing
    fmt.Println("The biggest one is", boxes.BiggestsColor().String())
    fmt.Println("The biggest one is", boxes.BiggestsColor())

Cool, isn't it?

Another example that will make you *love* interfaces is the `sort package`_
which provides functions to sort slices of type ``int``, ``float``, ``string``
and wild and wonderous things.

Let's first see a basic example, and then I'll show you the magic of this
package.

.. code-block:: go
    :linenos:

    package main
    import(
        "fmt"
        "sort"
    )

    func main() {
        // list is a slice of ints. It is unsorted as you can see
        list := []int {1, 23, 65, 11, 0, 3, 233, 88, 99}
        fmt.Println("The list is: ", list)

        // let's use Ints function that comes in sort
        // Ints([]int) sorts its parameter in ibcreasing order. Go read its doc.
        sort.Ints(list)
        fmt.Println("The sorted list is: ", list)
    }

Output:

.. container:: output

    | The list is:  [1 23 65 11 0 3 233 88 99]
    | The sorted list is:  [0 1 3 11 23 65 88 99 233]

Simple and does the job. But what I wanted to show you is even more beautiful.

In fact, the sort package defines a interface simply called ``Interface`` that
contains three methods:

.. code-block:: go
    :linenos:

    type Interface interface {
        // Len is the number of elements in the collection.
        Len() int
        // Less returns whether the element with index i should sort
        // before the element with index j.
        Less(i, j int) bool
        // Swap swaps the elements with indexes i and j.
        Swap(i, j int)
    }

From the `sort Interface doc`_:

  "A type, typically a collection, that satisfies sort. Interface can be sorted
  by the routines in this package. The methods require that the elements of the
  collection be enumerated by an integer index.

So all we need in order to sort slices of any type is to implement these  three
methods! Let's give it a try with a slice of ``Human`` that we want to sort
based on their ages.

.. code-block:: go
    :linenos:

    package main
    import (
        "fmt"
        "strconv"
        "sort"
    )

    type Human struct {
        name string
        age int
        phone string
    }

    func (h Human) String() string {
        return "(name: " + h.name + " - age: "+strconv.Itoa(h.age)+ " years)"
    }

    type HumanGroup []Human //HumanGroup is a type of slices that contain Humans

    func (g HumanGroup) Len() int {
        return len(g)
    }

    func (g HumanGroup) Less(i, j int) bool {
        if g[i].age < g[j].age {
            return true
        }
        return false
    }

    func (g HumanGroup) Swap(i, j int){
        g[i], g[j] = g[j], g[i]
    }

    func main(){
        group := HumanGroup{
            Human{name:"Bart", age:24},
            Human{name:"Bob", age:23},
            Human{name:"Gertrude", age:104},
            Human{name:"Paul", age:44},
            Human{name:"Sam", age:34},
            Human{name:"Jack", age:54},
            Human{name:"Martha", age:74},
            Human{name:"Leo", age:4},
        }

        //Let's print this group as it is
        fmt.Println("The unsorted group is:")
        for _, v := range group{
            fmt.Println(v)
        }

        //Now let's sort it using the sort.Sort function
        sort.Sort(group)

        //Print the sorted group
        fmt.Println("\nThe sorted group is:")
        for _, v := range group{
            fmt.Println(v)
        }
    }

Output:

.. container:: output

    | The unsorted group is:
    | (name: Bart - age: 24 years)
    | (name: Bob - age: 23 years)
    | (name: Gertrude - age: 104 years)
    | (name: Paul - age: 44 years)
    | (name: Sam - age: 34 years)
    | (name: Jack - age: 54 years)
    | (name: Martha - age: 74 years)
    | (name: Leo - age: 4 years)
    |
    | The sorted group is:
    | (name: Leo - age: 4 years)
    | (name: Bob - age: 23 years)
    | (name: Bart - age: 24 years)
    | (name: Sam - age: 34 years)
    | (name: Paul - age: 44 years)
    | (name: Jack - age: 54 years)
    | (name: Martha - age: 74 years)
    | (name: Gertrude - age: 104 years)

Victory! It worked like promised. We didn't implement the sorting of
``HumanGroup``, all we did is implement some methods (``Len``, ``Less``, and
``Swap``) that the ``sort.Sort`` function needed to use to sort the group for
us.

I know that you are curious, and that you wonder how this kind of *magic* works.
It's actually simple. The function Sort of the package sort has this signature:
``func Sort(data Interface)``

It accepts any value of any type that implements the interface
``sort.Interface``, and deep inside (specifically: in functions that
``sort.Sort`` calls) there are calls for the ``Len``, ``Less``, and ``Swap``
methods that you defined for your type.

Go and look into the `sort package source code`_, you'll see that in plain Go.

We have seen different examples of functions that use interfaces as their
parameters offering an abstract way to do this on variables of different types
provided they implement the interface.

Let's do that ourselves! Let's write a function that accepts an interface as
it's parameter and see how brilliant we can be.

Our own example
---------------
We've seen the ``Max(s []int) int`` function, do you remember? We also saw
``Older(s []Person) Person``. They both have the same functionality.  In
fact, retrieving the Max of a slice of ints, a slice of floats, or the older
person in a group comes down to the same thing: loop and compare values.

Let's do it.

.. code-block:: go
    :linenos:

    package main
    import (
        "fmt"
        "strconv"
    )

    //A basic Person struct
    type Person struct {
        name string
        age int
    }

    //Some slices of ints, floats and Persons
    type IntSlice []int
    type Float32Slice []float32
    type PersonSlice []Person

    type MaxInterface interface {
        // Len is the number of elements in the collection.
        Len() int
        //Get returns the element with index i in the collection
        Get(i int) interface{}
        //Bigger returns whether the element at index i is bigger that the j one
        Bigger(i, j int) bool
    }

    //Len implementation for our three types
    func (x IntSlice) Len() int {return len(x)}
    func (x Float32Slice) Len() int {return len(x)}
    func (x PersonSlice) Len() int {return len(x)}

    //Get implementation for our three types
    func(x IntSlice) Get(i int) interface{} {return x[i]}
    func(x Float32Slice) Get(i int) interface{} {return x[i]}
    func(x PersonSlice) Get(i int) interface{} {return x[i]}

    //Bigger implementation for our three types
    func (x IntSlice) Bigger(i, j int) bool {
        if x[i] > x[j] { //comparing two int
            return true
        }
        return false
    }

    func (x Float32Slice) Bigger(i, j int) bool {
        if x[i] > x[j] { //comparing two float32
            return true
        }
        return false
    }

    func (x PersonSlice) Bigger(i, j int) bool {
        if x[i].age > x[j].age { //comparing two Person ages
            return true
        }
        return false
    }

    //Person implements fmt.Stringer interface
    func (p Person) String() string {
        return "(name: " + p.name + " - age: "+strconv.Itoa(p.age)+ " years)"
    }

    /*
     Returns a bool and a value
     - The bool is set to true if there is a MAX in the collection
     - The value is set to the MAX value or nil, if the bool is false
    */
    func Max(data MaxInterface) (ok bool, max interface{}) {
        if data.Len() == 0{
            return false, nil //no elements in the collection, no Max value
        }
        if data.Len() == 1{ //Only one element, return it alongside with true
            return true, data.Get(1)
        }
        max = data.Get(0)//the first element is the max for now
        m := 0
        for i:=1; i<data.Len(); i++ {
            if data.Bigger(i, m){ //we found a bigger value in our slice
                max = data.Get(i)
                m = i
            }
        }
        return true, max
    }

    func main() {
        islice := IntSlice {1, 2, 44, 6, 44, 222}
        fslice := Float32Slice{1.99, 3.14, 24.8}
        group := PersonSlice{
            Person{name:"Bart", age:24},
            Person{name:"Bob", age:23},
            Person{name:"Gertrude", age:104},
            Person{name:"Paul", age:44},
            Person{name:"Sam", age:34},
            Person{name:"Jack", age:54},
            Person{name:"Martha", age:74},
            Person{name:"Leo", age:4},
        }

        //Use Max function with these different collections
        _, m := Max(islice)
        fmt.Println("The biggest integer in islice is :", m)
        _, m = Max(fslice)
        fmt.Println("The biggest float in fslice is :", m)
        _, m = Max(group)
        fmt.Println("The oldest person in the group is:", m)
    }

Output:

.. container:: output

    | The biggest integer in islice is : 222
    | The biggest float in fslice is : 24.8
    | The oldest person in the group is: (name: Gertrude - age: 104 years)

The ``MaxInterface`` interface contains three methods that must be implemented
by types to satisfy it.

* ``Len() int``: must return the length of the collection.
* ``Get(int i) interface{}``: must return the element at index ``i`` in the
  collection. Notice how this method returns a result of type ``interface{}`` -
  We designed it this way, so that collections of any type may return a value
  their own type, which may then be stored in an empty interface result.
* ``Bigger(i, j int) bool``: This methods returns ``true`` if the element at
  index ``i`` of the collection is bigger than the one at index ``j``, or
  ``false`` otherwise.

Why these three methods?

* ``Len() int``: Because no one said that collections must be slices. Imagine a
  complex data structure that has its own definition of length.
* ``Get(i int) interface{}``: again, no one said we're working with slices.
  Complex data structures may store and index their elements in a more complex
  fashion than ``slice_or_array[index]``.
* ``Bigger(i, j int) bool``: comparing numbers is obvious, we know which one is
  bigger, and programming languages come with numeric comparison operators such
  ``<``, ``==``, ``>`` and so on... But this notion of bigger value can be
  subtile. Person A is older (They say *bigger* in many languages) than B, if
  his age is bigger (greater) than B's age.

The different implementations of these methods by our types are fairly simple
and straightforward.

The heart of the program is the function: ``Max(data MaxInterface) (ok bool, max
interface{})`` which takes as a parameter ``data`` of type ``MaxInterface``.
``data`` has the three methods we discussed above. ``Max()`` returns two
results: a ``bool`` (``true`` if there is a Max, ``false`` if the collection is
empty) and a value of type ``interface{}`` which stores the Maximum value of the
collection.

Notice how the function ``Max()`` is implemented: no mention is made of any
specific collection type, everything it uses comes from the interface type.
This *abstraction* is what makes ``Max()`` usable by any type that implements
``MaxInterface``.

Each time we call ``Max()`` on a given data collection of a given type, it calls
these methods as they are implemented by that type. This works like a charm. If
we need to find out the max value of any collection, we only need implement the
MaxInterface for that type and it will work, just as it worked for ``fmt.Print``
and ``sort.Sort``.

That wrapps up this chapter. I'll stop here, take a break and have a good
day. Next, we will see some little details about interfaces. Don't worry! It
will be easier than this one, I promise.

C'mon, admit it! You've got a *NERDON* after this chapter!!!

.. external links and footnotes:

.. _fmt package: http://golang.org/pkg/fmt/index.html
.. _Stringer: http://golang.org/pkg/fmt/index.html#Stringer
.. _sort package: http://golang.org/pkg/sort/index.html
.. _sort Interface doc: http://golang.org/pkg/sort/index.html#Interface
.. _sort package source code: http://goo.gl/yjmvc
