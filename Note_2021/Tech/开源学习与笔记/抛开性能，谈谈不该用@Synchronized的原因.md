

clang rewrites @synchronized(expr) into 
objc_sync_enter(expr)
@try stmt @finally { objc_sync_exit(expr); }


