---
title: "UiAutomator Basics"
excerpt: "Everything you need to know to start using UiAutomator"
comments: true
related: true
header:
    overlay_image: /assets/images/uiautomator_basics.jpg
    overlay_filter: 0.25
date: 2017-08-19
categories:
  - android
tags:
  - uiautomator
  - testing
  - android
---
{% include toc title="UiAutomator Basics" icon="file-text" %}
## Why UiAutomator?
Time to time my colleagues from other departments ask me about framework I'm using for automating UI tests on Android and AndroidTV devices. And they really wonder when I tell them I'm using **UiAutomator** for those purposes. Not because they think it's not appropriate solution for their tasks, but rather because they haven't even heard about this kind of framework.

From my side it's odd to hear that. No, don't get me wrong, I'm familiar with all those fancy frameworks like Appium, Espresso, Robotium etc (and we'll talk about them in my further posts). But let's face the truth - most of the functionality these instruments provide you'll never use, and to support them you'll have to have whole team of automation engineers. And that's expensive, right?

On the other hand there is UiAutomator.
> UiAutomator is a light-weight, easy-to-learn library, developed by Google to make things done fast, when you don't want to spend lot's of time on developing test code and its maintainance.

Still not convinced? What if I tell you this little tool can help you to build fully functional, extendable and reliable test frameworks? Tempting, isn't it? Stay tuned!

**Disclaimer:** we're going to talk about UiAutomator2 since it's the most recent version, which includes all the fixes of the problems UiAutomator(1) had.
{: .notice--info}

## How to get started with UiAutomator?
### Android Studio
First of all, you'll need to configure your Android Studio to be able to use support libraries, which include UiAutomator. This can be done in Android Studio settings under Android SDK panel.

{% include figure image_path="/assets/images/android_support.jpg" alt="how to configure uiautomator" caption="Go to Android Studio Settings, open Android SDK panel and enable Android Support Repository" %}

### Gradle
Next thing we'll need is to set up Gradle. To do that, following changes should be added to `build.gradle` file. 

```groovy
android {
	...
    defaultConfig {
        ...
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
    }
...
dependencies {
	...
    androidTestCompile 'com.android.support.test.uiautomator:uiautomator-v18:2.1.3'
}
```
Since other settings in the file depend on your app and what it's built for, they are up to you. But these two lines must be included into `build.gradle` in order to use UiAutomator in your project.

That's basically it! It wasn't difficult, right? You're ready to start writing you first test.

## Introduction to UiAutomator library
As I said before UiAutomator is a light-weight library so you won't need to learn lots of classes to start using it. Here are the main ones:

* **[BySelector](https://developer.android.com/reference/android/support/test/uiautomator/BySelector.html)** - class for specifying criteria for matching UI elements in app UI herarchy. It's similar to Selenium's `By` class.

```java
BySelector selector = new BySelector().text("foo");
```
* **[By](https://developer.android.com/reference/android/support/test/uiautomator/By.html)** - utility class for providing static factory methods for constructing `BySelector`s using a shortened syntax. Folowing line brings the same meaning as `new BySelector().text("foo")`:

```java
BySelector selector = By.text("foo");
```

* **[UiObject2](https://developer.android.com/reference/android/support/test/uiautomator/UiObject2.html)** - one of the most important classes in UiAutomator library. It represents UI element in Android app hierarchy. It's similar to Selenium's `WebElement` or Appium's `MobileElement`. You can do lots of things with it (for example read and verify element's attributes, click on it, wait for it, look for inner elements and many more).

```java
UiObject2 element = device(By.text("foo"));
element.click();
```

* **[UiDevice](https://developer.android.com/reference/android/support/test/uiautomator/UiDevice.html)** - another important class for building successfull UI test framework. It represents the device itself - either Android Phone, TV or even Watch. With its help you can find elements, wait for them, press device's system buttons and even control remote in case of Android TV devices. Its Selenium equivalent would be `WebDriver`, which I guess is familiar to most of you.

```java
UiDevice device = UiDevice.getInstance(InstrumentationRegistry.getInstrumentation());
device.pressBack();
```

* **[Until](https://developer.android.com/reference/android/support/test/uiautomator/Until.html)** - utility class for constructing common conditions used for searching and waiting for elements (i.e. [SearchCondition](https://developer.android.com/reference/android/support/test/uiautomator/SearchCondition.html) and [UiObject2Condition](https://developer.android.com/reference/android/support/test/uiautomator/UiObject2Condition.html), which were prototyped from Selenium's `ExpectedConditions`). I'm going to write a post about conditions and waits in UiAutomator next week. Stay tuned!

```java
long timeout = 10000L;
device.wait(Until.gone(By.text("foo")), timeout)
```
* **[Configurator](https://developer.android.com/reference/android/support/test/uiautomator/Configurator.html)** - from the name of the class you could easily get that its main purpose is to configure your UiAutomator test runner. For example I use it to remove implicit timeouts of UiAutomator, because I like to decide whenever to wait for events or not by myself. But of course it's up to you.

```java
Configurator configurator = Configurator.getInstance();
long timeout = 0L;
configurator.setWaitForSelectorTimeout(timeout);
```

These are the most common classes in UiAutomator library. But there are others, which might be useful in building your automated test framework. You can check them out in the [official documentation](https://developer.android.com/reference/android/support/test/uiautomator/package-summary.html).

## UiAutomator test example
Since you already know about main classes in the library let's find out how to use them in your tests.
Here is the code snippet of the simple test script.

```java
@RunWith(AndroidJUnit4.class)
public class SimpleTest {

    private static final long DEFAULT_TIMEOUT = 30000L;
    private static final String PACKAGE_NAME = "com.example.app";
    private UiDevice device;

    @Before
    public void setUp() {
        // Disabling waiting for selector implicit timeout
        Configurator.getInstance().setWaitForSelectorTimeout(0);
        // Initializing UiDevice instance
        device = UiDevice.getInstance(InstrumentationRegistry
                                .getInstrumentation());

        // Starting the app
        Context context = InstrumentationRegistry.getInstrumentation()
                                .getContext();
        Intent intent = context.getPackageManager()
                                .getLaunchIntentForPackage(PACKAGE_NAME);
        context.startActivity(intent);

        // Waiting for app activity to appear
        device.wait(Until
                .hasObject(By.pkg(PACKAGE_NAME).depth(0)), DEFAULT_TIMEOUT);
    }

    @Test
    public void test() {
        // Finding element
        UiObject2 editText = device.findObject(By.text("edit_text"));
        // Sending text to element
        editText.setText("123456");

        // Waiting for element
        BySelector submitButtonSelector = By.text("submit");
        UiObject2 submitButton = device.wait(Until
                .findObject(submitButtonSelector), DEFAULT_TIMEOUT);
        // Clicking on element
        submitButton.click();

        // Waiting for element to disappear
        device.wait(Until.gone(submitButtonSelector), DEFAULT_TIMEOUT);
    }

    @After
    public void tearDown() {
        // Taking screenshot after test
        device.takeScreenshot(new File("screenshot-file-name.jpg"));
    }
}
```

Alright folks, now you have everything to start writing your own tests using UiAutomator! Let me know if this post was useful for you and check out my other ones.

Cheers!
