---
title: "Waits in XCUITest"
excerpt: "Building reliable automation for iOS/tvOS platforms with waits and expectations"
comments: true
related: true
header:
    image: /assets/images/xctest_waits_thumbnail.png
    overlay_image: /assets/images/xctest_waits.jpg
    overlay_filter: 0.25
date: 2017-10-06
tags:
  - ios
---
{% include toc title="Explicit waits in XCUITest" icon="file-text" %}
## Waiting is Essential
The time of the static content on the Web has long gone. It was replaced by dynamic websites and asynchronous mobile applications. Implicit waits cannot help us anymore in avoiding flaky tests while automating those assets. I wrote the whole article about [Waiting in Android Functional Tests](https://alexilyenko.github.io/uiautomator-waiting/) and I have to say iOS testing is not different. If you want to build fast and reliable automation for your project, you should implement explicit waits in one way or another.

Today we'll talk about expectations and waiters in XCTest framework, which might be applied in both unit and functional (user interface) tests of your iOS/tvOS/watchOS app.

If you've never worked with XCUITest before, I suggest to read my post about [XCTest UI Testing Basics](https://alexilyenko.github.io/xcuitest-basics/) first. I introduced main XCTest classes along with main rules of using them in it.  
{: .notice--info}

## Expectations
First of all, we have to figure out what we're going to wait for. Basically, this is nothing new, we will wait for some conditions to be satisfied or, if we're talking in XCUITest language, "some **expectations** to be fulfilled". While releasing Xcode 7, Apple introduced [`XCTestExpectation`](https://developer.apple.com/documentation/xctest/xctestexpectation) class, which represents expected outcome in an asynchronous test. There are several types of default expectations in XCTest library:
* **XCTKVOExpectation** - expectation which is fulfilled when Key Value Observing condition is met.
```swift
let element = XCUIApplication().
let expectation = XCTKVOExpectation(keyPath: "exists",
                        object: xCUIElement,
                        expectedValue: true)
```
* **XCTNSNotificationExpectation** - expectation which is satisfied when notification is received
```swift
let expectation = XCTNSNotificationExpectation(name:
                        Notification.Name("MyNotification"))
```
* **XCTDarwinNotificationExpectation** - the same as the previous one, with the difference of notification type. In this case we will wait for Darwin notification to be received
```swift
let expectation = XCTDarwinNotificationExpectation(
                notificationName: "DarwinNotificationName")
```
* **XCTNSPredicateExpectation** - in my opinion the main expectation in functional testing. Basically I think we can not build robust UI automation without this type of expectations. It is fulfilled when given `NSPredicate` is satisfied.
```swift
let expectation = XCTNSPredicateExpectation(predicate:
                      NSPredicate(format: "exists == true"),
                      object: xCUIElement)
```

Let's talk a bit about `NSPredicate`s.
### NSPredicates
I found `NSPredicate` the most convenient way for specifying criteria of waiting for `XCUIElement` or its attributes in XCUITest framework. With its help we are able to build advanced locators and combine them with each others. For example:
```swift
let orNSPredicate = NSPredicate(format:
   "label CONTAINS 'something' OR name MATCHES 'someRegex'")
let andNSPredicate = NSPredicate(format:
   "name BEGINSWITH 'prefix' AND NOT name ENDSWITH 'suffix'")
let count = NSPredicate(format: "self.count = 2")
```  
In addition to this, `NSPredicate` may be also used in searching for elements, which might be really handy:
```swift
let predicate = NSPredicate(format: "value CONTAINS 'word'")
let field = app.textFields.matching(predicate)
```

## How to Wait in XCTest
There are couple of waiting options built into XCUITest framework.
### XCTestCase Wait
As you may already know, to implement any test in XCTest we have to extend basic `XCTestCase` class with our test class. The thing you might not know is this class has already defined method `wait`. This means any test in our framework can use it to wait for expectations. It accepts array of expectation objects and synchronously waits for each of them in the given order for given amount of time.
```swift
wait(for: [expectation1, expectation2],
     timeout: TimeInterval(timeoutValue))
```

I know what you may want to ask. What if we want extract waits to separate util class, which is considered a good practice? If we do that we won't be able to use `wait` method of `XCTestCase` class assuming our util class won't extend it. That's right, but of course there is a solution out there. Its name is `XCTWaiter`.
### XCTWaiter
This class was introduced by Apple along with Xcode 8 release, which means if you're using somewhat the latest version, you probably have access to it.
```swift
_ = XCTWaiter.wait(for: [expectation1, expectation2],
            timeout: TimeInterval(timeoutValue))
```
From the first glance one can say that this method is doing exactly the same as test case's `wait`. But there are hidden advantages in using `XCTWaiter`. If you take a closer look at the snippets above, you may notice that the second example returns something. It reveals the object of `XCTWaiter.Result` class which represents the result of expectations supposed to be fulfilled. There are four types of result which can be returned by waiter:
- `.completed` means condition was satisfied
- `.timedOut` - result which says expectation was not fulfilled during given timeout
- `.incorrectOrder` represents situation when expectations were fulfilled in the order not equal to the given one
- `.invertedFulfillment` is a result that means inverted expectation was fulfilled
- `.interrupted` is saying waiting was interrupted prior to its expectations being fulfilled or timed out

The next thing deserving attention is `XCTWaiter` won't fail your test even if expectation was not fulfilled. Now it's completely our responsibility as developers of the tests. If you think about this, it's a great improvement. We are now completely in control of when and how to fail our tests. Now it's possible to wait even for optional conditions without any risks of fails.
```swift
func waitForExpectation(expectation:XCTestExpectation,
                        time: Double,
                        safe: Bool = false) {
 let result: XCTWaiter.Result =
                    XCTWaiter().wait(for: [expectation],
                                     timeout: time)
 if !safe && result != .completed {
  // if expectation is strict and was not fulfilled
  XCTFail("Condition was not satisfied during \(time) seconds")
 }
}
```
### Custom expectations
The last thing, I wanted to cover today, is creating your custom expectations and waiting for their fulfillment. There might be situations out there when you want to wait for something, but none of the default expectations is suitable for that. There is a technique for this too!

For example, we need to wait for our asynchronous method `executeSomething` to be completed. All we need to do is to invoke `fulfill` method of `XCTestExpectation` object in the `completion` handler block like this:
```swift
let expectation = XCTestExpectation(description:
                                "doSomething() is finished")
executeSomething(completion: { _ in
            doSomething()
            expectation.fulfill()
        }, failed: { _ in
            print("Something went wrong!")
        })
_ = XCTWaiter().wait(for: [expectation],
                timeout: TimeInterval(timeoutValue))
```
These are the main concepts of waiting in XCUITest. I will explain how they might be used in building successful iOS automation in the next series of posts so stay tuned!
[<img src="{{ site.url }}{{ site.baseurl }}/assets/images/share_message.png" alt="Feel free to share!">](https://alexilyenko.github.io/)
