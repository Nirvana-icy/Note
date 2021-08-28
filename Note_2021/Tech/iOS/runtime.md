### 

_objc_init > _dyld_objc_notify_register > load_image

```c
// Discover load methods
    {
        mutex_locker_t lock2(runtimeLock);
        prepare_load_methods((const headerType *)mh);
    }

    // Call +load methods (without runtimeLock - re-entrant)
    call_load_methods();
```

```c
    environ_init();
    tls_init();
    static_init();
    runtime_init();
    exception_init();
    cache_t::init();
    _imp_implementationWithBlock_init();

    _dyld_objc_notify_register(&map_images, load_images, unmap_image);
```

### NSObject

objc_storeWeak
objc_storeStrong

### Objc-runtime-new.mm

realizeClassWithoutSwift

// Attach categories
// 把category里面的东西也合并进来进来
methodizeClass(cls);

类的load方法中，能调用分类的方法。

objc_setAssociatedObject
