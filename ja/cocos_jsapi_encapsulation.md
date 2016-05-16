## COCOS JSAPI Encapsulation

<!-- toc -->

### Data conversion

#### To Native

```C++
bool jsval_to_int32( JSContext *cx, JS::HandleValue vp, int32_t *ret );
bool jsval_to_uint16( JSContext *cx, JS::HandleValue vp, uint16_t *ret );
bool jsval_to_long( JSContext *cx, JS::HandleValue vp, long *out);
bool jsval_to_long_long(JSContext *cx, JS::HandleValue v, long long* ret);
bool jsval_to_ccpoint(JSContext *cx, JS::HandleValue v, cocos2d::Point* ret);
bool jsval_to_ccrect(JSContext *cx, JS::HandleValue v, cocos2d::Rect* ret);
bool jsval_to_ccsize(JSContext *cx, JS::HandleValue v, cocos2d::Size* ret);
bool jsval_to_cccolor4b(JSContext *cx, JS::HandleValue v, cocos2d::Color4B* ret);
bool jsval_to_ccarray(JSContext* cx, JS::HandleValue v, cocos2d::__Array** ret);
bool jsval_to_ccdictionary(JSContext* cx, JS::HandleValue v, cocos2d::__Dictionary** ret);
// ......
```


#### From Native

```c++
jsval int32_to_jsval( JSContext *cx, int32_t l);
jsval long_to_jsval( JSContext *cx, long number );
jsval long_long_to_jsval(JSContext* cx, long long v);
jsval std_string_to_jsval(JSContext* cx, const std::string& v);
jsval c_string_to_jsval(JSContext* cx, const char* v, size_t length = -1);
jsval ccpoint_to_jsval(JSContext* cx, const cocos2d::Point& v);
jsval ccrect_to_jsval(JSContext* cx, const cocos2d::Rect& v);
jsval ccsize_to_jsval(JSContext* cx, const cocos2d::Size& v);
jsval cccolor4b_to_jsval(JSContext* cx, const cocos2d::Color4B& v);
jsval ccdictionary_to_jsval(JSContext* cx, cocos2d::__Dictionary *dict);
jsval ccarray_to_jsval(JSContext* cx, cocos2d::__Array *arr);
// ......
```


### Wrapper

#### JSStringWrapper

A simple utility to avoid mem leaking when using JSString

```C++
class JSStringWrapper
{
public:
    JSStringWrapper();
    JSStringWrapper(JSString* str, JSContext* cx = NULL);
    JSStringWrapper(jsval val, JSContext* cx = NULL);
    ~JSStringWrapper();
    void set(jsval val, JSContext* cx);
    void set(JSString* str, JSContext* cx);
    const char* get();
private:
    const char* _buffer;
};
```


#### JSFunctionWrapper

Wraps a function and `this` object

```C++
class JSFunctionWrapper
{
public:
    JSFunctionWrapper(JSContext* cx, JS::HandleObject jsthis, JS::HandleValue fval);
    ~JSFunctionWrapper();

    bool invoke(unsigned int argc, jsval *argv, JS::MutableHandleValue rval);
private:
    JSContext *_cx;
    mozilla::Maybe<JS::PersistentRootedObject> _jsthis;
    mozilla::Maybe<JS::PersistentRootedValue> _fval;
};
```


####  JSCallbackWrapper

```C++
class JSCallbackWrapper: public cocos2d::Ref {
public:
    JSCallbackWrapper();
    virtual ~JSCallbackWrapper();
    void setJSCallbackFunc(JS::HandleValue callback);
    void setJSCallbackThis(JS::HandleValue thisObj);
    void setJSExtraData(JS::HandleValue data);
    
    const jsval getJSCallbackFunc() const;
    const jsval getJSCallbackThis() const;
    const jsval getJSExtraData() const;
protected:
    mozilla::Maybe<JS::PersistentRootedValue> _jsCallback;
    mozilla::Maybe<JS::PersistentRootedValue> _jsThisObj;
    mozilla::Maybe<JS::PersistentRootedValue> _extraData;
};
```


#### JSScheduleWrapper

```C++
class JSScheduleWrapper: public JSCallbackWrapper {
public:
    JSScheduleWrapper();
    virtual ~JSScheduleWrapper();

    static void setTargetForSchedule(
        JS::HandleValue sched, JSScheduleWrapper *target
    );
    static void setTargetForJSObject(
        JS::HandleObject jsTargetObj, JSScheduleWrapper *target
    );
    // ...
protected:
    Ref* _pTarget;
    mozilla::Maybe<JS::PersistentRootedObject> _pPureJSTarget;
    int _priority;
    bool _isUpdateSchedule;
};
```


#### JSTouchDelegate

```C++
class JSTouchDelegate: public cocos2d::Ref
{
public:
    JSTouchDelegate();
    ~JSTouchDelegate();
    
    static void setDelegateForJSObject(
        JSObject* pJSObj, JSTouchDelegate* pDelegate
    );
    void setJSObject(JS::HandleObject obj);
    void registerStandardDelegate(int priority);
    void registerTargetedDelegate(int priority, bool swallowsTouches);
    // ...
private:
    mozilla::Maybe<JS::PersistentRootedObject> _obj;
    typedef std::unordered_map<JSObject*, JSTouchDelegate*> TouchDelegateMap;
    typedef std::pair<JSObject*, JSTouchDelegate*> TouchDelegatePair;
    static TouchDelegateMap sTouchDelegateMap;
    cocos2d::EventListenerTouchOneByOne*  _touchListenerOneByOne;
    cocos2d::EventListenerTouchAllAtOnce* _touchListenerAllAtOnce;
};
```

