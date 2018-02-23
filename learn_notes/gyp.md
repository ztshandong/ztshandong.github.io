# MAC
```sh
node v9.5.0
npm 5.6.0
python 必须 2.7
python 2.7.10
npm install -g node-gyp
node-gyp v3.6.2
node-gyp clean configure build
```

# DEMO
```sh
mkdir gyp

binding.cc
#include <node.h>
#include <v8.h>

void Method(const v8::FunctionCallbackInfo<v8::Value>& args) {
  v8::Isolate* isolate = args.GetIsolate();
  args.GetReturnValue().Set(v8::String::NewFromUtf8(isolate, "world"));
}

void init(v8::Local<v8::Object> exports) {
  NODE_SET_METHOD(exports, "hello", Method);
}

NODE_MODULE(NODE_GYP_MODULE_NAME, init)


binding.gyp
{
  'targets': [
    {
      'target_name': 'binding',
      'defines': [ 'V8_DEPRECATION_WARNINGS=1' ],
      'sources': [ 'binding.cc' ]
    }
  ]
}


hellogyp.js
var hello = require('./build/Release/binding.node').hello();
console.log(hello);

node hellogyp.js 
```

# 普通传参
```sh
lz_primitive.cc

#include "node.h"
#include "v8.h"

using v8::Isolate;
using v8::FunctionCallbackInfo;
using v8::Value;
using v8::Local;
using v8::String;
using v8::Object;
using v8::Number;
using v8::Boolean;

void method(const FunctionCallbackInfo<Value> &args){
    //
    Isolate * isolate = args.GetIsolate();

    if (args[0]->IsNumber()){
        double a = args[0]->NumberValue();
        Local<Number> return_val =  Number::New(isolate,a+1);
        args.GetReturnValue().Set(return_val);
    };

    if (args[0]->IsBoolean()){
        bool a = args[0]->BooleanValue();
        Local<Boolean> return_val = Boolean::New(isolate,!a);
        args.GetReturnValue().Set(return_val);
    };

    if (args[0]->IsString()){
        Local<String> add_str = String::NewFromUtf8(isolate,"lizhiqianduan");
        add_str = String::Concat(args[0]->ToString(),add_str);
        args.GetReturnValue().Set(add_str);
    };

}

void init(Local<Object> exports) {
    NODE_SET_METHOD(exports, "method", method);
}

NODE_MODULE(addon, init)



binding.gyp 
{
  
    "targets":[
        {
            # 指定C++模块的名字，这里为lz_primitive.node
            'target_name': 'lz_primitive',
            'sources': [
                "lz_primitive.cc"
            ]
        }
    ]
}



test.js
var readline = require('readline');
  
//创建readline接口实例
var  rl = readline.createInterface({
    input:process.stdin,
    output:process.stdout
});

var lz_primitive = require("./build/Release/lz_primitive.node");
var reg = new RegExp("^[0-9]*$");
rl.on('line', function(line){
    switch(line.trim()) {
        case 'close':
            rl.close();
            break;
        default:
        var str=line.trim();
if(reg.test(str))
{
    str = parseInt(str);
}

console.log(Object.prototype.toString.call(str));

    var a = lz_primitive.method(str);

console.log(" return --> ",a);
            break;
    }
});
rl.on('close', function() {
    console.log('bye bye');
    process.exit(0);
});

node-gyp clean configure build
```

# 传递函数
```sh
lz_function.cc
#include "node.h"
#include "v8.h"
 
using namespace v8;
 
 
void method(const FunctionCallbackInfo<Value> &args){
    Isolate * isolate = args.GetIsolate();
 
    if(args[0]->IsFunction()){
        Local<Function> js_fn = Local<Function>::Cast(args[0]);
        for(int i=0;i<11;i++){
            printf("%d\n", i);
        }
        js_fn->Call(Null(isolate),0,NULL);
        args.GetReturnValue().Set(js_fn);
    }
}
 
void init(Local<Object> exports,Local<Object> module) {
    NODE_SET_METHOD(exports, "method", method);
}
 
NODE_MODULE(addon, init)


binding.gyp
{
     
    "targets":[
        {
            'target_name': 'lz_primitive',
            'sources': [
                "lz_primitive.cc"
            ]
        }
        ,{
            'target_name': 'lz_function',
            'sources': [
                "lz_function.cc"
            ]
        }
    ]
}


test.js
var lz_function = require("./build/Release/lz_function.node");
function fn_param(){
    console.log("call fn fn_param by sync!");
}
var a = lz_function.method(fn_param);
console.log(a);
console.log("test sync");

node-gyp clean configure build
```

# 传递对象
```sh
lz_object.cc
#include "node.h"
#include "node_object_wrap.h"
#include "v8.h"
 
using namespace v8;
 
// 给我们的对象附加一个函数
void cb(const FunctionCallbackInfo<Value> &args){
    Isolate* isolate = args.GetIsolate();
    args.GetReturnValue().Set(String::NewFromUtf8(isolate, "Hello World!"));
    printf("this is called in addon module!\n");
}
 
// 供外部调用的方法
void method(const FunctionCallbackInfo<Value> &args){
     
    Isolate * isolate = args.GetIsolate();
    // 需要返回的v8对象
    Local<Object>  v8Obj = v8::Object::New(isolate);
    // 取得函数的指针
    //FunctionCallback out_cb = cb;
 
    // 给v8对象设置一个键名为key值为111的字段
    v8Obj->Set(v8::String::NewFromUtf8(isolate,"key"),Integer::New(isolate,111));
    // 给v8对象设置一个键名为run_cplus_fn的函数
    //v8Obj->Set(v8::String::NewFromUtf8(isolate,"run_cplus_fn"),out_cb);
    
    // Function 类型的声明并赋值
    Local<FunctionTemplate> tpl = v8::FunctionTemplate::New(isolate, cb);
    Local<Function> fn = tpl->GetFunction();
    // 函数名字
    //fn->SetName(String::NewFromUtf8(isolate, "run_cplus_fn"));
    v8Obj->Set(v8::String::NewFromUtf8(isolate, "run_cplus_fn"), fn);

    // 如果传递的参数是v8::Object类型，再给对象设置一个键名为js_obj_keys的数组，数组值为js对象的key集合
    if(args[0]->IsObject()){
        Local<Object>  received_v8_obj = args[0]->ToObject();
        Local<Array> keys = received_v8_obj->GetOwnPropertyNames();
        v8Obj->Set(String::NewFromUtf8(isolate,"js_obj_keys"),keys);
    }
    args.GetReturnValue().Set(v8Obj);
}
 
void init(Local<Object> exports,Local<Object> module) {
    NODE_SET_METHOD(exports, "method", method);
}
 
NODE_MODULE(addon, init)


binding.gyp
{
     
    "targets":[
       {
            'target_name': 'lz_object',
            'sources': [
                "lz_object.cc"
            ]
        }
    ]
}


test.js
var lz_object = require("./build/Release/lz_object.node");
 
var no_obj_pass = lz_object.method();
console.log(no_obj_pass);
var pass_obj = lz_object.method({key:1,key_a:2,c:"sdf",test:[1,2,3]});
console.log(pass_obj);
no_obj_pass.run_cplus_fn();
pass_obj.run_cplus_fn();

node-gyp clean configure build
```

