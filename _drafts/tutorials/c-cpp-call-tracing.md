---
layout: post
title:  "How to trace C/C++ function calls on Linux"
author: "ihonen"
---

# The dumb approach: Do It Yourself™

You might come up with the idea of making your own call tracing macros – something along these lines:
```c++
#define ENTER_FUNCTION() printf("CALL  %s\n", __func__)
#define RETURN(expr)     printf("RET   %s\n",    __func__); return expr
#define THROW(expr)      printf("THROW %s\n",     __func__); throw expr
```

However, you'll soon find out that this approach has at least the following problems – and I'm sure there are more:

1. Obviously, you will have to *insert* the macros by hand. If you forget to insert a macro, the trace will break right then and there.
2. Likewise, you will have to *remove* the macros by hand should you ever want to get rid of them.
3. Exceptions may propagate several levels up the function call chain before they are caught, which means that you would have to write a try-catch block around every function call that may `THROW` and then `THROW` again to pass to propagate the exception to the desired exception handler.
4. If you want to trace calls to functions that reside in external libraries, you will have to modify the source code of said libraries to accommodate your tracing system.
5. Macros are evil and ought to be avoided whenever possible.

# The smart approach: make the compiler do it for you

Instrumentation means inserting extra code amidst pre-existing code by automated means. In this case, we're instrumenting the code with calls to the predefined functions `__cyg_profile_func_enter` and `__cyg_profile_func_exit`.

[GCC instrumentation options](https://gcc.gnu.org/onlinedocs/gcc/Instrumentation-Options.html)