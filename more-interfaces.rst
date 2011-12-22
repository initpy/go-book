.. include:: common.txt

More on interfaces
******************
We discovered the magic of interfaces in the previous chapter, and how a simple
concept as defining a type's behavior can offer tremendous opportunities.
I you haven't read the previous chapter, you shouldn't read this one, simply
because most of what we're about to study depends on a good understanding of the
previous chapter.

Shall we begin?

Knowing what is stored in a interface variable
==============================================
We know that a variable of a given interface can store any value of any type
that implements this interface. That's the good part, but what if we wanted to
retrieve a value stored in an interface variable and put it in a regular type
variable, how do we know what exact type was "wrapped" in that interface
variable?

Let's see an example to make the question clearer.

.. code-block:: go
    :linenos:

    type Element interface{}
    type List [] Element

    //...

    func main(){
        //...
        var nbr int
        elt := l[i]
        //The question is how do I convert elt to int, to assign it to nbr
        //and is the value boxed in elt even an int?
        //...
    }

So, the question is: How do I test for the type that is stored in an interface
variable?

Comma-ok type assertion
-----------------------
Go comes with a handy syntax to know whether it is possible to convert an
interface value to a given type value, it's as easy as this: ``v, ok =
elt.(T)``, where ``v`` is a variable of type ``T``, ``ok`` a boolean, and
``elt`` the interface variable.

If it is possible to convert ``elt`` to type ``T``, then ``ok`` is set to
``true``, and ``v`` is set to the result of this conversion. otherwise, ``ok``
is set to ``false``.

Let's use this comma ok type assertion in an example:

.. code-block:: go
    :linenos:

    package main

    import (
        "fmt"
        "strconv" //for conversions to and from string 
    )

    type Element interface{}
    type List [] Element

    type Person struct{
        name string
        age int
    }

    //For printng. See previous chapter.
    func (p Person) String() string{
        return "(name: " + p.name + " - age: "+strconv.Itoa(p.age)+ " years)"
    }

    func main(){
        list := make(List, 3)
        list[0] = 1 //an int
        list[1] = "Hello" //a string
        list[2] = Person{"Dennis", 70}

        for i, elt := range list{
            if v, ok := elt.(int); ok{
                fmt.Printf("list[%d] is an int and its value is %d\n", i, v) 
            } else if v, ok := elt.(string); ok{
                fmt.Printf("list[%d] is a string and its value is %s\n", i, v) 
            } else if v, ok := elt.(Person); ok{
                fmt.Printf("list[%d] is a Person and its value is %s\n", i, v) 
            } else {
                fmt.Println("list[%d] is of a different type", i) 
            }
        }
    }

Output:

.. container:: output

    | list[0] is an int and its value is 1
    | list[1] is a string and its value is Hello
    | list[2] is a Person and its value is (name: Dennis - age: 70 years)

It's that easy.

Notice the syntax we used with our ``if``\s, I hope that you still remember that
you can initialize inside an ``if``! Do you?

Yes, I know, the more we test for other types, the more the  ``if``/``else``
chain gets harder to read. And this is why they invented the type switch! 

The type switch test
--------------------
Better to show it off with an example, right? Ok, let's rewrite the previous
example. Here we Go!

.. code-block:: go
    :linenos:

    package main

    import (
        "fmt"
        "strconv" //for conversions to and from string 
    )

    type Element interface{}
    type List [] Element

    type Person struct{
        name string
        age int
    }

    //For printng. See previous chapter.
    func (p Person) String() string{
        return "(name: " + p.name + " - age: "+strconv.Itoa(p.age)+ " years)"
    }

    func main(){
        list := make(List, 3)
        list[0] = 1 //an int
        list[1] = "Hello" //a string
        list[2] = Person{"Dennis", 70}

        for i, elt := range list{
            switch v := elt.(type){
                case int:
                    fmt.Printf("list[%d] is an int and its value is %d\n", i, v) 
                case string:
                    fmt.Printf("list[%d] is a string and its value is %s\n", i, v) 
                case Person:
                    fmt.Printf("list[%d] is a Person and its value is %s\n", i, v) 
                default:
                    fmt.Println("list[%d] is of a different type", i) 
            }
        }
    }

Output:

.. container:: output

    | list[0] is an int and its value is 1
    | list[1] is a string and its value is Hello
    | list[2] is a Person and its value is (name: Dennis - age: 70 years)

Now repeat after me: The ``elt.(type)`` construct SHOULD NOT be used outside of
a switch statement! ---Can you use it elsewhere? **NO, YOU CAN NOT!** 

If you need to make a single test, use the comma-ok test. Just DON'T use
``elt.(type)`` outside of a switch statement.

Embedding interfaces
====================
What's really nice with Go is the *logic* side of its syntax. When we learnt
about anonymous fields in ``struct``\s we found it quite natural, didn't we?
Now, by applying the same logic, wouldn't it be nice to be able to embed an
interface ``iface1`` in another interface ``iface2`` so that ``iface2`` inherits
the methods in ``iface1``? I say "logic", because after all: interfaces are sets
of methods, just like structs are sets of fields. And so it is! In Go you can
put an interface type into another.

Example: Suppose that you have an indexed collections of elements, and that you
want to get the minimum, and the maximum value of this collection without
changing the elements order in this collection.

One silly but illustrative way to do this is by using the ``sort.Interface`` we
saw in the previous chapter. But then again, the function ``sort.Sort`` provided
by the sort package actually changes the input collection! We add two
methods: ``Get`` and ``Copy``.

.. code-block:: go
    :linenos:

    package main

    import (
        "fmt"
        "strconv"
        "sort"
    )

    type Person struct{
        name string
        age int
        phone string
    }

    type MinMax interface{
        sort.Interface
        Copy() MinMax
        Get(i int) interface{}
    }

    func (h Person) String() string{
        return "(name: " + h.name + " - age: "+strconv.Itoa(h.age)+ " years)"
    }

    type People []Person //People is a type of slices that contain Persons

    func (g People) Len() int{
        return len(g) 
    }

    func (g People) Less(i, j int) bool{
        if g[i].age < g[j].age {
            return true 
        }
        return false
    }

    func (g People) Swap(i, j int){
        g[i], g[j] = g[j], g[i]
    }

    func (g People) Get(i int) interface{} {return g[i]}

    func (g People) Copy() MinMax{
        c := make(People, len(g))
        copy(c, g)
        return c
    }

    func GetMinMax(C MinMax) (min, max interface{}){
        K := C.Copy()
        sort.Sort(K)
        min, max =  K.Get(0), K.Get(K.Len()-1)
        return
    }

    func main(){
        group := People{
            Person{name:"Bart", age:24},
            Person{name:"Bob", age:23},
            Person{name:"Gertrude", age:104},
            Person{name:"Paul", age:44},
            Person{name:"Sam", age:34},
            Person{name:"Jack", age:54},
            Person{name:"Martha", age:74},
            Person{name:"Leo", age:4},
        }

        //Let's print this group as it is
        fmt.Println("The unsorted group is:")
        for _, v := range group{
            fmt.Println(v) 
        }

        //Now let's get the older and the younger
        younger, older := GetMinMax(group)
        fmt.Println("\n➞ Younger is", younger)
        fmt.Println("➞ Older is ", older)

        //Let's print this group again
        fmt.Println("\nThe original group is still:")
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
    | ➞ Younger is (name: Leo - age: 4 years)
    | ➞ Older is  (name: Gertrude - age: 104 years)
    |
    | The original group is still:
    | (name: Bart - age: 24 years)
    | (name: Bob - age: 23 years)
    | (name: Gertrude - age: 104 years)
    | (name: Paul - age: 44 years)
    | (name: Sam - age: 34 years)
    | (name: Jack - age: 54 years)
    | (name: Martha - age: 74 years)
    | (name: Leo - age: 4 years)

The example is idiotic but it worked as desired. Mind you, interfaces embedding
can be very useful, you can find some interfaces embedding in some Go packages
as well. For example, the `container/heap`_ package that provides heap
operations of data collections that implement this interface:

.. code-block:: go
    :linenos:

    //heap.Interface
    type Interface interface {
        sort.Interface //embeds sort.Interface
        Push(x interface{}) //a Push method to push elements into the heap
        Pop() interface{} //a Pop elements that pops elements from the heap
    }

Another example is the `io.ReadWriter`_ interface that is a combinaison of two
interfaces: `io.Reader`_ and `io.Writer`_ both also defined in this package.

.. code-block:: go
    :linenos:

    //io.ReadWriter
    type ReadWriter interface {
        Reader
        Writer
    }

Types that implement io.ReadWriter can read and write since they implement both
interfaces Reader and Writer.

.. external links and footnotes:

.. _container/heap: http://golang.org/pkg/container/heap
.. _io.ReadWriter: http://golang.org/pkg/io/index.html#ReadWriter
.. _io.Reader: http://golang.org/pkg/io/index.html#Reader
.. _io.Writer: http://golang.org/pkg/io/index.html#Writer
