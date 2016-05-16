## SpiderMonkey JSAPI Code Snippets

<!-- toc -->

### Define `Primitive types`

`javascript`

```javascript
var v;

v = 0;
v = 0.5;
v = someString;
v = null;
v = undefined;
v = false;
```

`C++`

```C++
JS::RootedValue v(cx);
JS::RootedString someString(cx, ...);

v.setInt32(0);           // or: v = JS::Int32Value(0);
v.setDouble(0.5);        // or: v = JS::DoubleValue(0.5);
v.setString(someString); // or: v = JS::StringValue(someString);
v.setNull();             // or: v = JS::NullValue();
v.setUndefined();        // or: v = JS::UndefinedValue();
v.setBoolean(false);     // or: v = JS::BooleanValue(false);
```


### Check `Primitive Types`

`javascript`

```javascript
var v = computeSomeValue();

var isString = typeof v === "string";
var isNumber = typeof v === "number";
var isNull = v === null;
var isBoolean = typeof v === "boolean";
var isObject = typeof v === "object" && v !== null;

```

`C++`

```C++
JS::RootedValue v(cx, ComputeSomeValue());

bool isString = v.isString();
bool isNumber = v.isNumber();
bool isInt32 = v.isInt32();
bool isNull = v.isNull();
bool isBoolean = v.isBoolean();
bool isObject = v.isObject();
```


### Define `Function`

`javascript`

```javascript
function justForFun() {
    return null;
}
```

`C++`

```C++
bool justForFun(JSContext *cx, unsigned argc, JS::Value *vp)
{
    JS::CallArgs args = JS::CallArgsFromVp(argc, vp);
    args.rval().setNull();
    return true;
}
```


### Creat `Array Object`

`javascript`

```javascript
var x = [];  // or "x = Array()", or "x = new Array"
```

`C++`

```C++
JS::RootedObject x(cx, JS_NewArrayObject(cx, 0));
if (!x)
    return false;
```


### Creat `Plain Object`

`javascript`

```javascript
var x = {};  // or "x = Object()", or "x = new Object"
```

`C++`

```C++
JS::RootedObject x(cx, JS_NewPlainObject(cx));
// or JS_NewObject(cx, JS::NullPtr(), JS::NullPtr(), JS::NullPtr());
if (!x)
    return false;
```


### New `Object`

`javascript`

```javascript
var person = new Person("Dave", 24);
```

`C++`

```C++
/* Step 1 - Get the value of Person and check that it is an object. */
JS::RootedValue constructor_val(cx);
if (!JS_GetProperty(cx, JS_GetGlobalObject(cx), "Person", &constructor_val))
    return false;
if (!constructor_val.isObject()) {
    JS_ReportError(cx, "Person is not a constructor");
    return false;
}
JS::RootedObject constructor(cx, &constructor_val.toObject());

/* Step 2 - Set up the arguments. */
JS::RootedString name_str(cx, JS_NewStringCopyZ(cx, "Dave"));
if (!name_str)
    return false;

JS::AutoValueArray<2> args(cx);
args[0].setString(name_str);
args[1].setInt32(24);

/* Step 3 - Call |new Person(...args)|, passing the arguments. */
JS::RootedObject obj(cx, JS_New(cx, constructor, args));
if (!obj)
    return false;
```


### Call Global `Function`

`javascript`

```javascript
var r = foo();  // where f is a global function
```

`C++`

```C++
JS::RootedValue r(cx);
if (!JS_CallFunctionName(cx, JS_GetGlobalObject(cx), "foo", 0, NULL, &r))
   return false;
```


### Return value in `Function`

`javascript`

```javascript
function justForFun() {
    return 23;
    // return 3.14159;
}
```

`C++`

```C++
bool justForFun(JSContext *cx, unsigned argc, JS::Value *vp)
{
    JS::CallArgs args = JS::CallArgsFromVp(argc, vp);
    args.rval().setInt32(23);
    // args.rval().setDouble(3.14159);
    return true;
}
```


### Get `Object` propert

`javascript`

```javascript
var x = y.myprop;
```

`C++`

```c++
JS::RootedValue x(cx);

assert(y.isObject());
JS::RootedObject yobj(cx, &y.toObject());
if (!JS_GetProperty(cx, yobj, "myprop", &x))
    return false;
```


### Set `Object` property

`javascipt`

```javascript
y.myprop = x;
```

`C++`

```C++
assert(y.isObject());
JS::RootedObject yobj(cx, &y.toObject());
if (!JS_SetProperty(cx, yobj, "myprop", &x))
    return false;
```


### Check `Object` property

`javascript`

```javascript
if ("myprop" in y) {
    // then do something
}
```

`C++`

```C++
bool found;

assert(y.isObject());
JS::RootedObject yobj(cx, &y.toObject());
if (!JS_HasProperty(cx, yobj, "myprop", &found))
    return false;
if (found) {
    // then do something
}
```


### Register `namespace` (`object`) 

`C++` `Cocos`

```C++
JS::RootedValue nsval(cx);
JS::RootedObject ns(cx);
if (nsval == JSVAL_VOID /* nsval.isNullOrUndefined() */) {
  ns.set(JS_NewObject(cx, NULL, JS::NullPtr(), JS::NullPtr()));
  nsval = OBJECT_TO_JSVAL(ns);
  JS_SetProperty(cx, global, "cc", nsval);
}
```

`javascript` `Binding`

``` javascript
cc.log(cc); // console: [object Object]
```


### Register Global `Functions`

`C++` `Cocos`

``` c++
JS_DefineFunction(cx, global, "require", func_impl_in_cpp, 1, JSPROP_PERMANENT);
```

`javascript`

``` javascript
require("path/to/fileA.js");
require("path/to/fileB.js");
require("path/to/fileC.js");
```


### Complie `RootedScript` from `.jsc`

`C++` `Cocos`

```C++
JS::RootedScript script(cx);
JS::RootedObject obj(cx, global);
JSAutoCompartment ac(cx, global);

Data data = getDataFromFile("xxx.jsc");
script = JS_DecodeScript(cx, data.getBytes(), static_cast<uint32_t>(data.getSize()), nullptr);
```


### Complie `RootedScript` from `.js`

`C++` `Cocos`

```C++
JS::RootedScript script(cx);
JS::RootedObject obj(cx, global);
JSAutoCompartment ac(cx, global);

std::string fullPath = fullPathForFilename("xxx.js");

JS::CompileOptions op(cx);
op.setUTF8(true);
op.setFileAndLine(fullPath.c_str(), 1);

JS::Compile(cx, obj, op, fullPath.c_str(), &script);
```


### Run Complied `RootedScript`

`C++` `Cocos`

``` C++
JS::RootedScript script(cx);
JS::RootedValue rval(cx);
JSAutoCompartment ac(cx, global);
JS_ExecuteScript(cx, global, script, &rval);
```


### Get `JSObject` proto name

`C++`

```C++
JS::RootedObject jsrTargetObject(cx, jsObject);
JS::RootedObject proto(cx);

if (JS_GetPrototype(cx, jsrTargetObject, &proto)) {
  CCLOG("%s", JS_GetClass(proto)->name);
}
```


### Iterate `JSObject` properties

`C++`

```C++
JS::RootedObject jsrTargetObject(cx, jsObject);
JS::RootedObject jsrPropertyIterator(
    cx, JS_NewPropertyIterator(cx, jsrTargetObject)
);
while (true) {
    JS::RootedId jsrIdp(cx);
    JS::RootedValue jsrKey(cx);
    // iter
    if (!JS_NextProperty(cx, jsrPropertyIterator, jsrIdp.address()) ||
        !JS_IdToValue(cx, jsrIdp, &jsrKey)) {
        CCLOG("JS_NextProperty || JS_IdToValue failed !");
        break;
    }
    if (jsrKey.isNullOrUndefined()) {
        CCLOG("End");
        break;
    }
    // get property
    if (jsrKey.isInt32()) {
        JS::RootedValue value(cx);
        JS_GetPropertyById(cx, jsrTargetObject, jsrIdp, &value);
        CCLOG("Key: %d - value: %d", jsrKey.toInt32(), value.toInt32());
    }
    else if (jsrKey.isString()) {
        // ...
    }
    // ...
}
```


### Iterate `JS::RootedValue ` Array

`C++`

```C++
if (JS_IsArrayObject(cx, value)) {
    JS::RootedObject jsrObjectTemp(cx, &value.toObject());
    uint32_t length;
    JS_GetArrayLength(cx, jsrObjectTemp, &length);
    for( uint32_t i = 0; i < length; i ++) {
        JS::RootedValue valarg(cx);
        JS_GetElement(cx, jsrObjectTemp, i, &valarg);
        if (valarg.isInt32()) {
            CCLOG("value: %d", valarg.toInt32());
        }
        else if (valarg.isDouble()) {
            // ...
        }
        // ...
    }
}
```

