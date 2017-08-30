---
title: "Waiting in UiAutomator"
excerpt: "Advanced technics for building custom waits in UiAutomator"
comments: true
related: true
header:
    image: "/assets/images/uiautomator_basics_thumbnail.jpg"
    overlay_image: /assets/images/uiautomator_waits.jpg
    overlay_filter: 0.25
date: 2017-08-27
tags:
  - uiautomator
  - android
---
{% include toc title="Waiting in UiAutomator" icon="file-text" %}
## Why to wait?
Real-life waiting is hard, and waiting in tests is not an exception. I guess all of us had difficult times, when one test failed just because we forgot about waits. Most of the test engineers use explicit waits in Selenium and similar frameworks and well-aware of their advantages over usual `Thread.sleep()`.

Waiting is an essential technique to build fast, reliable and efficient automation in your projects. UiAutomator is no different. But it could be hard to understand how to use waits with Google's UI test tool, because of lack of documentation on the Internet. I'll try to share easy and straightforward approach to waiting for conditions with new classes, introduced in UiAutomator2.

If you're new to UiAutomator, I recommend to read my article on [UiAutomator Essentials](https://alexilyenko.github.io/uiautomator-basics/) first, where I introduced framework's main classes and API.
{: .notice--info}

## Conditions
Wait methods in UiAutomator look like `wait(Condition condition, long timeout)` and are used for, well, waiting for conditions to be fulfilled during given timeout.
### Common conditions
Most common conditions which might come in handy in every day automation are conveniently gathered in one utility class - `Until`. One can found three actual types of conditions there:
* **SearchCondition** - a condition which is satisfied by searching for some element by specified selector
```java
  // until element is available
  SearchCondition<UiObject2> isAvailable =
                 Until.findObject(By.text("foo"));
  // until element is gone
  SearchCondition<Boolean> isGone =
                 Until.gone(By.text("foo"));
```
* **UiObject2Condition** - condition which is satisfied when `UiObject2` is in particular state or has specific attribute
```java
  // until element is focused
  UiObject2Condition<Boolean> isFocused =
                 Until.focused(true);
  // until element's text contains
  UiObject2Condition<Boolean> textContains =
                 Until.textContains("foo");
```
* **EventCondition** - condition which depends on event or series of event which should occur before given timeout expire
```java
  // until new window appears in the app
  EventCondition<Boolean> newWindowAppears =
                 Until.newWindow();
  // until previous scroll is finished
  EventCondition<Boolean> isScrollFinished =   
                 Until.scrollFinished(Direction.UP);
```

### Building your own conditions
Of course you can build your own conditions if you feel like basic ones are not enough for your. All you need to do is to implement needed interface. For example, here is how I implemented condition which will be fulfilled if resource id of element starts with some prefix:
```java
// Condition will be satisfied if given element
// has specified count of children
public static UiObject2Condition<Boolean> hasChildren(final int childCount) {
        return new UiObject2Condition<Boolean>() {
            @Override
            boolean apply(UiObject2 object) {
                return object.getChildCount() == childCount;
            }
        };
    }
```

## Using waits
There are only two classes in UiAutomator which are able to utilize `Condition` - `UiDevice` and `UiObject2`.

* **UiDevice**, similarly to Selenium's `WebDriver`, represents Android device itself and with its help you can wait for elements or events
```java
  UiDevice device = UiDevice.getInstance(
       InstrumentationRegistry.getInstrumentation());
  UiObject2 element = device.wait(Until
       .findObject(By.text("foo"), DEFAULT_TIMEOUT)));
```

* **UiObject2** represents UI element in app hierarchy and will allow you to wait for its or its attributes' changes. It's similar to Selenium's `WebElement`.
```java
  UiObject2 element = driver.findElement(By.res("foo"));
  element.wait(Until.textContains("bar"),
        DEFAULT_TIMEOUT);
```

`UiObject` and `UiObject2` are two different classes. Unlike first version `UiObject2` can be used even if underlying view object is terminated. If you're using UiAutomator2, I advice you to use only the second version since all the new APIs support it.
{: .notice--info}

In the given examples UiAutomator will wait for some condition to be satisfied during `DEFAULT_TIMEOUT` of type `Long`. It will poll every **1 second** and check if condition is equal to `true`. For now polling time is a constant and can not be changed, but hopefully Google team will change that in future releases. If condition is not satisfied during the timeout, it will throw the `TimeoutException`.

Basically that's all you need to know to start using waits in UiAutomator. In the next article I'll explain how you can utilize them in your framework and build successful Android automation.

[<img src="{{ site.url }}{{ site.baseurl }}/assets/images/share_message.png" alt="Feel free to share!">](https://alexilyenko.github.io/)
