.. include:: common.txt

.. More *meth* please!!! ;)
More on methods
***************
We have learned how to write methods in Go, that they can be seen as functions
with an implicit first parameter that is its receiver, and we saw some
facilities that Go offers when working with pointers and receivers.

In this chapter we will see how to implement some OOP concepts using methods.
It will be fun and relatively easy.

Structs with anonymous fields
=============================
I didn't tell you about this when we studied ``struct``\s, but we actually can
declare fields without specifying names for them (just the type). It is for
this reason that we call them *anonymous fields* (the lack of a name).

When an anonymous field is a ``struct`` its fields are inserted (embedded)
into the struct containing it.

Let's see an example to help clarify this concept:

.. code-block:: go
    :linenos:

    package main
    import "fmt"

    type Human struct {
        name string
        age int
        weight int //in lb, mind you
    }

    type Student struct {
        Human //an anonymous field of type Human
        speciality string
    }

    func main() {
        //Mark is a Student, notice how we declare a literal of this type:
        mark := Student{Human{"Mark", 25, 120}, "Computer Science"}
        //Now let's access the fields:
        fmt.Println("His name is", mark.name)
        fmt.Println("His age is", mark.age)
        fmt.Println("His weight is", mark.weight)
        fmt.Println("His speciality is", mark.speciality)
        //Change his speciality
        mark.speciality = "AI"
        fmt.Println("Mark changed his speciality")
        fmt.Println("His speciality is", mark.speciality)
        //change his age. He got older
        fmt.Println("Mark become old")
        mark.age = 46
        fmt.Println("His age is", mark.age)
        //Mark gained some weight as time goes by
        fmt.Println("Mark is not an athlet anymore")
        mark.weight += 60
        fmt.Println("His weight is", mark.weight)
    }

Output:

.. container:: output

    | His name is Mark
    | His age is 25
    | His weight is 120
    | His speciality is Computer Science
    | Mark changed his speciality
    | His speciality is AI
    | Mark become old
    | His age is 46
    | Mark is not an athlet anymore
    | His weight is 180

On line 29 and 33 we were able to access and change the fields ``age`` and
``weight`` just as if they were *declared* as fields of the ``Student`` struct.
This is because fields of ``Human`` are embedded in the struct ``Student``.

Cool, isn't it? But there's more than this interpolation of fields, an anonymous
field can be used with its type as its name!

This means that ``Student`` has a field named ``Human`` that can be used as
regular field with this name.

.. code-block:: go
    :linenos:

    // We can access and modify the Human field using the embedded struct name:
    mark.Human = Human{"Marcus", 55, 220} // Nervous breakdown, he became a Nun.
    mark.Human.age -= 1 // Soon he will be back in diapers...

Anonymous fields of any type
============================
Accessing, and changing an anonymous field by name is quite useful for
anonymous fields that are not ``struct``\s. * -- Who told you that anonymous
fields must be* ``struct``\s?! * -- I didn't!*

In fact, any *named type* and pointers to named types are completely acceptable.

Let's see some anonymous action:

.. code-block:: go
    :linenos:

    package main
    import "fmt"

    type Skills []string

    type Human struct {
        name string
        age int
        weight int //in lb, mind you
    }

    type Student struct {
        Human //an anonymous field of type Human
        Skills //anonymous field for his skills
        int //we will use this int as an anonymous field for his preferred number
        speciality string
    }

    func main() {
        //Jane is a Student, look how we declare a literal of this type
        //by specifying only some of its fields. We saw this before
        jane := Student{Human:Human{"Jane", 35, 100}, speciality:"Biology"}
        //Now let's access the fields:
        fmt.Println("Her name is", jane.name)
        fmt.Println("Her age is", jane.age)
        fmt.Println("Her weight is", jane.weight)
        fmt.Println("Her speciality is", jane.speciality)
        //Let's change some anonymous fields
        jane.Skills = []string{"anatomy"}
        fmt.Println("Her skills are", jane.Skills)
        fmt.Println("She acquired two new ones")
        jane.Skills = append(jane.Skills, "physics", "golang")
        fmt.Println("Her skills now are", jane.Skills)
        //her preferred number, which is an int anonymous field
        jane.int = 3
        fmt.Println("Her preferred number is", jane.int)
    }

Output:

.. container:: output

    | Her name is Jane
    | Her age is 35
    | Her weight is 100
    | Her speciality is Biology
    | Her skills are [anatomy]
    | She acquired two new ones
    | Her skills now are [anatomy physics golang]
    | Her preferred number is 3

The anonymous field mechanism lets us *inherit* some (or even all)
of the implementation of a given type from another type or types.

Anonymous fields conflicts
==========================
What happens if ``Human`` has a field ``phone`` and ``Student`` has also a field
with the same name?

This kind of conflicts are solved in Go simply by saying that the *outer* name
*hides* the inner one. i.e. when accessing ``phone`` you are working with
``Student``'s one.

This provides a way to *override* a field that is present in the "inherited"
anonymous field by the one that we explicitely specify in our type.

If you still need to access the anonymous one, you'll have to use the type's
name syntax:

.. code-block:: go
    :linenos:

    package main
    import "fmt"

    type Human struct {
        name string
        age int
        phone string //his own mobile number
    }

    type Employee struct {
        Human //an anonymous field of type Human
        speciality string
        phone string //work's phone number
    }

    func main() {
        Bob := Employee{Human{"Bob", 34, "777-444-XXXX"}, "Designer", "333-222"}
        fmt.Println("Bob's work phone is: " Bob.phone)
        //Now we need to specify Human to access Human's phone
        fmt.Println("Bob's personal phone is: " Bob.Human.phone)
    }

Output:

.. container:: output

    | Bob's work phone is:  333-222
    | Bob's personal phone is:  777-444-XXXX

Great! So now you ask,

  "*Now, what does all this have to do with methods?*"

Attaboy, you didn't forget the main subject of the chapter. Kudos.

Methods on anonymous fields
===========================
The good surprise is that methods behave exactly like anonymous fields. If an
anonymous field implements a given method, this method will be available for the
type that is using this anonymous field.

If the ``Human`` type implements a method ``SayHi()`` that prints a greeting,
this same method is available for both ``Student`` and ``Employee``! You won't
need to write it twice for them:

.. code-block:: go
    :linenos:

    package main
    import "fmt"

    type Human struct {
        name string
        age int
        phone string //his own mobile number
    }

    type Student struct {
        Human //an anonymous field of type Human
        school string
    }

    type Employee struct {
        Human //an anonymous field of type Human
        company string
    }

    //A human method to say hi
    func (h *Human) SayHi() {
        fmt.Printf("Hi, I am %s you can call me on %s\n", h.name, h.phone)
    }

    func main() {
        mark := Student{Human{"Mark", 25, "222-222-YYYY"}, "MIT"}
        sam := Employee{Human{"Sam", 45, "111-888-XXXX"}, "Golang Inc"}

        mark.SayHi()
        sam.SayHi()
    }

Output:

.. container:: output

    | Hi, I am Mark you can call me on 222-222-YYYY
    | Hi, I am Sam you can call me on 111-888-XXXX

.. admonition:: Philosophy

    Think about it: you often need to represent *things* that share some
    *attributes* or *characteristics*, this is solved with the anonymous field
    mechanism. And when you need to represent a shared *behavior* or
    *functionality*, you can use a method on an anonymous field.

Overriding a method
===================
What if you want the ``Employee`` to tell you where he works as well?
Easy, the same rule for overriding fields on conflicts applies for methods,
and we happily exploit this fact:

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
    }

    type Employee struct {
        Human //an anonymous field of type Human
        company string
    }

    //A human method to say hi
    func (h *Human) SayHi() {
        fmt.Printf("Hi, I am %s you can call me on %s\n", h.name, h.phone)
    }

    //Employee's method overrides Human's one
    func (e *Employee) SayHi() {
        fmt.Printf("Hi, I am %s, I work at %s. Call me on %s\n", e.name,
            e.company, e.phone) //Yes you can split into 2 lines here.
    }

    func main() {
        mark := Student{Human{"Mark", 25, "222-222-YYYY"}, "MIT"}
        sam := Employee{Human{"Sam", 45, "111-888-XXXX"}, "Golang Inc"}

        mark.SayHi()
        sam.SayHi()
    }

Output:

.. container:: output

    | Hi, I am Mark you can call me on 222-222-YYYY
    | Hi, I am Sam, I work at Golang Inc. Call me on 111-888-XXXX

And... It worked like a charm!

With these simple concepts, you can now design shorter, nicer and expressive
programs. In the next chapter we will learn how to enhance this experience even
more with a new notion: Interfaces.

