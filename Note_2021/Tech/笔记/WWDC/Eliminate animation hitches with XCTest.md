[Eliminate animation hitches with XCTest](https://developer.apple.com/videos/play/wwdc2020/10077)

### Hitch metrics

#### Hitch time
Time in ms that a frame is late to display.

FPS is an absolute target that is easily skewed.

1. If your test contains any resting time during the execution of an animation,
then fps is useless since we did not expect any frames to be swapped during the resting period.

2. We often intentionally do not target the max fps.

Maybe a game only wants to run at 30fps,or a video at 24fps.

For power and performance reason, the hands on the Clock app icon only run at 10 fps.

#### Hitch ratio

Hitch time in ms per second for a given duration.

[Image]

### Development workflow

Catching hitches using Performance XCTests.

### Production  workflow

What's New in MetricKit.    WWDC20.
Diagnose Performance Issues with the Xcode Organizer.   WWDC20.

### Performance XCTest

#### XCTClockMetric
#### XCTCPUMetric
#### XCTMemoryMetric
#### XCTOSSignpostMetric
#### XCTStorageMetric
#### XCTApplicationLaunchMetric

### Performance XCTest Demo

#### Avoid Perf Impact

1. Create a separate test scheme for your Performance XCTests. 

2. Use the release build configuration, uncheck debug executable.

3. Disable automatic screenshot collection, turn off code coverage.

4. Turn off all diagnostic options.

#### Performance Test Report 

1. Set Baseline