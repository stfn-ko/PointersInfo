# C/C++ Pointers

## Table of contents

["Reviewing the basic terminology"](#reviewing-the-basic-terminology)
["A pointer scenario"](#a-pointer-scenario)
["Dereferencing the pointer"](#dereferencing-the-pointer)
["Dereferencing and accessing a structure data member"](#dereferencing-and-accessing-a-structure-data-member)
["Multi byte data types"](#multi-byte-data-types)
["Pointers to dynamically allocated memory"](#pointers-to-dynamically-allocated-memory)
["Losing and leaking addresses"](#losing-and-leaking-addresses)
["C++ smart pointers"](#c-smart-pointers)
["Null pointers"](#null-pointers)
["More about memory addresses and why you probably dont need to know"](#more-about-memory-addresses-and-why-you-probably-dont-need-to-know)

> _Author: Tony Delroy | Editor: Stephan Kolontay_

## Reviewing the basic terminology

It's usually good enough - unless you're programming assembly - to envisage a **pointer** containing a numeric memory address, with 1 referring to the second byte in the process's memory, 2 the third, 3 the fourth and so on....

- What happened to 0 and the first byte? Well, we'll get to that later - see null pointers below.
- For a more accurate definition of what pointers store, and how memory and addresses relate, see ["More about memory addresses, and why you probably don't need to know"](#more-about-memory-addresses-and-why-you-probably-dont-need-to-know) at the end.

When you want to access the data/value in the memory that the pointer points to - the contents of the address with that numerical index - then you dereference the pointer.

Different computer languages have different notations to tell the compiler or interpreter that you're now interested in the pointed-to object's (current) value - I focus below on C and C++.

## A pointer scenario

Consider in C, given a pointer such as p below...

```C++
const char* p = "abc";
```

...four bytes with the numerical values used to encode the letters 'a', 'b', 'c', and a 0 byte to denote the end of the textual data, are stored somewhere in memory and the numerical address of that data is stored in `p`. This way C encodes text in memory is known as ["ASCIIZ"](https://en.wikipedia.org/wiki/Null-terminated_string).

For example, if the string literal happened to be at address 0x1000 and `p` a 32-bit pointer at 0x2000, the memory content would be:

```C++
Memory Address (hex)    Variable name    Contents
1000                                     'a' == 97 (ASCII)
1001                                     'b' == 98
1002                                     'c' == 99
1003                                     0
...
2000-2003               p                1000 hex
```

Note that there is no variable name/identifier for address 0x1000, but we can indirectly refer to the string literal using a pointer storing its address: `p`.

## Dereferencing the pointer

To refer to the characters p points to, we dereference `p` using one of these notations (again, for C):

```C++
assert(*p == 'a');  // The first character at address p will be 'a'
assert(p[1] == 'b'); // p[1] actually dereferences a pointer created by adding
                     // p and 1 times the size of the things to which p points:
                     // In this case they're char which are 1 byte in C...
assert(*(p + 1) == 'b');  // Another notation for p[1]
```

You can also move pointers through the pointed-to data, dereferencing them as you go:

```C++
++p;  // Increment p so it's now 0x1001
assert(*p == 'b');  // p == 0x1001 which is where the 'b' is...
```

If you have some data that can be written to, then you can do things like this:

```C++
int x = 2;
int* p_x = &x;  // Put the address of the x variable into the pointer p_x
*p_x = 4;       // Change the memory at the address in p_x to be 4
assert(x == 4); // Check x is now 4
```

Above, you must have known at compile time that you would need a variable called `x`, and the code asks the compiler to arrange where it should be stored, ensuring the address will be available via `&x`.

## Dereferencing and accessing a structure data member

In C, if you have a variable that is a pointer to a structure with data members, you can access those members using the `->` dereferencing operator:

```C++
typedef struct X { int i_; double d_; } X;
X x;
X* p = &x;
p->d_ = 3.14159;  // Dereference and access data member x.d_
(*p).d_ *= -1;    // Another equivalent notation for accessing x.d_
```

## Multi-byte data types

To use a pointer, a computer program also needs some insight into the type of data that is being pointed at - if that data type needs more than one byte to represent, then the pointer normally points to the lowest-numbered byte in the data.

So, looking at a slightly more complex example:

```C++
double sizes[] = { 10.3, 13.4, 11.2, 19.4 };
double* p = sizes;
assert(p[0] == 10.3);  // Knows to look at all the bytes in the first double value
assert(p[1] == 13.4);  // Actually looks at bytes from address p + 1 * sizeof(double)
                       // (sizeof(double) is almost always eight bytes)
++p;                   // Advance p by sizeof(double)
assert(*p == 13.4);    // The double at memory beginning at address p has value 13.4
*(p + 2) = 29.8;       // Change sizes[3] from 19.4 to 29.8
                       // Note earlier ++p and + 2 here => sizes[3]
```

## Pointers to dynamically allocated memory

Sometimes you don't know how much memory you'll need until your program is running and sees what data is thrown at it... then you can dynamically allocate memory using malloc. It is common practice to store the address in a pointer...

```C++
int* p = (int*)malloc(sizeof(int)); // Get some memory somewhere...
*p = 10;            // Dereference the pointer to the memory, then write a value in
fn(*p);             // Call a function, passing it the value at address p
(*p) += 3;          // Change the value, adding 3 to it
free(p);            // Release the memory back to the heap allocation library
```

In C++, memory allocation is normally done with the new operator, and deallocation with `delete`:

```C++
int* p = new int(10); // Memory for one int with initial value 10
delete p;

p = new int[10];      // Memory for ten ints with unspecified initial value
delete[] p;

p = new int[10]();    // Memory for ten ints that are value initialised (to 0)
delete[] p;
```

See also ["C++ smart pointers"](#c-smart-pointers) below.

## Losing and leaking addresses

Often a pointer may be the only indication of where some data or buffer exists in memory. If ongoing use of that data/buffer is needed, or the ability to call `free()` or `delete` to avoid leaking the memory, then the programmer must operate on a copy of the pointer...

```C++
const char* p = asprintf("name: %s", name);  // Common but non-Standard printf-on-heap

// Replace non-printable characters with underscores....
for (const char* q = p; *q; ++q)
    if (!isprint(*q))
        *q = '_';

printf("%s\n", p); // Only q was modified
free(p);
```

...or carefully orchestrate reversal of any changes...

```C++
const size_t n = ...;
p += n;
...
p -= n;  // Restore earlier value...
free(p);
```

## C++ smart pointers

In C++, it's best practice to use ["smart pointer"](http://en.wikipedia.org/wiki/Smart_pointer) objects to store and manage the pointers, automatically deallocating them when the smart pointers' destructors run. Since C++11 the Standard Library provides two, [`unique_ptr`](http://en.cppreference.com/w/cpp/memory/unique_ptr) for when there's a single owner for an allocated object...

```C++
{
    std::unique_ptr<T> p{new T(42, "meaning")};
    call_a_function(p);
    // The function above might throw, so delete here is unreliable, but...
} // p's destructor's guaranteed to run "here", calling delete
```

...and [`shared_ptr`](http://en.cppreference.com/w/cpp/memory/shared_ptr) for share ownership (using ["reference counting"](http://en.wikipedia.org/wiki/Reference_counting))...

```C++
{
    auto p = std::make_shared<T>(3.14, "pi");
    number_storage1.may_add(p); // Might copy p into its container
    number_storage2.may_add(p); // Might copy p into its container    } // p's destructor will only delete the T if neither may_add copied it
```

## Null pointers

In C, `NULL` and `0` - and additionally in C++ `nullptr` - can be used to indicate that a pointer doesn't currently hold the memory address of a variable, and shouldn't be dereferenced or used in pointer arithmetic. For example:

```C++
const char* p_filename = NULL; // Or "= 0", or "= nullptr" in C++
int c;
while ((c = getopt(argc, argv, "f:")) != -1)
    switch (c) {
      case f: p_filename = optarg; break;
    }
if (p_filename)  // Only NULL converts to false
    ...   // Only get here if -f flag specified
```

In C and C++, just as inbuilt numeric types don't necessarily default to `0`, nor `bools` to `false`, pointers are not always set to `NULL`. All these are set to `0`/`false`/`NULL` when they're static variables or (C++ only) direct or indirect member variables of static objects or their bases, or undergo zero initialisation (e.g. `new T();` and `new T(x, y, z);` perform zero-initialisation on T's members including pointers, whereas `new T;` does not).

Further, when you assign `0`, `NULL` and `nullptr` to a pointer the bits in the pointer are not necessarily all reset: the pointer may not contain "0" at the hardware level, or refer to address 0 in your virtual address space. The compiler is allowed to store something else there if it has reason to, but whatever it does - if you come along and compare the pointer to `0`, `NULL`, `nullptr` or another pointer that was assigned any of those, the comparison must work as expected. So, below the source code at the compiler level, `NULL` is potentially a bit "magical" in the C and C++ languages...

## More about memory addresses, and why you probably don't need to know

More strictly, initialised pointers store a bit-pattern identifying either `NULL` or a (often ["virtual"](http://en.wikipedia.org/wiki/Virtual_address_space)) memory address.

The simple case is where this is a numeric offset into the process's entire virtual address space; in more complex cases the pointer may be relative to some specific memory area, which the CPU may select based on CPU "segment" registers or some manner of segment id encoded in the bit-pattern, and/or looking in different places depending on the machine code instructions using the address.

For example, an `int*` properly initialised to point to an int variable might - after casting to a `float*` - access memory in "GPU" memory quite distinct from the memory where the int variable is, then once cast to and used as a function pointer it might point into further distinct memory holding machine opcodes for the program (with the numeric value of the `int* ` effectively a random, invalid pointer within these other memory regions).

3GL programming languages like C and C++ tend to hide this complexity, such that:

- If the compiler gives you a pointer to a variable or function, you can dereference it freely (as long as the variable's not destructed/deallocated meanwhile) and it's the compiler's problem whether e.g. a particular CPU segment register needs to be restored beforehand, or a distinct machine code instruction used

- If you get a pointer to an element in an array, you can use pointer arithmetic to move anywhere else in the array, or even to form an address one-past-the-end of the array that's legal to compare with other pointers to elements in the array (or that have similarly been moved by pointer arithmetic to the same one-past-the-end value); again in C and C++, it's up to the compiler to ensure this "just works"

- Specific OS functions, e.g. shared memory mapping, may give you pointers, and they'll "just work" within the range of addresses that makes sense for them

- Attempts to move legal pointers beyond these boundaries, or to cast arbitrary numbers to pointers, or use pointers cast to unrelated types, typically have undefined behaviour, so should be avoided in higher level libraries and applications, but code for OSes, device drivers, etc. may need to rely on behaviour left undefined by the C or C++ Standard, that is nevertheless well defined by their specific implementation or hardware.
