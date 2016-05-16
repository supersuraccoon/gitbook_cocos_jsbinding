## SpiderMonkey Fundamentals

<!-- toc -->

### SpiderMonkey

-   Mozilla's `JavaScript` engine written in C/C++
-   Compiles and executes scripts containing `JavaScript` statements and functions. 
-   Handles memory allocation for the objects needed to execute scripts, and cleans up—garbage collects—objects no longer needs.

### Runtimes
`JSRuntime`

-   The space in which the JavaScript variables, objects, scripts, and contexts allocated. 
-   `JSContext` and object lives within a `JSRuntime` and cannot travel to other runtimes or be shared across runtimes. 
-   Most applications only need one runtime.

`C++`  `Cocos`

```C++
/* 
    ScriptingCore.h
*/
JSRuntime *_rt;

/* 
    ScriptingCore.cpp
*/
_rt = JS_NewRuntime(8L * 1024L * 1024L);
JS_SetGCParameter(_rt, JSGC_MAX_BYTES, 0xffffffff);
JS_SetTrustedPrincipals(_rt, &shellTrustedPrincipals);
JS_SetSecurityCallbacks(_rt, &securityCallbacks);
JS_SetNativeStackQuota(_rt, JSB_MAX_STACK_QUOTA);
JS::RuntimeOptionsRef(_rt).setIon(true);
JS::RuntimeOptionsRef(_rt).setBaseline(true);
// ......
```


### Contexts
`JSContext`

-   Do things involving JavaScript code and objects. 
    -   Compile and execute scripts
    -   Get and set object properties
    -   Call JavaScript functions
    -   Convert JavaScript data from one type to another
    -   Create objects
-   **JSAPI** functions often require a `JSContext *` as the argument
-   Represents the **execution of JS code**. 
-   Contains a **JS stack** and is associated with a runtime. 

`C++`  `Cocos`

``` C++
/* 
    ScriptingCore.h
*/
JSContext *_cx;

/* 
    ScriptingCore.cpp
*/
_cx = JS_NewContext(_rt, 8192);
```


### Global Objects
`mozilla::Maybe<JS::PersistentRootedObject>`

-   Contains all the classes, functions, and variables available for JavaScript code to use. 
-   **JSAPI** applications have full control over what global properties scripts can see. 
-   Application starts out by creating an object and populating it with the standard JavaScript classes, like `Array` and `Object`. Then adds custom classes, functions, and variables.

`C++`  `Cocos`

``` C++
/* 
    ScriptingCore.h
*/
mozilla::Maybe<JS::PersistentRootedObject> _global;

/* 
    ScriptingCore.cpp
*/
_global.construct(_cx);
_global.ref() = NewGlobalObject(_cx);
JSAutoCompartment ac(_cx, _global.ref());
js::SetDefaultObjectForContext(_cx, _global.ref());
```


### Compartments

`JSCompartments`

-   `JSContexts` are control, `JSCompartments` are data.

-   `JSCompartment`  is a **memory space** that `objects` and other `GCthings` are stored within.

-   `JSContext` is associated with a single `JSCompartment` at all times.

-   `JSContext` is "running inside" the  associated `JSCompartment`. 

-   Any `object` created with the `JSContext` will be **physically stored** within the context's current `JSCompartment`. 

#### Cross-Compartment Call

To access data in another `JSCompartment`.

-   `JSContext` must first "enter" that other `JSCompartment`. 

-   A `JSContext` has a field `cx->compartment` that gives the current compartment. 

-   To make a `cross-compartment call`, `cx->compartment` is updated to the new compartment. 

Only objects in the current compartment can be accessed, so to access an object in a different compartment, this containing compartment has to be entered first. Compartments have to be entered and left in LIFO order.

 `JSAutoCompartment`guarantees that by automatically entering the given compartment and leaving it upon getting out of scope:

`C++`

```C++
void foo(JSContext *cx, JObject *obj) {
    // in some compartment 'c'
    {
        JSAutoCompartment ac(cx, obj);  // constructor enters
        // in the compartment of 'obj'
    }                                 // destructor leaves
    // back in compartment 'c'
}
```

