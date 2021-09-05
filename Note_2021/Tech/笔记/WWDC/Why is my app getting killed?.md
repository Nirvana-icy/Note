### Top causes of app terminations

#### Crashes

1. Segmentation fault

2. Illegal instruction

3. Asserts and uncaugth exceptions

#### Watchdog

Timeout during key transitions:

1. didFinishLaunchingWithOptions

2. didEnterBackground

3. willEnterForeground

Disabled in Simulator and in the debugger.

Eliminate deadlocks, infinite loops, synchronous work.

#### CPU resource limit

1. High sustained CPU load in background.

2. Energy Exception Report

    Xcode Organizer

    MXCPUExceptionDiagnostic

3. Call stack points out hotspots in code

4. Consider moving work into BGProcessingTask

#### Memory footprint exceeded

1. App using too much memory

2. Same limit for foreground and background

3. Use Instruments and Memory Debugger

4. Keep in mind older devices

Device before iPhone 6s,keep memory usage under 200MB.

#### Memory pressure exit(jetsam)

1. Not a bug with you app

2. Most common termination

3. System freeing up memory for active applications

This is the most common background termination.

***Reducing jetsame rate***

Aim for less than 50MB in the background

Upon backgrounding

1. Flush state to disk

2. Clear out image views

3. Drop caches

***Recovering from jetsam***

1. Save state upon entering background


2. Adopt UIKit State Restoration

#### Background task timeout

You can get some extra runtime to finish up critical work, by calling UIApplication.beginBackgroundTask.

When you finish that work, you call endBackgroundTask.

<!-- #### Actions: -->

#### Reduce Terminations

1. Identify and fix terminations.

2. Reduce memmory usage.

3. Implement state restoration.

<!-- MXBackgroundExitData provides the counts of each termination type. -->


