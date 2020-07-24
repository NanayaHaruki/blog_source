---
title: databinding 双向绑定在EditText上的一些问题
date: 2018-02-01 19:39:27
tags: 
- Android
- DataBinding
categories: Android
---

 有很多种方式，比如手动过滤类或名字，如下
```
Gson gson = new GsonBuilder().setExclusionStrategies(new ExclusionStrategy() {   
            @Override  
            public boolean shouldSkipField(FieldAttributes f) {  
                 //过滤掉字段名包含"age"  
                return f.getName().contains("age");  
            }  
              
            @Override  
            public boolean shouldSkipClass(Class<?> clazz) {  
                //过滤掉 类名包含 Bean的类  
                return clazz.getName().contains("Bean");  
            }  
        }).create();  
```
这里主要通过`@Expose`来处理。
先写个data class
```
class User {
    @Expose(serialize = true, deserialize = false)
    var age: Int? = null
    @Expose(serialize = false, deserialize = true)
    var name: String? = null

    @Expose(serialize = false, deserialize = false)
    var isMale: Boolean? = null
    @Expose(serialize = true, deserialize = true)
    var isFemale: Boolean? = null

    @Expose
    var address: String? = null
    var phone: String? = null
}
```
列出了6种情况。serialize 为序列化，deserialize 为反序列化，让我们看看区别吧。
```
        val json = """{
    "age": 13,
    "name": "Lily",
    "isMale": true,
    "isFemale": false,
    "address": "江苏省",
    "phone": "13999999999"
}"""
        LogUtils.d(json)
        val user = Gson().fromJson(json,User::class.java)
        val userBuilder = GsonBuilder().excludeFieldsWithoutExposeAnnotation().create().fromJson(json,User::class.java)
        printUser(user)
        printUser(userBuilder)
    }

    fun printUser(user:User) {
        LogUtils.d("""
            age ${user.age}           true false
            name ${user.name}          false true
            isFemale ${user.isFemale}     false false
            isMale ${user.isMale}       true true
            address ${user.address}      default
            phone ${user.phone}         none
        """.trimIndent())
    }
```
结果是这样的
```
┌─────────────────────────────────────────────────
│ main, onCreate(MainActivity.kt:44)
├┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄
│ {
│     "age": 13,
│     "name": "Lily",
│     "isMale": true,
│     "isFemale": false,
│     "address": "江苏省",
│     "phone": "13999999999"
│ }
└─────────────────────────────────────────────────
┌─────────────────────────────────────────────────
│ main, printUser(MainActivity.kt:52)
├┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄
│ age 13           true false
│ name Lily          false true
│ isMale true       false false
│ isFemale false     true true
│ address 江苏省      default
│ phone 13999999999         none
└─────────────────────────────────────────────────
┌─────────────────────────────────────────────────
│ main, printUser(MainActivity.kt:52)
├┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄
│ age null           true false
│ name Lily          false true
│ isMale null       false false
│ isFemale false     true true
│ address 江苏省      default
│ phone null         none
└─────────────────────────────────────────────────
```

直接用`Gson().fromJson()`不受影响。
在使用`GsonBuilder().excludeFieldsWithoutExposeAnnotation().create().fromJson()`的时候，看到注解上标注了`deserialize = false`的`age` `isMale`和没加上注解的`phone`没有被赋值。

---
现在来试试对象转成json
```
val user=User().apply {
    age = 14
    name= "Lily"
    isMale = false
    isFemale = true
    address = "江苏省"
    phone = "13999999999"
}
LogUtils.json(Gson().toJson(user))
LogUtils.json(GsonBuilder().excludeFieldsWithoutExposeAnnotation().create().toJson(user))
```
结果是这样的
```
┌───────────────────────────────────────────────────────
│ main, onCreate(MainActivity.kt:58)
├┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄
│ {
│     "address": "江苏省",
│     "age": 14,
│     "isFemale": true,
│     "isMale": false,
│     "name": "Lily",
│     "phone": "13999999999"
│ }
└───────────────────────────────────────────────────────
┌───────────────────────────────────────────────────────
│ main, onCreate(MainActivity.kt:59)
├┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄
│ {
│     "address": "江苏省",
│     "age": 14,
│     "isFemale": true
│ }
└───────────────────────────────────────────────────────
```
用`Gson().toJson()`依旧不受影响。
而使用了`excludeFieldsWithoutExposeAnnotation`可以看到，标注了`serialize = false`的`name` `isMale`和没有加注解的`phone`没有被赋值。
---
结论：当使用`GsonBuilder().excludeFieldsWithoutExposeAnnotation().create()`的时候，对象转json的序列化过程受到`@Expose(serialize = false)`影响。
json转对象的反序列化过程受到`@Expose(deserialize = false)`影响

@Expose()  默认为Expose(deserialize = true,serialize = true)

---
EOF

