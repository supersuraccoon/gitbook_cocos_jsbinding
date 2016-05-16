## COCOS JSBinding Workflow

<!-- toc -->

### Setup JSB Enviroment

``` C++
ScriptingCore* sc = ScriptingCore::getInstance();
sc->addRegisterCallback(register_all_cocos2dx);
sc->addRegisterCallback(register_cocos2dx_js_core);
sc->addRegisterCallback(jsb_register_system);
// ......
sc->start();    
sc->runScript("jsb_boot.js");
```

#### `initRegister`

Add `registerDefaultClasses` for later `Class` && `Functions` registeration

```C++
ScriptingCore::ScriptingCore() {
    initRegister();
}
void ScriptingCore::initRegister() {
    this->addRegisterCallback(registerDefaultClasses);
}
```


#### `addRegisterCallback` 

Add `Native cocos Class` to list for later `Class` && `Functions` registeration

``` C++
// ScriptingCore.cpp
void ScriptingCore::addRegisterCallback(sc_register_sth callback) {
    registrationList.push_back(callback);
}

// sc_register_sth
typedef void (*sc_register_sth)(JSContext* cx, JS::HandleObject global);

// registrationList
static std::vector<sc_register_sth> registrationList;
```

#### `createGlobalContext`

Setup SpiderMonkey Enviroment

```C++
void ScriptingCore::createGlobalContext() {
    _rt = JS_NewRuntime(8L * 1024L * 1024L);
    // ...
    _cx = JS_NewContext(_rt, 8192);
    // ...
    _global.construct(_cx);
    _global.ref() = NewGlobalObject(_cx);
    // ...
}
```

#### `jsb_prepare.js` 

Register some `Functions` in `javascript`

```javascript
// ...
var cc = cc || {};
var jsb = jsb || {};
cc.defineGetterSetter = function (proto, prop, getter, setter) {/* ... */};
var ClassManager = {/* ... */};
cc.Class = function() {/* ... */};
cc.Class.extend = function (prop) {/* ... */};
// ...
```

#### Register `registrationList`

``` C++
std::vector<sc_register_sth>::iterator it;
for (it = registrationList.begin(); it != registrationList.end(); it++) {
    sc_register_sth callback = *it;
    callback(_cx, _global.ref());
}
```

#### Register `registerDefaultClasses`

```C++
void registerDefaultClasses(JSContext* cx, JS::HandleObject global) {
    // ...
}
```

##### Register `cc` namespace

```C++
JS::RootedValue nsval(cx);
JS::RootedObject ns(cx);
ns.set(JS_NewObject(cx, NULL, JS::NullPtr(), JS::NullPtr()));
nsval = OBJECT_TO_JSVAL(ns);
JS_SetProperty(cx, global, "cc", nsval);
```

##### Register global `Functions`

To use in `javascript`

```C++
JS_DefineFunction(cx, global, "require", /* ... */);
JS_DefineFunction(cx, global, "log", /* ... */);
JS_DefineFunction(cx, global, "forceGC", /* ... */);
// ...
```

#### Register `register_all_cocos2dx`

```C++
void register_all_cocos2dx(JSContext* cx, JS::HandleObject obj) {
    JS::RootedObject ns(cx);
    get_or_create_js_obj(cx, obj, "cc", &ns);
    // ...
    js_register_cocos2dx_Node(cx, ns);
    js_register_cocos2dx_Sprite(cx, ns);
    js_register_cocos2dx_Action(cx, ns);
    // ...
}
```

##### Register `js_register_cocos2dx_Node`

Register Native `cocos::Node Class` with  `Functions` and `Properties`

```C++
void js_register_cocos2dx_Node(JSContext *cx, JS::HandleObject global) {
    jsb_cocos2d_Node_class = (JSClass *)calloc(1, sizeof(JSClass));
    jsb_cocos2d_Node_class->name = "Node";
    // ...
    static JSFunctionSpec funcs[] = {
        JS_FN("addChild", js_cocos2dx_Node_addChild, /* ... */),
        JS_FN("removeChild", js_cocos2dx_Node_removeChild, /* ... */),
        // ...
        JS_FS_END
    };
    // ...          
    jsb_cocos2d_Node_prototype = JS_InitClass(/* ... */);
    // ...
}
```

Then we can use `cc.Node` in  `javascript`

``` javascript
var node = new cc.Node();
node.addChild(/* ... */);
```

And the `Bind Native Function` will be called:

`js_cocos2dx_Node_constructor`

`js_cocos2dx_Node_addChild`

##### Add to `_js_global_type_map`

Add the `cocos::Node` class type to `_js_global_type_map` for later `Object` creation.

```C++
JS::RootedObject proto(cx, jsb_cocos2d_Node_prototype);
jsb_register_class<cocos2d::Node>(
    cx, jsb_cocos2d_Node_class, proto, JS::NullPtr()
);
```


#### Register Other Native `Classes`


#### `jsb_boot.js`

Prepare boot enviroment in `javascript`


### Create Cocos Object

```javascript
// javascript
var sprite = new cc.Sprite("xxx.png");
```

#### Call `new cc.Sprite("xxx.png")` from `javascript`


#### Trigger `ctor` in  `C++` 

```C++
static bool js_cocos2dx_Sprite_ctor(/**/)
```

##### Create `NativeObject` `cocos::Sprite` 

```C++
cocos2d::Sprite *nobj = new (std::nothrow) cocos2d::Sprite();
```

##### Create `JSObject`

```C++
// jsb_ref_create_jsobject
JS::RootedObject proto(cx, typeClass->proto.ref());
JS::RootedObject parent(cx, typeClass->parentProto.ref());
JS::RootedObject js_obj(cx, JS_NewObject(cx, typeClass->jsclass, proto, parent));
```

##### Bind `NativeObject` with `JSObject`

```C++
JS::RootedObject jsobj(
    cx, jsb_ref_create_jsobject(cx, cobj, typeClass, "cocos2d::Sprite")
);
// jsb_ref_create_jsobject
js_proxy_t* newproxy = jsb_new_proxy(ref, js_obj);
```

#####  Add `JSObject` to `GC Root`

```C++
// jsb_ref_create_jsobject
JS::AddNamedObjectRoot(cx, js_obj, "cocos2d::Sprite");
```

#####  Return created `Object` to `javascript`

```C++
args.rval().set(OBJECT_TO_JSVAL(jsobj));
```

##### Call `_ctor` in `javascript` if any from `C++`

```C++
bool isFound = false;
if (JS_HasProperty(cx, obj, "_ctor", &isFound) && isFound)
  executeFunctionWithOwner(OBJECT_TO_JSVAL(obj), "_ctor", args);
```

##### jsb_create_apis.js

``` javascript
// jsb_create_apis.js
_p = cc.Sprite.prototype;
_p._ctor = function(fileName, rect) {
    // ......
    this.initWithFile(fileName);
};
```


#### Trigger `initWithFile` in  `C++`

```C++
bool js_cocos2dx_Sprite_initWithFile(/**/)
```

##### Try to find the `NativeObject` (`cobj`) bind with the `JSObject`

```C++
cocos2d::Sprite* cobj = nullptr;

JS::CallArgs args = JS::CallArgsFromVp(argc, vp);
JS::RootedObject obj(cx);
obj.set(args.thisv().toObjectOrNull());

js_proxy_t *proxy = jsb_get_js_proxy(obj);
cobj = (cocos2d::Sprite *)(proxy ? proxy->ptr : nullptr);
JSB_PRECONDITION2( cobj, cx, false, "js_cocos2dx_Sprite_initWithFile : Invalid Native Object");
```

#####  Call `initWithFile` on `NativeObject` with `param`

```C++
std::string arg0;
ok &= jsval_to_std_string(cx, args.get(0), &arg0);
if (!ok) { ok = true; break; }
bool ret = cobj->initWithFile(arg0); // arg0 ==> "xxx.png"
```

### Remove Cocos Object

```javascript
// javascript
sprite.removeFromParentAndCleanup();
```

#### Call `removeFromParentAndCleanup` from `javascript` 


#### Trigger `removeFromParentAndCleanup` in `C++`

```C++
bool js_cocos2dx_Node_removeFromParentAndCleanup(/**/)
```

##### Try to find the `NativeObject` bind to the `JSObject` when created

```C++
cocos2d::Node* cobj = nullptr;

JS::CallArgs args = JS::CallArgsFromVp(argc, vp);
JS::RootedObject obj(cx);
obj.set(args.thisv().toObjectOrNull());

js_proxy_t *proxy = jsb_get_js_proxy(obj);
cobj = (cocos2d::Node *)(proxy ? proxy->ptr : nullptr);
JSB_PRECONDITION2( cobj, cx, false, "js_cocos2dx_Node_removeFromParentAndCleanup : Invalid Native Object");
```

##### Call `removeFromParent` on `NativeObject` with `param`

```C++
cobj->removeFromParent();
```

##### `C++` do the `remove ` stuffs

##### Trigger `cocos::Ref`  `destructor`

```C++
Ref::~Ref() {
    // ...
    ScriptEngineManager::getInstance()
        ->getScriptEngine()
        ->removeScriptObjectByObject(this);
}
```

##### Trigger `removeScriptObjectByObject` in `C++`

```C++
void ScriptingCore::removeScriptObjectByObject(cocos2d::Ref* nativeObj)
```

##### Find the `JSObject` bind to the `NativeObject`

```C++
auto proxy = jsb_get_native_proxy(nativeObj);
```

##### Remove the `JSObject` from `GC Root`

```C++
JS::RemoveObjectRoot(cx, &proxy->obj);
```

##### Remove the `proxy`

```C++
jsb_remove_proxy(proxy);
```

### Event Register in Cocos

```javascript
// javascript
cc.eventManager.addCustomListener("game_on_show", this.onGameShow.bind(this));
```

#### Call `addCustomListener` from `javascript`


#### Trigger `addCustomEventListener` in `C++`

```C++
bool js_cocos2dx_EventDispatcher_addCustomEventListener(JSContext *cx, uint32_t argc, jsval *vp
```

##### Check `Function`

```C++
JS_TypeOfValue(cx, args.get(1)) == JSTYPE_FUNCTION
```

##### Create `JSFunctionWrapper` with `this` and `onGameShow`

```C++
JS::RootedObject jstarget(cx, args.thisv().toObjectOrNull());
std::shared_ptr<JSFunctionWrapper> func(new JSFunctionWrapper(cx, jstarget, args.get(1)));
```

##### Create a `lambda` for `Callback` with data

```C++
auto lambda = [=](cocos2d::EventCustom* larg0) -> void {/**/};
```

##### Create `cocos2d::EventListenerCustom`

```C++
cocos2d::EventListenerCustom* ret = cobj->addCustomEventListener(arg0, arg1);
```

##### Return `cocos2d::EventListenerCustom` to  `javascript`

```C++
jsret = OBJECT_TO_JSVAL(
js_get_or_create_jsobject<cocos2d::EventListenerCustom> (
    cx, (cocos2d::EventListenerCustom*)ret
)
args.rval().set(jsret);
```

### Event Dispatch in Cocos

``` javascript
// javascript
cc.eventManager.dispatchCustomEvent("game_on_show", "custom_data");
```

#### Call `dispatchCustomEvent` from `javascript`


#### Create `cc.EventCustom` in `javascript`

```javascript
cc.eventManager.dispatchCustomEvent = function (eventName, userData) {
    var ev = new cc.EventCustom(eventName);
    ev.setUserData(userData);
    this.dispatchEvent(ev);
};
```

#### Trigger `EventCustom_constructor` in `C++`

```C++
bool js_cocos2dx_EventCustom_constructor(/* ... */) {/* ... */}
```

#### Set user data in `javascript`

```javascript
ev.setUserData(optionalUserData);
```

**User data is not passed to `C++`, stored in `javascript`

```javascript
// jsb_cocos2d.js
cc.Node.prototype.setUserData = function (data) {
    this.userData = data;
};
```

#### Call `dispatchEvent`

```javascript
this.dispatchEvent(ev);
```

#### Trigger `EventDispatcher_dispatchEvent` in `C++`

```C++
bool js_cocos2dx_EventDispatcher_dispatchEvent(/* ... */) {
    cocos2d::Event* arg0 = nullptr;
    js_proxy_t *jsProxy;
    JS::RootedObject tmpObj(cx, args.get(0).toObjectOrNull());
    jsProxy = jsb_get_js_proxy(tmpObj);
    cobj->dispatchEvent(arg0);
}
```

#### Trigger stored `lamba` function

```C++
void {
    JSB_AUTOCOMPARTMENT_WITH_GLOBAL_OBJCET
    jsval largv[1];
    largv[0] =  OBJECT_TO_JSVAL(
    js_get_or_create_jsobject<cocos2d::EventCustom>(
        cx, (cocos2d::EventCustom*)larg0)
    );
  
    JS::RootedValue rval(cx);
    bool succeed = func->invoke(1, &largv[0], &rval);
};
```

#### `JSAPI` `invoke` called

```C++
extern JS_PUBLIC_API(bool)
JS_CallFunctionValue(/**/);
```

#### `javascript` receive event with data

```javascript
onGameShow: function (event) {
    event.getUserData();
}
```

#### Get user data stored in `javascript`

```javascript
cc.Node.prototype.getUserData = function () {
    return this.userData;
};
```


### CallFunc in Cocos

```javascript
// javascript
this.runAction(
    cc.callFunc(this.onGameShow, this, "some_data")
);
```

#### Call `cc.callFunc` in `javascript`


#### Trigger `CallFunc_ctor` in `C++`

```C++
static bool js_cocos2dx_CallFunc_ctor(/* JSNative */)
{
    JS::CallArgs args = JS::CallArgsFromVp(argc, vp);
    JS::RootedObject obj(cx, args.thisv().toObjectOrNull());
    cocos2d::CallFunc *nobj = new (std::nothrow) cocos2d::CallFunc();
    auto newproxy = jsb_new_proxy(nobj, obj);
    jsb_ref_init(cx, &newproxy->obj, nobj, "cocos2d::CallFunc");
    bool isFound = false;
    if (JS_HasProperty(cx, obj, "_ctor", &isFound) && isFound)
        executeFunctionWithOwner(OBJECT_TO_JSVAL(obj), "_ctor", args);
    args.rval().setUndefined();
    return true;
}
```

#### Trigger `_ctor` in  `javascript`

```javascript
cc.CallFunc.prototype._ctor = function(selector, selectorTarget, data) {
    // ......
    this.initWithFunction(selector, selectorTarget, data);
};
```

#### Trigger `initWithFunction` in `C++`

```C++
bool js_cocos2dx_CallFunc_initWithFunction(/* JSNative */) {
    // ......
    JS::CallArgs args = JS::CallArgsFromVp(argc, vp);
    JS::RootedObject obj(cx, args.thisv().toObjectOrNull());
    js_proxy_t *proxy = jsb_get_js_proxy(obj);
    CallFuncN *action = (cocos2d::CallFuncN *)(proxy ? proxy->ptr : NULL);
    JSB_PRECONDITION2(action, cx, false, "Invalid Native Object");
    // ......
}
```

#### Create `JSCallbackWrapper` to keep the `param` 

`target` `selector` `data` 

```C++
std::shared_ptr<JSCallbackWrapper> tmpCobj(new JSCallbackWrapper());

JS::RootedValue callback(cx, args.get(0));
tmpCobj->setJSCallbackFunc(callback);

JS::RootedValue thisObj(cx, args.get(1));
tmpCobj->setJSCallbackThis(thisObj);

JS::RootedValue data(cx, args.get(2));
tmpCobj->setJSExtraData(data);
```

#### Add a `lamba` function to `NativeObject`

```C++
action->initWithFunction([=](Node* sender) {/* */});
```

#### Trigger `lamba` from `C++`

```C++
{
    JSB_AUTOCOMPARTMENT_WITH_GLOBAL_OBJCET

    JS::RootedValue jsvalThis(cx, tmpCobj->getJSCallbackThis());
    JS::RootedObject thisObj(cx, jsvalThis.toObjectOrNull());
    JS::RootedValue jsvalCallback(cx, tmpCobj->getJSCallbackFunc());
    JS::RootedValue jsvalExtraData(cx, tmpCobj->getJSExtraData());

    JS::RootedValue senderVal(cx);
    js_type_class_t *typeClass = 
        js_get_type_from_native<cocos2d::Node>(sender);
    auto jsobj = jsb_ref_get_or_create_jsobject(
        cx, sender, typeClass, "cocos2d::Node"
    );
    senderVal.set(OBJECT_TO_JSVAL(jsobj));

    JS::RootedValue retval(cx);
    jsval valArr[2];
    valArr[0] = senderVal;
    valArr[1] = jsvalExtraData;

    JS::HandleValueArray callArgs = 
        JS::HandleValueArray::fromMarkedLocation(2, valArr);
    JS_CallFunctionValue(cx, thisObj, jsvalCallback, callArgs, &retval);
}
```

#### Trigger `callback` in `javascript`

```javascript
onGameShow: function(sender, extraData) {
    /* ... */
}
```

### Schedule in Cocos

``` javascript
// javascript
node.schedule(this.scheduleFunc, 1);
```

#### Call `schedule` in `javascript`


#### Trigger `node_schedule` in `C++`

```C++
bool js_CCNode_schedule(JSContext *cx, uint32_t argc, jsval *vp) {
    /* ... */
}
```

#####  Get the `Schedule Object` bind to the `NativeObject`

```C++
JS::RootedValue thisValue(cx, args.thisv());
JS::RootedObject obj(cx, thisValue.toObjectOrNull());
js_proxy_t *proxy = jsb_get_js_proxy(obj);
cocos2d::Node *node = (cocos2d::Node *)(proxy ? proxy->ptr : NULL);
Scheduler *sched = node->getScheduler();
```

##### Create a `JSScheduleWrapper` to keep `params`

``` C++
JSScheduleWrapper *tmpCobj = NULL;
// ...
tmpCobj = new JSScheduleWrapper();
tmpCobj->autorelease();
tmpCobj->setJSCallbackThis(thisValue);
tmpCobj->setJSCallbackFunc(args.get(0));
tmpCobj->setTarget(node);
JSScheduleWrapper::setTargetForSchedule(args.get(0), tmpCobj);
JSScheduleWrapper::setTargetForJSObject(obj, tmpCobj);
```

##### Call `schedual` function on the `NativeObject` 

```C++
sched->schedule(
    schedule_selector(JSScheduleWrapper::scheduleFunc), 
    tmpCobj, 
    interval, !node->isRunning()
);
```

#### Trigger JSScheduleWrapper::scheduleFunc

```C++
void JSScheduleWrapper::scheduleFunc(float dt) {
    // ...  
    JS::RootedValue callback(cx, getJSCallbackFunc());
    JS::HandleValueArray args = JS::HandleValueArray::fromMarkedLocation(1, &data);
    JS::RootedValue retval(cx);
    JS::RootedObject callbackTarget(cx, getJSCallbackThis().toObjectOrNull());
    JS_CallFunctionValue(cx, callbackTarget, callback, args, &retval);
}
```

#### Trigger `callback` in `javascript`

``` javascript
scheduleFunc:function (dt) {
    // ...
} 
```

