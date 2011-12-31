.. include:: common.txt

Methods: a taste of OOP
***********************
In the previous chapter, we saw some nice things to do with functions as values
that can be assigned to variables, passed to and returned from other functions.
We finished with the fact that we actually can use functions as ``struct``
fields.


Today, we'll see a kind of an *extrapolation* of functions, that is functions
with a *receiver*, which are called **methods**.

What is a method?
=================
Suppose that you have a ``struct`` representing a rectangle. And you want *this*
rectangle to *tell* you *its own* area.

The way we'd do this with functions would be something like this:

.. code-block:: go
    :linenos:

    package main
    import "fmt"

    type Rectangle struct {
        width, height float64
    }

    func area(r Rectangle) float64 {
        return r.width*r.height
    }

    func main() {
        r1 := Rectangle{12, 2}
        r2 := Rectangle{9, 4}
        fmt.Println("Area of r1 is: ", area(r1))
        fmt.Println("Area of r2 is: ", area(r2))
    }

Output:

.. container:: output

    | Area of r1 is:  24
    | Area of r2 is:  36

This works as expected, but in the example above, the function ``area`` is not
*part* of the ``Rectangle`` type. It *expects* a ``Rectangle`` parameter as
its input.

*Yes, so what?* You'd say. No problem, it's just if you decide to add circles
and triangles and other polygons to your program, and you want to compute their
areas, you'd have to write different functions *with different names* for a
*functionality* or a *characteristic* that is, after all, the *same*.

You'd have to write: ``area_rectangle``, ``area_circle``, ``area_triangle``...

And this is not elegant. Because the *area* of a *shape* is a *characteristic*
of this shape. It should be a part of it, *belong* to it, just like its other
fields.

And this leads us to methods: A method is function that is bound or attached to
a given type. Its syntax is the same as a traditional function except that we
specify a *receiver* of this type just after the keyword ``func``.

In the words of `Rob Pike`_:

  "*A method is a function with an implicit first argument, called a receiver.*"

``func (ReceiverType r) func_name (parameters) (results)``

Let's illustrate this with an example:

.. code-block:: go
    :linenos:

    package main
    import ("fmt"; "math") //Hey! Look how we used a semi-colon! Neat technique!

    type Rectangle struct {
        width, height float64
    }

    type Circle struct {
        radius float64
    }

    /*
     Notice how we specified a receiver of type Rectangle to this method.
     Notice also how this methods -in this case- doesn't need input parameters
     because the data it need is part of its receiver r
    */
    func (r Rectangle) area() float64 {
        return r.width * r.height  //using fields of the receiver
    }

    // Another method with the SAME name but with a different receiver.
    func (c Circle) area() float64 {
        return c.radius * c.radius * math.Pi
    }

    func main() {
        r1 := Rectangle{12, 2}
        r2 := Rectangle{9, 4}
        c1 := Circle{10}
        c2 := Circle{25}
        //Now look how we call our methods.
        fmt.Println("Area of r1 is: ", r1.area())
        fmt.Println("Area of r2 is: ", r2.area())
        fmt.Println("Area of c1 is: ", c1.area())
        fmt.Println("Area of c2 is: ", c2.area())
    }

Output:

.. container:: output

    | Area of r1 is:  24
    | Area of r2 is:  36
    | Area of c1 is:  314.1592653589793
    | Area of c2 is:  1963.4954084936207

A few things to note about methods:

- Methods of different receivers are different methods even if they share the
  name.
- A method has access to its receiver's fields (data).
- A method is called with the dot notation like ``struct`` fields.

So? Are methods applicable only for ``struct`` types? The anwser is No. In fact,
you can write methods for any *named* type that you define, that is not a
pointer:

.. code-block:: go
    :linenos:

    package main
    import "fmt"

    //We define two new types
    type SliceOfints []int
    type AgesByNames map[string]int

    func (s SliceOfints) sum() int {
        sum := 0
        for _, value := range s {
            sum += value
        }
        return sum
    }

    func (people AgesByNames) older() string {
        a := 0
        n := ""
        for key, value := range people {
            if value > a {
                a = value
                n = key
            }
        }
        return n
    }

    func main() {
        s := SliceOfints {1, 2, 3, 4, 5}
        folks := AgesByNames {
            "Bob": 36,
            "Mike": 44,
            "Jane": 30,
            "Popey": 100, //look at this comma. when it's the last and
        } //the brace is here, the comma above is obligatory.

        fmt.Println("The sum of ints in the slice s is: ", s.sum())
        fmt.Println("The older in the map folks is:", folks.older())
    }

If you missed the comma remark, go back and re-read the code carefully.

Output:

.. container:: output

    | The sum of ints in the slice s is:  15
    | The older in the map folks is: Popey

*Now, wait a minute!* (You say this with your best Bill Cosby's impression)
*What is this "named types" thing that you're telling me now?*
Sorry, my bad. I didn't need to tell you before.
And I didn't want to distract you with this *detail* back then.

It's in fact easy. You can define new types as much as you want.
``struct`` is in fact a specific case of this syntax.
Look back, we actually used this in the previous chapter!

You can create aliases for built-in and composite types with the following
syntax:

``type type_name type_literal``

Examples:

.. code-block:: go
    :linenos:

    //ages is an alias for int
    type ages int
    //money is an alias for float32
    type money float32
    //we define months as a map of strings and their associated number of days
    type months map[string]int
    //m is a variable of type months
    m := months {
        "January":31,
        "February":28,
        ...
        "December":31,
    }

See? It's actually easy, and it can be handy to give more meaning to your
code, by giving names to complicated composite -- or even simple -- types.

Back to our methods.

So, yes, you can define methods for any named type, even if it's an alias to a
pre-declared type. Needless to say that you can define as many methods, for any
given named type, as you want.

Let's see a more advanced example, and we will discuss some details of it just
after.

Tis the story of a set of colored boxes. They have widths, heights, depths
and colors of course! We want to find out the color of the biggest box, and
eventually paint them all black (Because you know... I see a red box, and I want
it painted black...)

Here we *Go*!

.. Very punny... :p

.. _colored-boxes-example:

.. code-block:: go
    :linenos:
    :emphasize-lines: 25, 43

    package main
    import "fmt"

    const(
        WHITE = iota
        BLACK
        BLUE
        RED
        YELLOW
    )

    type Color byte

    type Box struct {
        width, height, depth float64
        color Color
    }

    type BoxList []Box //a slice of boxes

    func (b Box) Volume() float64 {
        return b.width * b.height * b.depth
    }

    func (b *Box) SetColor(c Color) {
        b.color = c
    }

    func (bl BoxList) BiggestsColor() Color {
        v := 0.00
        k := Color(WHITE) //initialize it to something
        for _, b := range bl {
            if b.Volume() > v {
                v = b.Volume()
                k = b.color
            }
        }
        return k
    }

    func (bl BoxList) PaintItBlack() {
        for i, _ := range bl {
            bl[i].SetColor(BLACK)
        }
    }

    func (c Color) String() string {
        strings := []string {"WHITE", "BLACK", "BLUE", "RED", "YELLOW"}
        return strings[c]
    }

    func main() {
        boxes := BoxList {
            Box{4, 4, 4, RED},
            Box{10, 10, 1, YELLOW},
            Box{1, 1, 20, BLACK},
            Box{10, 10, 1, BLUE},
            Box{10, 30, 1, WHITE},
            Box{20, 20, 20, YELLOW},
        }

        fmt.Printf("We have %d boxes in our set\n", len(boxes))
        fmt.Println("The volume of the first one is", boxes[0].Volume(), "cm³")
        fmt.Println("The color of the last one is", boxes[len(boxes)-1].color.String())
        fmt.Println("The biggest one is", boxes.BiggestsColor().String())
        //I want it painted black
        fmt.Println("Let's paint them all black")
        boxes.PaintItBlack()
        fmt.Println("The color of the second one is", boxes[1].color.String())
        //obviously, it will be... BLACK!
        fmt.Println("Obviously, now, the biggest one is", boxes.BiggestsColor().String())
    }

Output:

.. container:: output

    | We have 6 boxes in our set
    | The volume of the first one is 64 cm³
    | The color of the last one is YELLOW
    | The biggest one is WHITE
    | Let's paint them all black
    | The color of the second one is BLACK
    | Obviously, now, the biggest one is BLACK

So we defined some ``const``\s with consecutive values using the ``iota`` idiom
to represent some colors.

And then we declared some types:

1. ``Color`` which is an alias to ``byte``.
2. ``Box`` struct to represent a box, it has three dimensions and a color.
3. ``BoxList`` which is a slice of ``Box``.

Simple and straightforward.

Then we wrote some methods for these types:

- ``Volume()`` with a receiver of type ``Box`` that returns the volume of the
  received box.
- ``SetColor(c Color)`` sets its receiver's color to ``c``.
- ``BiggestsColor()`` with a receiver of type ``BoxList`` returns the color of
  the ``Box`` with the biggest volume that exists within the slice.
- ``PaintItBlack()`` with a receiver of type ``BoxList`` sets the colors of all
  ``Box``\es in the slice to BLACK.
- ``String()`` a method with a receiver of type ``Color`` returns a string
  representation of this color.

All this is simple. For real. We *translated* our vision of the problem into
*things* that have methods that describe and implement a *behavior*.

Pointer receivers
=================
Now, look at line 25 that I highlighted on purpose. The receiver is a pointer to
Box! Yes, you can use ``*Box`` too. The restriction with methods is that the
type ``Box`` itself (or any receiver's type) shouldn't be a pointer.

Why did we use a pointer? You have 10 seconds to think about it, and then read
on the next paragraph. I'll start counting:

  10, 9, 8...

Ready?

Ok! Let's find out if you were correct.
We used a pointer because we needed the ``SetColor`` method to be able to
change the value of the field 'color' of its receiver. If we did not use a
pointer, then the method would recieve a *copy* of the receiver ``b``
(passed by value) and hence the changes that it will make will affect the copy
only, not the original.

Think of the receiver as a parameter that the method has in input, and
ensure that you understand and remember the difference between
:ref:`passing by value and reference<value-reference>`.

Structs and pointers simplification
-----------------------------------
Again with the method ``SetColor``, intelligent readers
(You are one of them, this I know!)
would say that we should have written ``(*b).color = c`` instead of ``b.color =
c``, since we need to dereference the pointer ``b`` to access the field color.

This is true! In fact, both forms are accepted because Go knows that you want
to access the fields of the value pointed to by the pointer (since a pointer has
no notion of fields) so it assumes that you wanted ``(*b)`` and it simplifies
this for you. Look Ma, Magic!

Even more simplification
------------------------
Experienced readers will also say:

  "On line 43 where we call ``SetColor`` on ``bl[i]``,
  shouldn't it be ``(&bl[i]).SetColor(BLACK)`` instead?
  Since ``SetColor`` expects a pointer of type ``*Box`` and not a value of
  type ``Box``?"

This is also quite true! Both forms are accepted. Go automatically does
the conversion for you because it *knows* what type the method expects as a
receiver.

In other words:

  If a method ``M`` expects a receiver of type ``*T``, you can call the method
  on a variable ``V`` of type ``T`` without passing it as ``&V`` to ``M``.

Similarly:

  If a method ``M`` expects a receiver of type ``T``, you can call the method
  on a variable ``P`` of type ``*T`` without passing it as ``*P`` to ``M``.

Example:

.. code-block:: go
    :linenos:

    package main
    import "fmt"

    type Number int

    //method inc has a receiver of type pointer to Number
    func (n *Number) inc() {
        *n++
    }

    //method print has a receiver of type Number
    func (n Number) print() {
        fmt.Println("The number is equal to", n)
    }

    func main() {

        i := Number(10) //say that i is of type Number and is equal to 10
        fmt.Println("i is equal to", i)

        fmt.Println("Let's increment it twice")
        i.inc() //same as (&i).inc() method expects a pointer, but that's okay
        fmt.Println("i is equal to", i)
        (&i).inc() //this also works as expected
        fmt.Println("i is equal to", i)

        p := &i //p is a pointer to i

        fmt.Println("Let's print it twice")
        p.print() //same as (*p).print() method expects a value, but that's okay
        i.print() //this also works as expected
    }

Output:

.. container:: output

    | i is equal to 10
    | Let's increment it twice
    | i is equal to 11
    | i is equal to 12
    | Let's print it twice
    | The number is equal to 12
    | The number is equal to 12

So don't worry, Go knows the type of a receiver, and knowing this it simplifies
by accepting ``V.M()`` as a shorthand of ``(&V).M()`` and ``P.M()`` as a
shorthand for ``(*P).M()``.

Anyone who has done much C/C++ programming will have realized at this point the
world of pain that Go saves us from by simply using sane assumptions.

Well, well, well... I know these pointers/values matters hurt heads, take a
break, go out, have a good coffee, and in the next chapter we will see some cool
things to do with methods.

.. external links and footnotes:

.. _Rob Pike: http://groups.google.com/group/golang-nuts/msg/d05976004eb9f72b
