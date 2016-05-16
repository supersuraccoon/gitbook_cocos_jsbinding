## JSBinding Code Snippets

<!-- toc -->

### Create `cocos::XClass`

`JSClass::jsb_cocos2d_Sprite_class`

```C++
JSClass* jsb_cocos2d_Sprite_class = (JSClass *)calloc(1, sizeof(JSClass));
jsb_cocos2d_Sprite_class->name = "Sprite";
jsb_cocos2d_Sprite_class->addProperty = JS_PropertyStub;
jsb_cocos2d_Sprite_class->delProperty = JS_DeletePropertyStub;
jsb_cocos2d_Sprite_class->getProperty = JS_PropertyStub;
jsb_cocos2d_Sprite_class->setProperty = JS_StrictPropertyStub;
jsb_cocos2d_Sprite_class->enumerate = JS_EnumerateStub;
jsb_cocos2d_Sprite_class->resolve = JS_ResolveStub;
jsb_cocos2d_Sprite_class->convert = JS_ConvertStub;
jsb_cocos2d_Sprite_class->finalize = jsb_ref_finalize;
jsb_cocos2d_Sprite_class->flags = JSCLASS_HAS_RESERVED_SLOTS(2);
```


### Create `cocos::XClass::InstanceProperties`

`JSPropertySpec`

```C++
static JSPropertySpec ins_properties[] = {
    JS_PSG("__nativeObj", js_is_native_obj, JSPROP_PERMANENT | JSPROP_ENUMERATE),
    JS_PS_END
};
```


### Create `cocos::XClass::StaticProperties`

`JSPropertySpec`

```C++
JSPropertySpec st_properties = NULL;
```


### Create `cocos::XClass::InstanceFunctions`

`JSFunctionSpec`

```C++
static JSFunctionSpec ins_funcs[] = {
    // ......
    JS_FN(
        "initWithFile", 
        js_cocos2dx_Sprite_initWithFile, 
        1, JSPROP_PERMANENT | JSPROP_ENUMERATE
    ),
    JS_FN(
        "ctor", 
        js_cocos2dx_Sprite_ctor, 
        0, JSPROP_PERMANENT | JSPROP_ENUMERATE
    ),
    JS_FS_END
};
```


### Create `cocos::XClass::StaticFunctions`

`JSFunctionSpec`

```C++
JSFunctionSpec *st_funcs = NULL;
```


### Create `cocos::XPrototype`

`JSObject`

```C++
JSObject* jsb_cocos2d_Sprite_prototype = JS_InitClass(
        cx, 
        global,                                 
        parent_proto,                           // parent proto
        jsb_cocos2d_Sprite_class,               // JSClass
        js_cocos2dx_Sprite_constructor,         // constructor
        0,                                      // constructor arguments count
        ins_properties,                         // instance properies
        ins_funcs,                              // instance functions
        st_properties,                          // static properties
        st_funcs                                // static functions
);
```


### Find `cocos::XClass` typename

```C++
TypeTest<cocos::XClass> t;
js_type_class_t *typeClass = nullptr;
std::string typeName = t.s_name();
auto typeMapIter = _js_global_type_map.find(typeName);
CCASSERT(typeMapIter != _js_global_type_map.end(), "Can't find the class type!");

typeClass = typeMapIter->second;
CCASSERT(typeClass, "The value is null.");
```


### Get `NativeObject` in Binding `Function`

```c++
bool js_xxx_func(JSContext *cx, uint32_t argc, jsval *vp) {
    JS::CallArgs args = JS::CallArgsFromVp(argc, vp);
    JS::RootedObject obj(cx, args.thisv().toObjectOrNull());

    js_proxy_t *proxy = jsb_get_js_proxy(obj);
    cocos2d::XClass* cobj = (cocos2d::XClass *)(proxy ? proxy->ptr : NULL);
    if (!cobj) {
        JS_ReportError(cx, "Invalid Native Object");
        return false;
    }
    // ...
}
```


### `input` && `output` params in Binding `Function`

```C++
bool js_xxx_func(JSContext *cx, uint32_t argc, jsval *vp) {
    JS::CallArgs args = JS::CallArgsFromVp(argc, vp);
    args.get(0); // input param 1
    args.get(1); // input param 2
    args.get(2); // input param 3
    // ...
    args.rval().set(/* return_js_val */);
    return true;    // do not forget this !!!
}
```


