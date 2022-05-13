---
layout: post
title:  "Tracing and logging C/C++ function calls on Linux"
author: "ihonen"
---

## The problem



## The dumb approach: Do It Yourself™

If you're anything like me and have a tendency to reinvent the wheel – badly – you may come up with the idea of making your own tracing macros. Perhaps something along these lines:
```c++
#define ENTER()      printf("CALL  %s\n", __func__)
#define RETURN(expr) printf("RET   %s\n", __func__); return expr
#define THROW(expr)  printf("THROW %s\n", __func__); throw expr

void foo()
{
    ENTER();
    // do stuff
    if (stuff.bad)
    {
        THROW(std::runtime_error("Stuff is bad"));
    }
    RETURN(42);
}
```

However, you'll soon find out that this approach has *at least* the following problems – and I'm sure there are more:

1. Obviously, you will have to *insert* the macros by hand. If you forget to insert a macro, the trace will break off right then and there.
2. Likewise, you will have to *remove* the macros by hand should you ever want to rid your code of the extra clutter they bring.
3. You can't correctly trace the propagation of exceptions up the call chain without catching the exception and re-`THROW`ing it in every function along the way. (This limitation obviously only applies to C++, as there are no exceptions in C.)
4. You can't trace calls to functions in external libraries without modifying the source code of those libraries.
5. Macros can cause a whole bunch of problems and ought to be avoided whenever possible.

So, yeah... This is not the way you'll want to do it.

## The smart approach: code instrumentation

A far more elegant solution to the problem is making use of *code instrumentation*.

In colloquial terms, instrumentation means inserting extra code in the middle of pre-existing code by automated means. In this case, we're instrumenting the code with calls to the following functions that are predefined by GCC:

```c++
// These functions are called whenever a function is entered/exited, even if the
// exit happens due to an exception.
void __cyg_profile_func_enter(void* this_fn, void* call_site);
void __cyg_profile_func_exit(void* this_fn, void* call_site);
```

## References

* [GCC instrumentation options](https://gcc.gnu.org/onlinedocs/gcc/Instrumentation-Options.html)
* [Instrumentation (Wikipedia)](https://en.wikipedia.org/wiki/Instrumentation)
