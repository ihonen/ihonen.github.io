---
layout: post
title:  "Linux: trace and log C/C++ function calls"
author: "ihonen"
---

## The problem

A couple of weeks ago I spent a good couple of hours trying to figure out why and where a process in the embedded system we are developing crashes. I thought next time it would be nice not to have to spend so much time on such a trivial task, so it occurred to me that I should build a proper function call tracing system that I can turn on at will and that shows me exactly what functions were called, when and where.

Function call tracing can serve several purposes:

1. If your program crashes, you'll be able to see what function was called last and get a pretty good idea as to where to start looking for the problem.
2. If you enable tracing, you can potentially make better sense of how a program you don't know works.
3. It's just plain cool. 8)

As is the case with everything always, tracing also has its downsides:

1. It is costly in terms of performance. Exactly how costly depends on the project and the particular methods used.
2. It is *extremely* costly in terms of storage space if you store the trace in a log file.
3. Things like function and class names in the trace can give away details about intellectual property, in addition to exposing information about the system to a potential attacker.

In any case, we want to build a tracing system, so that's exactly what we're going to do.

## The dumb approach: Do It Yourself™

If you're anything like me and have a tendency to reinvent the wheel – badly – instead of actually learning to use the tools at your disposal, you may come up with the idea of making your own tracing macros. Perhaps something along these lines:
```c++
#define ENTER()      printf("CALL  %s\n", __func__)
#define RETURN(expr) printf("RET   %s\n", __func__); return expr
#define THROW(expr)  printf("THROW %s\n", __func__); throw expr

int foo()
{
    ENTER();

    // Do stuff.
    if (stuff.bad)
        THROW(std::runtime_error("Stuff is bad"));

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

In layman's terms, instrumentation means inserting extra code in the middle of pre-existing code by automated means. In this case, we're instrumenting the code with calls to the following functions that are predefined by GCC

```c++
// These functions are called whenever a function is entered/exited,
// even if the exit happens due to an exception.
void __cyg_profile_func_enter(void* this_fn, void* call_site);
void __cyg_profile_func_exit(void* this_fn, void* call_site);
```

so that when we have a function like

```c++
int foo()
{
    // Do stuff.
    return 42;
}
```

it conceptually becomes:

```c++
int foo()
{
    __cyg_profile_func_enter();
    // Do stuff.
    __cyg_profile_func_exit();
    return 42;
}
```

## References

* [GCC instrumentation options](https://gcc.gnu.org/onlinedocs/gcc/Instrumentation-Options.html)
* [Instrumentation (Wikipedia)](https://en.wikipedia.org/wiki/Instrumentation)
