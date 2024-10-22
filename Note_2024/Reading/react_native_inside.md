# RN Setup Overview.

# RCTBridge setUp & inside RCTBridge

# Layout logic and View Flattening algorithm in C++


----

# New Arch
## React-RuntimeApple
1. RCTHost
2. RCTInstance
3. RCTFabricSurface
4. RCTSurfaceView
5. RCTSurfaceStage
6. RCTSurfaceTouchHandler

### RCTSurfaceRresenter
1. RCTMountingManager
2. RCTSurfaceRegistry
3. RCTScheduler
4. RuntimeExecutor

## RCTBridge
### RCTContextExecutor: RCTJavaScriptExecutor
1. RCTJavaScriptEventDispatcher
2. RCTTiming
   
### RCTUIManager
1. RCTJavaScriptEventDispatcher *eventDispatcher
2. RCTSparseArray *shadowViewRegistry
3. RCTSparseArray *viewRegistry
4. RCTAnimationRegistry *animationRegistry



