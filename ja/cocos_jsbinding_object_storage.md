##  COCOS JSBinding Objects Storage

<!-- toc -->

### `NativeObject` 

eg. `cocos::Sprite`

-   Added to `_native_js_global_map`
-   Added to `_js_native_global_map`

### `JSObject` 

eg. `cc.Sprite`

-   Added to `_native_js_global_map`
-   Added to `_js_native_global_map`
-   Added to `NamedObjectRoot` `GC Thing`

### _native_js_global_map

```C++
/*
    Key: NativeObject
    
    void* as the native object. usually cocos2d::Ref
    eg. cocos2d::Sprite
*/
<void*, js_proxy_t*> _native_js_global_map;
```

### _js_native_global_map

```C++
/*
    Key : JSObject
    
    JSObject* as the JS object, as a heap object
    eg. cc.Sprite
*/ 
<JSObject*, js_proxy_t*> _js_native_global_map;
```

### js_proxy_t

```C++
// js_proxy_t
typedef struct js_proxy {
    /** 
        the native object. usually cocos2d::Ref 
        eg. cocos2d::Sprite
    */
    void *ptr;
    /** 
        the JS object, as a heap object 
        eg. cc.Sprite 
    */
    JS::Heap<JSObject*> obj;
    /** 
        the raw pointer to JSObject 
    */
    JSObject* _jsobj;
} js_proxy_t;
```

### _js_global_type_map

```C++
/*
    Key : Cocos Class name
    
    How to get: 
    TypeTest<T> t;
    std::string typeName = t.s_name();
  
    eg:
    TypeTest<cocos2d::Sprite> t;
    t.s_name() => "N7cocos2d6SpriteE"
*/
<std::string, js_type_class_t*> _js_global_type_map;
```

### js_type_class

``` C++
typedef struct js_type_class {
    JSClass *jsclass;
    mozilla::Maybe<JS::PersistentRootedObject> proto;
    mozilla::Maybe<JS::PersistentRootedObject> parentProto;
} js_type_class_t;
```

<br />

### Helper APIs

```C++
js_proxy_t* jsb_new_proxy(void* nativeObj, JS::HandleObject jsObj);
js_proxy_t* jsb_get_native_proxy(void* nativeObj);
js_proxy_t* jsb_get_js_proxy(JSObject* jsObj);
void jsb_remove_proxy(js_proxy_t* proxy);

void jsb_ref_init(JSContext* cx, JS::Heap<JSObject*> *obj, cocos2d::Ref* ref, const char* debug);
void jsb_ref_autoreleased_init(JSContext* cx, JS::Heap<JSObject*> *obj, cocos2d::Ref* ref, const char* debug);
JSObject* jsb_ref_create_jsobject(JSContext *cx, cocos2d::Ref *ref, js_type_class_t *typeClass, const char* debug);
JSObject* jsb_ref_autoreleased_create_jsobject(JSContext *cx, cocos2d::Ref *ref, js_type_class_t *typeClass, const char* debug);
JSObject* jsb_ref_get_or_create_jsobject(JSContext *cx, cocos2d::Ref *ref, js_type_class_t *typeClass, const char* debug);
JSObject* jsb_ref_autoreleased_get_or_create_jsobject(JSContext *cx, cocos2d::Ref *ref, js_type_class_t *typeClass, const char* debug);

template<class T>
inline js_proxy_t *js_get_or_create_proxy(JSContext *cx, T *native_obj)
template<class T>
JSObject* js_get_or_create_jsobject(JSContext *cx, typename std::enable_if<!std::is_base_of<cocos2d::Ref,T>::value,T>::type *native_obj)
  
  
template <class T>
inline js_type_class_t *js_get_type_from_native(T* native_obj)
```

