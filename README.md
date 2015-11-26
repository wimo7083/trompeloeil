## *Trompeloeil*

![trompeloeil logo](trompeloeil-logo.png)  [![Build Status](https://travis-ci.org/rollbear/trompeloeil.svg?branch=master)](https://travis-ci.org/rollbear/trompeloeil) [![Coverage Status](https://coveralls.io/repos/rollbear/trompeloeil/badge.svg?branch=master&service=github)](https://coveralls.io/github/rollbear/trompeloeil?branch=master)

> *trompe l'oeil* noun    (Concise Encyclopedia)  
> Style of representation in which a painted object is intended
> to deceive the viewer into believing it is the object itself...


What is it?
-----------

A header only mocking framework for C++14.

Documentation
-------------

- [Introduction](http://playfulprogramming.blogspot.com/2014/12/introducing-trompeloeil-c-mocking.html)
- [Cook Book](docs/CookBook.md)
- [FAQ](docs/FAQ.md)
- [Reference](docs/reference.md)

Also, follow up with the post on [sequencing](http://playfulprogramming.blogspot.se/2015/01/sequence-control-with-trompeloeil-c.html) for examples on how to restrict or relax allowed sequences of matching calls.  

If you want to contribute, read ACCU
[overload 125](http://accu.org/var/uploads/journals/Overload125.pdf)
to learn the internals.

Teaser
------
```Cpp
#include <trompeloeil.hpp>

class Interface
{
public:
  virtual bool foo(int, std::string& s) = 0;
  virtual bool bar(int) = 0;
  virtual bool bar(std::string) = 0;
};

class Mock : public Interface
{
public:
  MAKE_MOCK2(foo, bool(int, std::string&));
  MAKE_MOCK1(bar, bool(int));
  MAKE_MOCK1(bar, bool(std::string));
};

TEST(exercise_interface_func)
{
  using trompeloeil::_;  // wild card for matching any value
  using trompeloeil::gt; // greater-than match

  Mock m;

  int local_var = 0;

  REQUIRE_CALL(m, bar(ANY(int)))
    .LR_SIDE_EFFECT(local_var = _1)
    .RETURN(_1 > 0)
    .IN_SEQUENCE(seq1)
    .TIMES(AT_LEAST(1));

  FORBID_CALL(m, bar(0));

  REQUIRE_CALL(m, bar("word"))
    .RETURN(true)
    .IN_SEQUENCE(seq2);

  REQUIRE_CALL(m, foo(gt(2), _))
    .WITH(_2 == "")
    .IN_SEQUENCE(seq1, seq2)
    .SIDE_EFFECT(_2 = "cat")
    .RETURN(true);

  interface_func(&m);
}
```

Compiler compatibility
----------------------

Trompeloeil is known to work with:

* GCC 4.9, 5.1, 5.2
* Clang 3.5, 3.6, 3.7
* VisualStudio 2015
