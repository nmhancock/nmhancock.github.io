---
layout: post
title: Comparing J and K arrays
---

[J][1] and [K][2] are both descendants of the [APL][3] family of array
programming languages. J is generally considered the more “math-y” of the
two languages due to it’s more complex notion of array “shape”, which we
will see in greater detail later. K, on the other hand, is the underlying
language behind KDB+, a commercial, high-performance NoSQL database noted
for its [speed][4]. As I am attempting to replicate KDB’s speed in one of my own
projects, I’m interested in seeing how the two APL descendants implement APL’s
fundamental datatype, the array.

J arrays are defined by the following two lines of code:
```c
typedef long I;
typedef struct {I t,c,n,r,s[1];}* A;
```

Note that APL dialects are written in a heavily abbreviated style, and the
proponents thereof tend to carry that style into other languages, as seen above.

_An Implementation of J_ by Robert Hui, co-developer of J with Ken Iverson,
explains the struct fields as follows (p.12):

* t = type
* c = reference count
* n = number of atoms (C fundamental types – char, int, etc.)
* r = rank
* s = shape

Where the shape array, s, consists of r integers whose product equals n followed
by the actual data of A in row-major format. Note that this definition requires
the ‘payload’ of A to be long-aligned. For example, to represent the string
“Hello, World” n = 14, r = 1, and s = {14, (long) H, (long) e, … }. To represent
the 3×3 matrix consisting of one through nine in row-major order, the values
are: n = 9, r = 2, s = {3, 3, 1, 2, …}. In English we would say that a given
array A is “An s\[0\] by s\[1\] by s\[2\] by .. s\[r-1\]” array.

Though it’s not strictly relevant, I’d be remiss if I didn’t mention that the
shape and rank properties make for very effective generalized operators. The
succinctness of these languages betrays the expressive power of this particular
generalization. Look up code examples, e.g. Project Euler submissions, to learn
more.

K arrays are slightly more involved, so I’ll remove the abbreviations and add
white space for clarity: (only the v3.0 struct reproduced below).

```c
typedef struct k0 {
    signed char m,a,t; // m,a are for internal use.
    char u;
    int32_t r;
    union {
        unsigned char g;
        int16_t h;
        int32_t i;
        int64_t j;
        float e;
        double f;
        char* s;
        struct k0* k;
        struct {
            int64_t n;
            unsigned char G0[1];
        };
    };
}* K;
```

* t = type
* u = attribute flags
* r = reference count
* n, where used, is the number of elements in the list G0.
* The payload itself is contained in the union.

Note that the two structures are actually quite similar in form. Type, reference
count, and number of atoms (in the struct portion of k0’s union) are the same. K
attempts to save the 8-byte overhead of ‘n’ for scalars by wrapping the scalar
values in a union, in tune with the performance focus I mentioned previously. To
enable the previous trick, K has to modify the type style that J uses- scalar
values of a given type are negative. For example, a character array is type 5,
so a single character would be type -5. Finally, K declares their array as an
unsigned char array to avoid unnecessary bytes due to alignment. There are,
however, three conceptual differences between the array implementations.

First, the struct k0\* k pointer within the union seems to be the enabler for K’s
“mixed”, or heterogeneous, arrays. In other words, unlike J, K allows a K object
consisting of a float, an integer, and 10 3×3 matrices of shorts. In this case
the implementation appears to be a K object consisting of an array of pointers
to the other K objects. So the memory layout would be:

	k0 : [ptr1, ptr2, ptr3]
	ptr1 : [float]
	ptr2 : [integer]
	ptr3 : [n = 10, integer, integer, ...]

Which indicates a pointer dereference overhead for each access of an element in
a ‘mixed’ list. That seems pretty expensive for an array based language where
everything is cache friendly.\*

Second, the K object contains no reference to shape or rank. I’m having trouble
finding documentation online as to how K treats rank, if at all, and what that
difference means in practice. My research up to this point seems to be that K’s
behavior is similar to classic APL, versus J which is different and can be
considered an evolution by its creators.

Third are the fields m and a, marked for internal use. Since K is proprietary
and the documentation doesn’t specify what m and a are used for and none of the
open source functions use them, I have no idea what they do. I’m tempted to
believe that they’re related to K’s serialization using mmap and its ability to
operate on compressed blocks of data, but that’s just speculation on my part.

\*: For a practical example, my research indicates that K’s dictionary type is
implemented as a combination of an array of keys and an array of values. To
lookup a value given a key requires accessing the key array, which is one
pointer dereference, looking up the key, accessing the value array, another
pointer dereference, and then looking up the same index of the value array, for
a total of two pointer dereferences. A hash table requires one pointer
dereference + the cost of the hash function in contrast.

Source for K arrays: Kx Cookbook – [Interfacing with
C](http://code.kx.com/wiki/Cookbook/InterfacingWithC#The_K_object_structure)

Source for J arrays: _An Implementation of J_ by Roger K. W. Hui
	
[1]: https://en.wikipedia.org/wiki/J_%28programming_language%29 "J"
[2]: https://en.wikipedia.org/wiki/K_%28programming_language%29 "K"
[3]: https://en.wikipedia.org/wiki/APL_%28programming_language%29 "APL"
[4]: http://kparc.com/q4/readme.txt "benchmarks"

![Milau Bridge](https://images.unsplash.com/photo-1463906033650-3288c7071a7b?dpr=2&auto=format&fit=crop&w=1500&h=1000&q=80&cs=tinysrgb&crop=)
Photo credit: [Luca Onniboni](https://unsplash.com/@lucaonniboni "Luca
Omniboni")
