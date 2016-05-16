## SpiderMonkey Rooting API

<!-- toc -->

### JS::Rooted`<T>`

-   Declares a variable of type T, whose value is always rooted.
-   May be automatically coerced to a JS::Handle`<T>`
-   Should be used whenever a local variable's value may be held live across a call which can trigger a GC.

```C++
template <typename T>
class MOZ_STACK_CLASS Rooted : public js::RootedBase<T>
{/**/};
```


### JS::Handle`<T>`

-   A const reference to a JS::Rooted`<T>`
-   Functions which take GC things or values as arguments and need to root those arguments should generally use handles for those arguments and avoid any explicit rooting.
    -   When several such functions call each other then redundant rooting of multiple copies of the GC thing can be avoided
    -   If the caller does not pass a rooted value a compile error will be generated, which is quicker and easier to fix than when relying on a separate rooting analysis.

```C++
template <typename T>
class MOZ_NONHEAP_CLASS Handle : public js::HandleBase<T>
{/**/}
```


### JS::MutableHandle`<T>`

-   A non-const reference to JS::Rooted`<T>`
-   Used in the same way as JS::Handle`<T>` and includes a `set(const T &v)` method to allow updating the value of the referenced JS::Rooted`<T>`.

```C++
template <typename T>
class MOZ_STACK_CLASS MutableHandle : public js::MutableHandleBase<T>
{/**/}
```


### typedef

```C++
template <typename T> class Handle;
template <typename T> class MutableHandle;
template <typename T> class Rooted;

typedef Handle<Value>               HandleValue;
typedef MutableHandle<Value>        MutableHandleValue;
typedef Rooted<JS::Value>           RootedValue;
```

