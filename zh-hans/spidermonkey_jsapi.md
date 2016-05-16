## SpiderMonkey JSAPI

<!-- toc -->

### JSAPI

The **JSAPI** is the C API for the SpiderMonkey `JavaScript` engine.

### JS::Value / jsval

```C++
namespace JS { class Value; }
typedef JS::Value jsval;
```

-   **Contain** `JavaScript` values of any type 
    -   `number`
    -   `string`
    -   `boolean`
    -   reference to object 
        -   `Object`
        -   `Array`
        -   `Date`
        -   `Function`
        -   `null`
        -   `undefined`
-   `JS::Value` and `jsval` are the same type, `jsval` is the old name


### JSClass

A JSClass acts as a vtable for JS objects that allows JSAPI clients to control various aspects of the behavior of an object like property lookup.

```C++
struct JSClass {
    const char          *name;
    uint32_t            flags;

    // Mandatory function pointer members.
    JSPropertyOp        addProperty;
    JSDeletePropertyOp  delProperty;
    JSPropertyOp        getProperty;
    JSStrictPropertyOp  setProperty;
    JSEnumerateOp       enumerate;
    JSResolveOp         resolve;
    JSConvertOp         convert;

    // Optional members (may be null).
    JSFinalizeOp        finalize;
    JSCheckAccessOp     checkAccess;
    JSNative            call;
    JSHasInstanceOp     hasInstance;
    JSNative            construct;
    JSTraceOp           trace;

    void                *reserved[42];
};
```


### JSObject

```C++
class JSObject : public js::gc::Cell {/*...*/};
```

-   The type of JavaScript objects in the JSAPI.

#### prototype

Most objects have a `prototype`. ( `JS_GetPrototype` )

-   An object inherits properties, including methods, from its prototype.

#### parent

Most objects have a `parent`. ( `JS_GetParent` )

#### own properties

A property of an object that is not inherited from its `prototype`. 

Almost every object can have any number of its `own properties`.

Each property has a name, a getter, a setter, and `property attributes`. ( `JS_GetPropertyAttributes` )

Most properties also have a `stored value`.

##### Stored value

The `stored value` of an object property is its last known value.

-   `JS_DefineProperty` allows the application to specify a property's initial stored value.

-   `JS_LookupProperty` fetches a property's stored value without triggering its getter.

#### JSClass

Every object is associated with a`JSClass`.

### JSString

Represents a primitive JavaScript `string` in the JSAPI.

An array of `char16_t` characters and a length.

### JSNative

```C++
typedef bool
(* JSNative)(JSContext *cx, unsigned argc, JS::Value *vp);
```
The type of many JSAPI callbacks. 

Each `JSNative` has the same signature, regardless of what arguments it expects to receive from JavaScript.

