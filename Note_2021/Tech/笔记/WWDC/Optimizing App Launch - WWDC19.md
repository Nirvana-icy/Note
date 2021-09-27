
# Optimizing App Launch - WWDC19

## What is Launch and Why Launch is Important

![Image](https://mmbiz.qpic.cn/mmbiz_jpg/M54fjP2zXtFQagicFtL9IKicpClazVd3VGQVCVFkS4AJuxnTztwE6ZePqNdIOvouhTybYuZkLkFyX11OxqEQgpbw/0?wx_fmt=jpeg)

1. First experience with your app should be delightful.

2. Indicative of your code's overal performance.

3. Many times of launch during a day.
For 1ms everyone saved, 162 days saves one day in world.

4. A lot of cpu work & memory work during launch.
Impacts the system performance and battery.

## Launch Types

![Image](https://mmbiz.qpic.cn/mmbiz_jpg/M54fjP2zXtFQagicFtL9IKicpClazVd3VGkrKj7niaZHENpfyqnxZ57B5nztwRHxFrOiaCicmEXooP8PaCOcrdqD7vA/0?wx_fmt=jpeg)

![Image](https://mmbiz.qpic.cn/mmbiz_jpg/M54fjP2zXtFQagicFtL9IKicpClazVd3VGMiaiapbA0hicibsJE1HrArtt47NoXN3DJRsVcSkKsGVHHsN2tsgyrg9znw/0?wx_fmt=jpeg)

## The Goal To Render First Frame

## Phases of App Launch

![Image](https://mmbiz.qpic.cn/mmbiz_jpg/M54fjP2zXtFQagicFtL9IKicpClazVd3VGo5RU0zfXav2JfUJfDFN5Csa8WXHIibKFfU5Lqrt5S1OU0H7M5EwCficQ/0?wx_fmt=jpeg)

### System Interface

#### DYLD3

Dynamic Linker loads shared libraries and frameworks(from iOS 13).

***Recommended Actions:***

1. Avoid linking unused frameworks.

2. Avoid dynamic library loading during launch. eg. DLOpen or NSBundleLoad.

3. Hard link all your dependencies.

#### libSystem Init

Initializes the interfaces with low level system components.

System side work with a fixed cost.

### Static Runtime Initialization

Initializes the language runtime.

Invokes all class static load methods.

***Recommended Actions:***

1. Expose dedicated initialization API in frameworks.

2. Reduce impact to launch by avoiding +[Class load]

3. Use +[Class initialize] to lazily conduct static initialization.

### UIKit Initialization

Instantiates the UIApplication and UIApplicationDelegate.

Begins event processing and integration with the system.

***Recommended Actions:***

1. Minimize work in UIApplication subclass.

2. Minimize work in UIApplicationDelegate initialization.

### Application Initialization.

Lifecycle Callbacks

#### Invokes UIApplicationDelegate app lifecycle callbacks

1. application:willFinishLaunchingWithOptions:

2. application:didFinishLaunchingWithOptions:

#### Invokes UIApplicationDelegate UI lifecycle callbacks (Old API)

1. applicationDidBecomeActive:

### Invokes UISceneDelegate UI lifecycle callbacks for each scene (New API)

1. scene:willConnectToSession:options:

2. sceneWillEnterForeground:

3. sceneDidBecomeActive:

***Recommended Actions:***

1. Defer unrelated work.

2. Share resources between scenes for New API.


### First Frame Render

Creates, performs layout for, and draws views

Commits and renders first frame

1. loadView

2. viewDidLoad

3. layoutSubviews

***Recommended Actions:***

1. Flatten view hierarchies and lazily load views

2. Optimize auto layout usage

### Extended

App-specific period after first frame

Displays asynchronously loaded data

App should be interactive and responsive

---

# How to properly measure your launch

![image](https://mmbiz.qpic.cn/mmbiz_jpg/M54fjP2zXtFQagicFtL9IKicpClazVd3VGheKezVwLHIlVRjDAJGIsuVibibH5Cdfc0xF2RpJHmfyF6CMv4VQ8JvicA/0?wx_fmt=jpeg)

![image](https://mmbiz.qpic.cn/mmbiz_jpg/M54fjP2zXtFQagicFtL9IKicpClazVd3VGOxnF4u911mOp0deUwxnKQaDtxGoxwUwMzbrsGRpZyR1axe0u3JAuiaA/0?wx_fmt=jpeg)

![image](https://mmbiz.qpic.cn/mmbiz_jpg/M54fjP2zXtFQagicFtL9IKicpClazVd3VGroQ7CFsVyT8iaiayciaR30uvNW22Wqe1d3veG78J85A1bZtNn3ysUNcibg/0?wx_fmt=jpeg)

![image](https://mmbiz.qpic.cn/mmbiz_jpg/M54fjP2zXtFQagicFtL9IKicpClazVd3VGiawjocU33G8XrtJg52Km23K9vX9OibCdzUbVibMuvL83Befbm36FmkD6Q/0?wx_fmt=jpeg)

---

# Tips and Tricks

![image](https://mmbiz.qpic.cn/mmbiz_jpg/M54fjP2zXtFQagicFtL9IKicpClazVd3VGo56bIG93F0qapSsst4nVa5Exf6ShlbI8hUqzkbN6jhQddzT9LecAyQ/0?wx_fmt=jpeg)

![image](https://mmbiz.qpic.cn/mmbiz_jpg/M54fjP2zXtFQagicFtL9IKicpClazVd3VGXiabw99yeVw4tOwtDGyq5ex4gFtbyzErEB0bYibNicLOCfm4HO2HE1Lyg/0?wx_fmt=jpeg)

![image](https://mmbiz.qpic.cn/mmbiz_jpg/M54fjP2zXtFQagicFtL9IKicpClazVd3VGGyfTXtuFKumJWdNiahoFJkbHtWye1qpuVHzxdOKwH7ibYWcsQhcwwzwA/0?wx_fmt=jpeg)

![image](https://mmbiz.qpic.cn/mmbiz_jpg/M54fjP2zXtFQagicFtL9IKicpClazVd3VGHwiaG3wr6Gz3g7kRK7Fbcgm82ZNQf07gkYp2Lt40XHwhwZDtib50XMKw/0?wx_fmt=jpeg)

---

# More to Learn

*Modernizing Grand Central Dispatch Usage - WWDC17*

*Multitasking and the Application Lifecycle - WWDC19*

*Getting the Most out of Multitasking - WWDC19*

*Mesuring Performance Using Logging - WWDC18*