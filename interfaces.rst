.. include:: common.txt

Interfaces: the awesomeness of Go
*********************************
Let's recapitulate a little bit. We saw basic data types, and using them, we
learned how to create composite data types. And we saw basic coltrol structures,
and then we combined them to write functions.

And then again, we discovered that functions are actually a kind of data, they
are values and they have types. We went from there to learn about methods.  We
used to write functions that act on data, to data types that *implement* a
functionality.

In this chapter, we will go even higher. We will learn to consider objects by
their ability to do something.

Enough spirituality, let's make this real.

What is an interface?
=====================
Simply said, interfaces are sets of methods. We use them to specify a behavior
of a given object. 

For example, in the previous chapter, we saw that both ``Student`` and
``Employee`` can ``SayHi``, they do it differently, but it doesn't matter. They
both *know* how to say "Hi".

Let's extrapolate: ``Student`` and ``Employee`` implement another method
``Sing`` and ``Employee`` implements ``SpendSalary`` while ``Student``
implements ``LoanMoney``.

So: ``Student`` implements: ``SayHi``, ``Sing`` and ``LoanMoney`` and
``Employee`` implements: ``SayHi``, ``Sing`` and ``SpendSalary``.

These sets of methods are *interfaces* that ``Student`` and ``Employee``
*satisfy*. For example: both ``Student`` and ``Employee`` satisfy the interface
that contains: ``SayHi`` and ``Sing``. But ``Employee`` doesn't satisfy the
interafeca that is composed of ``SayHi``, ``Sing`` and ``LoanMoney`` because
``Employee`` doesn't implement ``LoanMoney``.

The interface type
==================
An interface type in a specification of a set of methods implemented by some
other objects. We use this syntax:

.. code-block:: go
    :linenos:

    type Human struct{
        name string
        age int
        phone string
    }

    type Student struct{
        Human //an anonymous field of type Human
        school string
        loan float32
    }

    type Employee struct{
        Human //an anonymous field of type Human
        company string
        money float32
    }

    //A human method to say hi
    func (h *Human) SayHi(){
        fmt.Printf("Hi, I am %s you can call me on %s\n", h.name, h.phone) 
    }

    //A human can sing a song
    func (h *Human) Sing(lyrics string){
        fmt.Println("La la la la...", lyrics) 
    }

    //Employee's method overrides Human's one
    func (e *Employee) SayHi(){
        fmt.Printf("Hi, I am %s, I work at %s. Call me on %s\n", e.name,
            e.company, e.phone) //Yes you can split into 2 lines here.
    }

    //A Student loans some money
    func (s *Student) LoanMoney(amount float32){
        loan += amount 
    }

    // An Employee spends some of his salary
    func (e *Employee) SpendSalary(amount float32){
        e.money -= amount 
    }

    // INTERFACES
    type Men interface{
        SayHi() 
        Sing(lyrics string) 
    }

    type YoungChap interface{
        SayHi() 
        Sing(song string) 
        LoanMoney(amount float32)
    }
    
    type OldGuy interface{
        SayHi() 
        Sing(song string) 
        SpendSalary(amount float32) 
    }

As you can see, an interface can be satisfied (or implemented) by an arbitrary
number of types. Here, the interface ``Men`` is implemented by both ``Student``
and ``Employee``.

Also, a type can implement an arbitrary number of interfaces, here, ``Student``
implements (or satisfies) both ``Men`` and ``YoungChap`` interfaces, and
``Employee`` satisfies ``Men`` and ``OldGuy``.

And finally, every type implements the empty interface and that contains, you
guessed it: 0 methods. We declare it: ``interface{}`` (No methods in it)

*Cool, but what's the point in defining interfaces types? Is it just to describe
how types behave, and what they have in common?* ---Of course there is more!

Interface values
================
Since interfaces are types, you may wonder what are the values of these types.
Tada! Here comes the wonderful news: If you declare a variable of an interface,
it may store any value that implements this interface!

That is, if we declare ``m`` of interface ``Men``, it may store a value of
type ``Student`` or ``Employee``, or even... ``Human``!  Because they all
implement methods specified in the ``Men`` interface.

Now think with me: if ``m`` can store values of these different types, we can
easily declare a slice of type ``Men`` that will contain heterogynous values,
and this was not possible with slices of classical types.

Let's see an example:

.. code-block:: go
    :linenos:

    package main
    import "fmt"

    type Human struct{
        name string
        age int
        phone string
    }

    type Student struct{
        Human //an anonymous field of type Human
        school string
        loan float32
    }

    type Employee struct{
        Human //an anonymous field of type Human
        company string
        money float32
    }

    //A human method to say hi
    func (h Human) SayHi(){
        fmt.Printf("Hi, I am %s you can call me on %s\n", h.name, h.phone) 
    }

    //A human can sing a song
    func (h Human) Sing(lyrics string){
        fmt.Println("La la la la...", lyrics) 
    }

    //Employee's method overrides Human's one
    func (e Employee) SayHi(){
        fmt.Printf("Hi, I am %s, I work at %s. Call me on %s\n", e.name,
            e.company, e.phone) //Yes you can split into 2 lines here.
    }

    // Interface Men is implemented by Human, Student and Employee
    // because it contains methods implemented by them.
    type Men interface{
        SayHi() 
        Sing(lyrics string) 
    }

    func main(){
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

As you may see, interfaces types are an abstract thing in that they don't
implement a given and precise data structure. They just say: if something can do
*this*, it may be used *here*.

Also, notice that types implementing a given interface do this simply by
implementing the methods specified in the interface, they don't *claim* 
explicitely that they are implementing this interface.

And similarly, an interface doesn't specify or even *care* which types
implement it. Look how ``Men`` made no mention of types ``Student`` or
``Employee``. All that matters for it is that if a type implements its methods,
it can store values of it.

The case of the empty interface
===============================
Sure, the empty interface ``interface{}`` doesn't contain any method, hence every
type implements *all* of its 0 methods. This interface, indeed, is not useful in
describing a behavior, but may serve as a type of variables that can store any
value, since all types implement it.

.. code-block:: go
    :linenos:

    // a is an empty interface variable
    var a interface{}
    var i int = 5
    s := "Hello world"
    //These are legal statements
    a = i
    a = s

A function that takes a parameter that is of type ``interface{}`` actually
accepts any type, and if it returns a result of type ``interface{}`` you can
expect it to return anything.

How is this even useful? Read on, you'll see.

Functions with interface parameters
===================================
The examples above showed us how an interface variable can store any value of
any type that satsifies this interface, and gave us an idea of how we can
construct containers of hetergenous data.

In the same train of thought, we can consider function (and methods) that
accept interfaces as parameters in order, justly, to be used with any type that
satisfies that interface.

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
``fmt.Print`` and ``fmt.Print`` will print out the output of its method
``String``.

Let's try this out:

.. code-block:: go
    :linenos:

    package main
    import (
        "fmt"
        "strconv" //for conversions to and from string 
    )

    type Human struct{
        name string
        age int
        phone string
    }

    //Returns a nice string representing a Human
    //With this method, Human implements fmt.Stringer
    func (h Human) String()string{
        //We called strconv.Itoa on h.age to make a string out of it.
        //Also, thank you, UNICODE!
        return "❰"+h.name+" - "+strconv.Itoa(h.age)+" years -  ✆ " +h.phone+"❱"
    }

    func main(){
        Bob := Human{"Bob", 39, "000-7777-XXX"}
        fmt.Println("This Human is : ", Bob)
    }

Output:

.. container:: output

    | This Human is : ❰Bob - 39 years -  ✆ 000-7777-XXX❱

Look at how we used ``fmt.Print``, we gave it ``Bob``, a variable of type
``Human``, and it printed it so nicely! All we had to do is make the ``Human``
type implement a simple method ``String`` that returns a string, which means
that we made ``Human`` satisfy the interface ``fmt.Srtinger``.

Do you remember the :ref:`colored boxes example<colored-boxes-example>`? We had
a type named ``Color`` that implemented a ``String`` method too. Go back to that
program, and instead of calling ``fmt.Println`` with the result of this method,
call it directly with the color. You'll see it will work as expected.

.. code-block:: go
    :linenos:

    //These two lines do the same thing
    fmt.Println("The biggest one is", boxes.BiggestsColor().String())
    fmt.Println("The biggest one is", boxes.BiggestsColor())

Cool, isn't it?

Another example that will make you *love* interfaces is the `sort package`_
which provides functions to sort slices of ``int``\s, ``float``\s, ``string``\s
and other useful things.

Let's see a basic example, and then I'll show you the magic of this package.

.. code-block:: go
    :linenos:

    package main
    import(
        "fmt" 
        "sort"
    )

    func main(){
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

Always from the `sort Interface doc`_: A type, typically a collection, that
satisfies sort.Interface can be sorted by the routines in this package. The
methods require that the elements of the collection be enumerated by an integer
index. 

So all we need to sort slices of any type is to implement these methods! Let's
give it a try with a slice of ``Human`` that we want to sort based on their
ages.

.. code-block:: go
    :linenos:

    package main
    import (
        "fmt"
        "strconv"
        "sort"
    )

    type Human struct{
        name string
        age int
        phone string
    }

    func (h Human) String() string{
        return "(name: " + h.name + " - age: "+strconv.Itoa(h.age)+ " years)"
    }

    type HumanGroup []Human //HumanGroup is a type of slices that contain Humans

    func (g HumanGroup) Len() int{
        return len(g) 
    }

    func (g HumanGroup) Less(i, j int) bool{
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
``sort.Interface``, and deep inside (understand: in functions that ``sort.Sort``
calls) there are calls for the ``Len``, ``Less``, and ``Swap`` methods that you
defined for your type.

Go and look into the `sort package source code`_, you'll see that in plain Go.

We saw two different examples of functions that use interfaces as their
parameters to offer an abstract way to do this on variables of different type
provided that they implement the said interface.

Let's just do that ourselves too! Let's write a function that accepts an
interface as its parameter and see how smart we can be.

Our own example
---------------
We've seen the ``Max(s []int) int`` function, do you remember? And we also saw
the ``Older(s []Person) Person`` one too! They have the same functionality.  In
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
    type Person struct{
        name string
        age int
    }

    //Some slices of ints, floats and Persons
    type IntSlice []int
    type Float32Slice []float32
    type PersonSlice []Person

    type MaxInterface interface{
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
    func (x IntSlice) Bigger(i, j int) bool{
        if x[i] > x[j] { //comparing two int
            return true 
        }
        return false
    }

    func (x Float32Slice) Bigger(i, j int) bool{
        if x[i] > x[j] { //comparing two float32
            return true 
        }
        return false
    }

    func (x PersonSlice) Bigger(i, j int) bool{
        if x[i].age > x[j].age { //comparing two Person ages
            return true 
        }
        return false
    }

    //Person implements fmt.Stringer interface
    func (p Person) String() string{
        return "(name: " + p.name + " - age: "+strconv.Itoa(p.age)+ " years)"
    }

    /*
     Returns a bool and a value
     - The bool is set to true if there is a MAX in the collection
     - The value is set to the MAX value or nil, if the bool is false
    */
    func Max(data MaxInterface) (ok bool, max interface{}){
        if data.Len() == 0{
            return false, nil //no elements in the collection, no Max value
        }
        if data.Len() == 1{ //Only one element, return it alongside with true
            return true, data.Get(1)
        }
        max = data.Get(0)//the first element is the max for now
        m := 0
        for i:=1; i<data.Len(); i++{ 
            if data.Bigger(i, m){ //we found a bigger value in our slice
                max = data.Get(i)
                m = i
            }
        }
        return true, max
    }

    func main(){
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

* ``Len() int``: This method returns the lentgh of the collection
* ``Get(int i) interface{}``: This one returns the element at index ``i`` in the
  collection. Notice how this method returns a result of type ``interface{}`` -
  We designed it this way, so that collections of any type may return a value
  their own type, and this value can be a stored in an empty interface result.
* ``Bigger(i, j int) bool``: This methods returns ``true`` if the element at
  index ``i`` of the collection is bigger than the one at index ``j``, or
  ``false`` otherwise.

Why three methods?

* ``Len() int``: Because no one said that collections must be slices. Imagine a
  complex data structure that has its own definition of length.
* ``Get(i int) interface{}``: again, no one said we're working with slices.
  Complex data structures may store and index their elements in a more complex
  fashion than ``slice_or_array[index]``.
* ``Bigger(i, j int) bool``: comparing numbers is obvious, we know which one is
  bigger, and programming languages come with numeric comparison operators such
  ``<``, ``==``, ``>`` and so on... But this notion of bigger value can be
  subtile. Person A is older (They say *bigger* in many languages) than B, if
  his age is bigger than B's one.

The different implementations of these methods by our types are pretty simple
and straightforward.

The heart of the program is the function: ``Max(data MaxInterface) (ok bool, max
interface{})``. It takes as a parameter ``data`` of type ``MaxInterface``. So
``data`` has the three methods discussed before. And it returns two results: a
``bool`` (``true`` if there is a Max, ``false`` if the collection is emply) and
a value of type ``interface{}`` which stores the Maximum value of the
collection.

Notice how the function Max is implemented: it doesn't make any mention of any
specific collection type, everything it uses comes from the interface type. This
*abstraction* is what makes it usable by any type that implements
``MaxInterface``. 

Each time we call Max on a given data collection of a given type, it calls these
methods as they are implemented by that type. And this works like a charm. If we
need to find out the Max of any collection, we will just have to implement the
MaxInterface for that type and it will work, like it worked for ``fmt.Print``
and ``sort.Sort``. 

And... that's it for this chapter. I'll stop here, take a break and have a good
day. Next, we will see some little details about interfaces. Don't worry! It
will be easier than this one, I promise. 

.. external links and footnotes:

.. _fmt package: http://golang.org/pkg/fmt/index.html
.. _Stringer: http://golang.org/pkg/fmt/index.html#Stringer
.. _sort package: http://golang.org/pkg/sort/index.html
.. _sort Interface doc: http://golang.org/pkg/sort/index.html#Interface
.. _sort package source code: http://goo.gl/yjmvc
