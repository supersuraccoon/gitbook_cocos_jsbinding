## SpiderMonkey Garbage Collection

<!-- toc -->

### Garbage Collection

-   JavaScript code implicitly allocates memory for objects, strings, variables, and so on. 

-   Garbage collection is the process by which the JavaScript engine detects when those pieces of memory are no longer reachable.
-   The application must be very careful to ensure that any values it needs are GC-reachable. 
-   Any object you leave lying around will be destroyed if you don't tell the JSAPI you're still using it. 

-   The application should take steps to reduce the performance impact of garbage collection.

### Keeping objects alive

-   If you just need the value to remain reachable for the duration of a `JSNative` call, store it in`*rval` or an element of the `argv` array. The values stored in those locations are always reachable.

-   If a custom object needs certain values to remain in memory, just store the values in `properties` of the object.

-   If a `function` creates new objects, strings, or numbers, it can use `JS_EnterLocalRootScope` and`JS_LeaveLocalRootScope` to keep those values alive for the duration of the function.

-   To keep a value alive permanently, store it in a `GC root`. ( `JS_Add*Root`)   


### GC thing pointer

Refer to memory allocated and managed by the SpiderMonkey garbage collector. 

-   JS::Value
-   JSObject*
-   JSString*
-   JSScript*
-   jsid

### GC things on the stack

-   GC thing pointers that are parameters to a function must be wrapped in `JS::Handle<T>`

-   A JS::Handle<T> is a reference to a `JS::Rooted<T>`

-   All `JS::Handle<T>` are created implicitly by referencing a `JS::Rooted<T>`

-   It is **not valid** to create a `JS::Handle<T>` manually

-   `JS::Handle` is immutable: it can only ever refer to the `JS::Rooted<T>` that it was created for

-   Use `JS::Handle<T>` for all function parameters taking GC thing pointers (except out-parameters)

-   All GC thing pointers used as an out-parameter must be wrapped in a `JS::MutableHandle<T>`.


### GC things on the heap

-   Must be wrapped in a `JS::Heap<T>`

-   The only exception to this is if they are added as roots with the` JS_Add<T>Root` functions or `JS::PersistentRooted` class, but don't do this unless it's really necessary.

-   `JS::Heap<T>` doesn't require a` JSContext*`, and can be constructed with or without an initial value parameter. 


### Summary

-   Use `JS::Rooted<T>` for local variables **on the stack**.
-   Use `JS::Handle<T>` for **function parameters**.
-   Use `JS::MutableHandle<T>` for **function out-parameters**.
-   Use an **implicit cast** from `JS::Rooted<T>` to get a `JS::Handle<T>`.
-   Use an **explicit address-of-operator** on `JS::Rooted<T>` to get a `JS::MutableHandle<T>`.
-   Return **raw pointers** from functions.
-   Use `JS::Rooted<T>` fields when possible for aggregates, otherwise use an `AutoRooter`.
-   Use `JS::Heap<T>` members for **heap data**.
-   Do not use `JS::Rooted<T>`, `JS::Handle<T>` or `JS::MutableHandle<T>` on the heap.
-   Do not use `JS::Rooted<T>` for **function parameters**.
-   Use `JS::PersistentRooted<T>` for things that are **alive until the process exits**.

