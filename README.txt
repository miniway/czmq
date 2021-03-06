.set GIT=https://github.com/zeromq/czmq
.sub 0MQ=ØMQ

# CZMQ - High-level C binding for 0MQ

## Contents

.toc

## Overview

### Scope and Goals

CZMQ has these goals:

* To wrap the 0MQ core API in semantics that are natural and lead to shorter, more readable applications.
* To hide the differences between versions of 0MQ, particularly 2.1 and 3.1.
* To provide a space for development of more sophisticated API semantics.

CZMQ grew out of concepts developed in [ØMQ - The Guide](http://zguide.zeromq.org) and [ZFL](http://zfl.zeromq.org). Until end-April 2011, CZMQ was known as *libzapi*.

[diagram]
                              +---------------+
                              |               |
                              | C application |
                              |               |
                              +-----+---+-----+
                                    |   |
                                    |   |
                       +------------+   |
                       |                |
                       v                |
  Open context    +---------+           |
  Create sockets  |         |           |    Connect, bind sockets
  Close sockets   |  CZMQ   |           |    get/set socket options
  Send/receive    |    cYEL |           |
  Multithreading  +----+----+           |
  Reactor pattern      |                |
  Hash container       +------------+   |
  List container                    |   |
  System clock                      v   v
  Close context                  +---------+
                                 |         |
                                 | libzmq  |
                                 |         |
                                 +---------+

[/diagram]

### Highlights

* Single API hides differences between 0MQ/2.1, and 0MQ/3.1.
* Work with messages as strings, individual frames, or multipart messages.
* Automatic closure of any open sockets at context termination.
* Automatic LINGER configuration of all sockets for context termination.
* Portable API for creating child threads and 0MQ pipes to talk to them.
* Simple reactor with one-off and repeated timers, and socket readers.
* System clock functions for sleeping and calculating timers.
* Easy API to get/set all socket options.
* Portable to Linux, UNIX, OS X, Windows (porting is not yet complete).
* Includes generic hash and list containers.
* Full selftests on all classes.

### Ownership and License

CZMQ is maintained by Pieter Hintjens and Mikko Koppanen (build system). Its other authors and contributors are listed in the AUTHORS file. It is held by the ZeroMQ organization at github.com.

The authors of CZMQ grant you use of this software under the terms of the GNU Lesser General Public License (LGPL). For details see the files `COPYING` and `COPYING.LESSER` in this directory.

### Contributing

CZMQ uses the [C4 (Collective Code Construction Contract)](http://rfc.zeromq.org/spec:16) process which says, "Everyone, without distinction or discrimination, SHALL have an equal right to become a Contributor under the terms of this contract".

To report an issue, use the [CZMQ issue tracker](https://github.com/zeromq/czmq/issues) at github.com.

## Using CZMQ

### Building and Installing

CZMQ uses autotools for packaging. To build from git (all example commands are for Linux):

    git clone git://github.com/zeromq/czmq.git
    cd czmq
    sh autogen.sh
    ./configure
    make all
    sudo make install
    sudo ldconfig

You will need the libtool and autotools packages. On FreeBSD, you may need to specify the default directories for configure:

    ./configure --with-libzmq=/usr/local

After building, you can run the CZMQ selftests:

    make check

### Linking with an Application

Include `czmq.h` in your application and link with libczmq. Here is a typical gcc link command:

    gcc -lczmq -lzmq myapp.c -o myapp

### API Summary

#### zctx - working with 0MQ contexts

.pull src/zctx.c@header,left

This is the class interface:

.pull include/zctx.h@interface,code

.pull src/zctx.c@discuss,left

#### zsocket - working with 0MQ sockets

.pull src/zsocket.c@header,left

This is the class interface:

.pull include/zsocket.h@interface,code

.pull src/zsocket.c@discuss,left

#### zsockopt - working with 0MQ socket options

.pull src/zsockopt.c@header,left

This is the class interface:

.pull include/zsockopt.h@interface,code

.pull src/zsockopt.c@discuss,left

#### zstr - sending and receiving strings

.pull src/zstr.c@header,left

[diagram]

           Memory                       Wire
           +-------------+---+          +---+-------------+
    Send   | S t r i n g | 0 |  ---->   | 6 | S t r i n g |
           +-------------+---+          +---+-------------+

           Wire                         Heap
           +---+-------------+          +-------------+---+
    Recv   | 6 | S t r i n g |  ---->   | S t r i n g | 0 |
           +---+-------------+          +-------------+---+

[/diagram]

This is the class interface:

.pull include/zstr.h@interface,code

.pull src/zstr.c@discuss,left

#### zfile - work with files

.pull src/zfile.c@header,left

This is the class interface:

.pull include/zfile.h@interface,code

.pull src/zfile.c@discuss,left

#### zframe - working with single message frames

.pull src/zframe.c@header,left

This is the class interface:

.pull include/zframe.h@interface,code

.pull src/zframe.c@discuss,left

#### zmsg - working with multipart messages

.pull src/zmsg.c@header,left

This is the class interface:

.pull include/zmsg.h@interface,code

.pull src/zmsg.c@discuss,left

#### zloop - event-driven reactor

.pull src/zloop.c@header,left

This is the class interface:

.pull include/zloop.h@interface,code

.pull src/zloop.c@discuss,left

#### zthread - working with system threads

.pull src/zthread.c@header,left

This is the class interface:

.pull include/zthread.h@interface,code

.pull src/zthread.c@discuss,left

#### zhash - expandable hash table container

.pull src/zhash.c@header,left

This is the class interface:

.pull include/zhash.h@interface,code

.pull src/zhash.c@discuss,left

#### zlist - singly-linked list container

.pull src/zlist.c@header,left

This is the class interface:

.pull include/zlist.h@interface,code

.pull src/zlist.c@discuss,left

#### zclock - millisecond clocks and delays

.pull src/zclock.c@header,left

This is the class interface:

.pull include/zclock.h@interface,code

.pull src/zclock.c@discuss,left

#### zmutex - wrap lightweight mutexes

.pull src/zmutex.c@header,left

This is the class interface:

.pull include/zmutex.h@interface,code

.pull src/zmutex.c@discuss,left

## Design Ideology

### The Problem with C

C has the significant advantage of being a small language that, if we take a little care with formatting and naming, can be easily interchanged between developers. Every C developer will use much the same 90% of the language. Larger languages like C++ provide powerful abstractions like STL containers but at the cost of interchange.

The huge problem with C is that any realistic application needs packages of functionality to bring the language up to the levels we expect for the 21st century. Much can be done by using external libraries but every additional library is a dependency that makes the resulting applications harder to build and port. While C itself is a highly portable language (and can be made more so by careful use of the preprocessor), most C libraries consider themselves part of the operating system, and as such do not attempt to be portable.

The answer to this, as we learned from building enterprise-level C applications at iMatix from 1995-2005, is to create our own fully portable, high-quality libraries of pre-packaged functionality, in C. Doing this right solves both the requirements of richness of the language, and of portability of the final applications.

### A Simple Class Model

C has no standard API style. It is one thing to write a useful component, but something else to provide an API that is consistent and obvious across many components. We learned from building [OpenAMQ](http://www.openamq.org), a messaging client and server of 0.5M LoC, that a consistent model for extending C makes life for the application developer much easier.

The general model is that of a class (the source package) that provides objects (in fact C structures). The application creates objects and then works with them. When done, the application destroys the object. In C, we tend to use the same name for the object as the class, when we can, and it looks like this (to take a fictitious CZMQ class):

    zregexp_t *regexp = zregexp_new (regexp_string);
    if (!regexp)
        printf ("E: invalid regular expression: %s\n", regexp_string);
    else
    if (zregexp_match (regexp, input_buffer))
        printf ("I: successful match for %s\n", input buffer);
    zregexp_destroy (&regexp);

As far as the C program is concerned, the object is a reference to a structure (not a void pointer). We pass the object reference to all methods, since this is still C. We could do weird stuff like put method addresses into the structure so that we can emulate a C++ syntax but it's not worthwhile. The goal is not to emulate an OO system, it's simply to gain consistency. The constructor returns an object reference, or NULL if it fails. The destructor nullifies the class pointer, and is idempotent.

What we aim at here is the simplest possible consistent syntax.

No model is fully consistent, and classes can define their own rules if it helps make a better result. For example:

* Some classes may not be opaque. For example, we have cases of generated serialization classes that encode and decode structures to/from binary buffers. It feels clumsy to have to use methods to access the properties of these classes.

* While every class has a new method that is the formal constructor, some methods may also act as constructors. For example, a "dup" method might take one object and return a second object.

* While every class has a destroy method that is the formal destructor, some methods may also act as destructors. For example, a method that sends an object may also destroy the object (so that ownership of any buffers can passed to background threads). Such methods take the same "pointer to a reference" argument as the destroy method.

### Naming Style

CZMQ aims for short, consistent names, following the theory that names we use most often should be shortest. Classes get one-word names, unless they are part of a family of classes in which case they may be two words, the first being the family name. Methods, similarly, get one-word names and we aim for consistency across classes (so a method that does something semantically similar in two classes will get the same name in both). So the canonical name for any method is:

    zclassname_methodname

And the reader can easily parse this without needing special syntax to separate the class name from the method name.

### Containers

After a long experiment with containers, we've decided that we need exactly two containers:

* A singly-linked list.
* A hash table using text keys.

These are zlist and zhash, respectively. Both store void pointers, with no attempt to manage the details of contained objects. You can use these containers to create lists of lists, hashes of lists, hashes of hashes, etc.

We assume that at some point we'll need to switch to a doubly-linked list.

### Portability

Creating a portable C application can be rewarding in terms of maintaining a single code base across many platforms, and keeping (expensive) system-specific knowledge separate from application developers. In most projects (like 0MQ core), there is no portability layer and application code does conditional compilation for all mixes of platforms. This leads to quite messy code.

CZMQ is a portability layer, similar to but thinner than libraries like the [Apache Portable Runtime](http://apr.apache.org) (APR).

These are the places a C application is subject to arbitrary system differences:

* Different compilers may offer slightly different variants of the C language, often lacking specific types or using neat non-portable names. Windows is a big culprit here. We solve this by 'patching' the language in czmq_prelude.h, e.g. defining int64_t on Windows.
* System header files are inconsistent, i.e. you need to include different files depending on the OS type and version. We solve this by pulling in all necessary header files in czmq_prelude.h. This is a proven brute-force approach that increases recompilation times but eliminates a major source of pain.
* System libraries are inconsistent, i.e. you need to link with different libraries depending on the OS type and version. We solve this with an external compilation tool, 'C', which detects the OS type and version (at runtime) and builds the necessary link commands.
* System functions are inconsistent, i.e. you need to call different functions depending, again, on OS type and version. We solve this by building small abstract classes that handle specific areas of functionality, and doing conditional compilation in these.

An example of the last:

    #if (defined (__UNIX__))
        pid = GetCurrentProcessId();
    #elif (defined (__WINDOWS__))
        pid = getpid ();
    #else
        pid = 0;
    #endif

CZMQ uses the GNU autotools system, so non-portable code can use the macros this defines. It can also use macros defined by the czmq_prelude.h header file.

### Technical Aspects

* *Thread safety*: the use of opaque structures is thread safe, though 0MQ applications should not share state between threads in any case.
* *Name spaces*: we prefix class names with `z`, which ensures that all exported functions are globally safe.
* *Library versioning*: we don't make any attempt to version the library at this stage. Classes are in our experience highly stable once they are built and tested, the only changes typically being added methods.
* *Performance*: for critical path processing, you may want to avoid creating and destroying classes. However on modern Linux systems the heap allocator is very fast. Individual classes can choose whether or not to nullify their data on allocation.
* *Self-testing*: every class has a `selftest` method that runs through the methods of the class. In theory, calling all selftest functions of all classes does a full unit test of the library. The `czmq_selftest` application does this.
* *Memory management*: CZMQ classes do not use any special memory management techiques to detect leaks. We've done this in the past but it makes the code relatively complex. Instead, we do memory leak testing using tools like valgrind.

## Under the Hood

### Adding a New Class

If you define a new CZMQ class `myclass` you need to:

* Write the `zmyclass.c` and `zmyclass.h` source files, in `src` and `include` respectively.
* Add`#include <zmyclass.h>` to `include/czmq.h`.
* Add the myclass header and test call to `src/czmq_selftest.c`.
* Add a reference documentation to 'doc/zmyclass.txt'.
* Add myclass to 'src/Makefile.am` and `doc/Makefile.am`.

The `bin/newclass.sh` shell script will automate these steps for you.

### Coding Style

In general the zctx class defines the style for the whole library. The overriding rules for coding style are consistency, clarity, and ease of maintenance. We use the C99 standard for syntax including principally:

* The // comment style.
* Variables definitions placed in or before the code that uses them.

So while ANSI C code might say:

    zblob_t *file_buffer;       /*  Buffer for our file */
    ... (100 lines of code)
    file_buffer = zblob_new ();
    ...

The style in CZMQ would be:

    zblob_t *file_buffer = zblob_new ();

### Assertions

We use assertions heavily to catch bad argument values. The CZMQ classes do not attempt to validate arguments and report errors; bad arguments are treated as fatal application programming errors.

We also use assertions heavily on calls to system functions that are never supposed to fail, where failure is to be treated as a fatal non-recoverable error (e.g. running out of memory).

Assertion code should always take this form:

    int rc = some_function (arguments);
    assert (rc == 0);

Rather than the side-effect form:

    assert (some_function (arguments) == 0);

Since assertions may be removed by an optimizing compiler.

### Method Styles

We aim for consistent method semantics where possible:

* new returns null if the constructor failed.
* destroy always voids the supplied reference pointer.
* dup, if defined, copies the object and returns null if the provided reference was null.

### Documentation

Man pages are generated from the class header and source files via the doc/mkman tool, and similar functionality in the gitdown tool (http://github.com/imatix/gitdown). The header file for a class must wrap its interface as follows (example is from include/zclock.h):

    //  @interface
    //  Sleep for a number of milliseconds
    void
        zclock_sleep (int msecs);

    //  Return current system clock as milliseconds
    int64_t
        zclock_time (void);

    //  Self test of this class
    int
        zclock_test (Bool verbose);
    //  @end

The source file for a class must provide documentation as follows:

    /*
    @header
    ...short explanation of class...
    @discuss
    ...longer discussion of how it works...
    @end
    */

The source file for a class then provides the self test example as follows:

    //  @selftest
    int64_t start = zclock_time ();
    zclock_sleep (10);
    assert ((zclock_time () - start) >= 10);
    //  @end

The template for man pages is in doc/mkman.

### Development

CZMQ is developed through a test-driven process that guarantees no memory violations or leaks in the code:

* Modify a class or method.
* Update the test method for that class.
* Run the 'selftest' script, which uses the Valgrind memcheck tool.
* Repeat until perfect.

### Porting CZMQ

When you try CZMQ on an OS that it's not been used on (ever, or for a while), you will hit code that does not compile. In some cases the patches are trivial, in other cases (usually when porting to Windows), the work needed to build equivalent functionality may be non-trivial. In any case, the benefit is that once ported, the functionality is available to all applications.

Before attempting to patch code for portability, please read the `czmq_prelude.h` header file. There are several typical types of changes you may need to make to get functionality working on a specific operating system:

* Defining typedefs which are missing on that specific compiler: do this in czmq_prelude.h.
* Defining macros that rename exotic library functions to more conventional names: do this in czmq_prelude.h.
* Reimplementing specific methods to use a non-standard API: this is typically needed on Windows. Do this in the relevant class, using #ifdefs to properly differentiate code for different platforms.

The canonical 'standard operating system' for all CZMQ code is Linux, gcc, POSIX. The canonical 'weird operating system' for CZMQ is Windows.

### Code Generation

We generate the zsockopt class using the mysterious but powerful GSL code generator. It's actually cool, since about 30 lines of XML are sufficient to generate 700 lines of code. Better, since many of the option data types changed in 0MQ/3.1, it's possible to completely hide the differences. To regenerate the zsockopt class, build and install GSL from https://github.com/imatix/gsl, and then:

    gsl sockopts

You may also enjoy using this same technique if you're writing bindings in other languages. See the sockopts.gsl file, this can be easily modified to produce code in whatever language interests you.

### This Document

This document is originally at README.txt and is built using [gitdown](http://github.com/imatix/gitdown).
