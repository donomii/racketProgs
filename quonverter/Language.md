# The hwyl language spec

The name of the language is "hwyl", Welsh for "fun".  In general, it isn't fun.  It does, however, feature 100% interoperability with the target language.  You can call any function in the target language without needing complex declarations or a FFI, and any function in the target language can call any hwyl function, without any special calling conventions (but see the types section).

Hwyl is a simple intermediate language, similar to C and Pascal, but simpler.  It is expected that it will be targetted by other compilers, so it has a rigid, verbose syntax that is easy to parse, and easy to auto-generate.  Despite that, it is perfectly possible to write programs in hwyl, and all development of hwyl happens in hwyl.  Hwyl is self hosting.

Hwyl looks like LISP, but is very definitely not LISP.  There are no closures, no co-routines, and no other conveniences provided by modern languages.  Hwyl is very much a throwback to the 80's and earlier, where languages were simple and we still managed to write some interesting stuff.

In hwyl, you can create functions and call functions.  That's pretty much it.  Oh, you can create structures as well.  There are no objects, no higher order functions, and no HM type system.

Hwyl is the lowest common denominator across many programming languages, which is the point of its existence.

## A simple program

     [
     [includes base.qon]
     [types]

     [functions

      [void say [string greeting int count] [declare]
            [body
             [if [> count 0]
                 [then
                  [printf "%s" greeting]
                  [say greeting [sub1 count]]]
                 [else
                  [return]]]
             ]]

      [int start [] [declare]
           [body
            [say "Hello world\n" 3]
            [return 0]
            ]]
      ]]

This shows off almost all the features of hwyl.  Every program must have an includes, types, and functions section, even if that section is empty.  If it is a program and not a library, it must have a "start" function, which is the equivalent of a "main" function.  The start function does not take any arguments, but you can access the command line arguments by calling "argList".

Functions must have the following format, which is basically ansi C:

    [returnType functionName [arguments] 
    [declare variables]
    [body
        statements
    ]]

All components are required.  Empty functions, and empty statements, are not permitted (fix this?).

The current built-in statements are:

    * set - set a variable 
    * set-struct - set a member of a struct.  I will be removing this in the future, or limiting it to the "one reference" rule.
    * return - immediately return from function
    * if - see above.  Both branches are always required.   

Currently, there are no loops, you have to use recursion.  

There are only two special built-in functions
    
    * get-struct - fetch a value froma structure
    * new - allocate a new structure on the heap (stack allocation is not currently supported)


That's pretty much it, except for the types

## the types

Types currently "need further work".  Much like function names, types are passed straight to the target language unchanged, except for the basic types, which are translated into the correct types for the target language.  The basic types are ```int, float, string```.  When targetting C, these become ```int, double, char*```, when targetting go, ```int64, float64, string``, and so on.  When targetting a flexible or dynamically typed language, we don't print any types, and let the compiler
figure it out.

You can also declare structs(records).  This is the definition of the box type, that hwyl uses to wrap native types.

    [types
           [Box
            [struct
              [lis struct Box* ]
              [str string ]
              [i int ]
              [typ string ]
              [voi bool ]
              [boo bool]
              [lengt int ]
              [car struct Box* ]
              [cdr struct Box* ]
              ]]
           [box Box*]
           [Pair Box]
           [pair Box*]
           [list Box*]
           ]

All types are defined the same way:  ```(newType type-definition)```.  Normally this is very straightforwards, e.g. ```(box Box*)``` defines a new type, ```box``` that is a Box pointer: ```Box*```.  

If the type definition is not a single word, all the type modifiers are written after the new type name.  e.g.

    (myBox static struct Box)

which will be compiled to

    typedef static struct Box myBox;

or similar, based on the target language.

Note that just like functions, types are the lowest common denominator shared across all target languages.  So hwyl does not support union types, HM types, or any other 'advanced' type features.
