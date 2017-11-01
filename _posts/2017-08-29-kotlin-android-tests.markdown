---
title: "Using Kotlin for Android Tests"
excerpt: "Setting up your environment for Instrumentation test development"
comments: true
related: true
header:
    image: /assets/images/android_kotlin_thumbnail1.png
    overlay_image: /assets/images/android_kotlin.jpg
    overlay_filter: 0.25
date: 2017-08-29
tags:
  - android
  - androidtv
  - kotlin
---
{% include toc title="Using Kotlin for Android UI Tests" icon="file-text" %}
## Why Kotlin?
If you follow the latest trends in Android world or development in general, you've likely already heard about [Kotlin](https://kotlinlang.org/). It's a programming language, developed by JetBrains company, creator of Android Studio. Among its numerous advantages one can find proper functions, less of boilerplate code, no-overhead null-safety, smart casts and stream-like arrays. If you're wondering what other pros and cons Kotlin has in comparison to Java, check out [official documentation](https://kotlinlang.org/docs/reference/comparison-to-java.html).

Besides that during the last Google I/O conference Android team [officially announced](https://developer.android.com/kotlin/index.html) support for Kotlin in Android projects. Obviously from now on language development will become more robust and largely scaled, allowing more and more Kotlin initiatives to appear in the nearest future. Meanwhile we, as Test Developers, can adopt this modern and powerful language in our day-to-day projects. Sounds sweet, right? Let's start then!

**Note**: if you haven't chosen the tool for your Android UI tests yet, I recommend to check out my post on [UiAutomator and its essentials](https://alexilyenko.github.io/uiautomator-basics/).
{: .notice--info}

## Android Studio Setup
Android Studio 3.0 ships with Kotlin bundled out of the box, which means you will no longer need to worry about compatibility issues and installation of any extras. But since most of us is still using the second version of the popular IDE, we have go through some additional steps. No worries, it wont' take long.

First of all, we'll need to install Kotlin plugin. This can be done by navigating to **Preferences** > **Plugins** > **Browse Repositories** and searching for `Kotlin` keyword there. After that just click on the `Install` button like it's shown in the figure

{% include figure image_path="/assets/images/kotlin_plugin.png" alt="How to install Kotlin plugin in Android Studio" caption="Select `Kotlin` and click on the `Install` button" %}

After this has been done you'll have to restart your Android Studio in order for plugin to work.

**Note**: in case you're using IntelliJ Idea, you won't need to worry about any installations, since IDE ships with Kotlin support enabled by default.
{: .notice--info}

## Project Setup
Next thing to do would be applying Kotlin plugin to both of your `build.gralde` files.
### Adding Kotlin Gradle Plugin
To set up Gradle plugin we'll need to modify `buildScript`, adding Kotlin plugin dependency, repository and version to it. After those manipulations our top-level `build.gradle` should look like this:
```gradle
buildscript {
 ext.kotlin_version = '1.1.4-2' // find the latest version version at
                                   // https://github.com/JetBrains/kotlin/releases/latest
 repositories {
  jcenter()
 }
 dependencies {
  classpath 'com.android.tools.build:gradle:2.3.3'
  classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
  // other dependencies go here
 }
}
// rest of the configuration
```
### Adding Kotlin dependency
To be able to use Kotlin in our Instrumentation tests we'll have to apply Kotlin plugin and add its dependency as `androidTestCompile` value to our app `build.gradle` file. Here is what should be set there:
```gradle
apply plugin: 'com.android.application'
apply plugin: 'kotlin-android'

// the rest of configuration

dependencies {
 // other dependencies go here
 androidTestCompile  "org.jetbrains.kotlin:kotlin-stdlib:$kotlin_version"
}
```

### Changing source folder (optional)
Default Android project structure is well-known to all mobile developers. Let's take a look at it.
{% include figure image_path="/assets/images/default_android_structure.png" alt="Default Android project structure" caption="Default Android project structure" %}

But since we're going to use **Kotlin** instead of **Java** in our Instrumentation tests, won't it have more sense to have source folder name to reflect that? This can be done by adding following line to your app `build.grale`:

```gradle
android {
 // project configuration
 sourceSets {
  androidTest.java.srcDirs += 'src/androidTest/kotlin'
 }
}
```
Finally our project overall structure will look like this
{% include figure image_path="/assets/images/custom_android_structure.png" alt="Custom Android project structure" %}

## Test Example
Let's create our first Kotlin test. Kotlin files have `.kt` extension and can be easily created in your project folder. Following simple example would be enough to ensure that everything set up correctly and Kotlin can be successfully leveraged in the further development.

```kotlin
@RunWith(AndroidJUnit4::class)
class ExampleKotlinInstrumentedTest {
 private val yourAppPackageName = "io.github.alexilyenko.sample"

 @Test
 fun useAppContext() {
  val appContext = InstrumentationRegistry.getTargetContext()
  assertEquals(yourAppPackageName, appContext.packageName)
 }
}
```
## Converting Java to Kotlin
If you happen to have some tests already written in Java and you want to migrate them to Kotlin, I have good news! Kotlin Plugin also includes tool for easy converting sources from one language to another. It can be found by navigating to **Code** > **Convert Java File to Kotlin File** menu.
{% include figure image_path="/assets/images/kotlin_convert_tool.png" alt="How to convert Java to Kotlin" caption="Converting Java to Kotlin" %}
There could be some warnings after converting. But overall the tool does surprisingly great job and you may rely on it in your development.

## Staying up-to-date with Kotlin version
In spite of all its advantages Kotlin is still young language, that's being constantly developed and changed. JetBrains team suggests everyone to stay up-to-date with new features and fixes. Android Studio plugin turns that into a no-brainer with simple channel switching under the **Tools** > **Kotlin** > **Configure  Kotlin Plugin Updates** menu. You could choose one of the three channels available - `Stable`, `Preview` or `Early Access`.
{% include figure image_path="/assets/images/update_kotlin.png" alt="Configure Kotlin updates" caption="Configuring Kotlin Updates in Android Studio" %}

At this point you should be able to start developing your Instrumentation tests in Kotlin. I'm sure you'll love this powerful language and it'll become your favorite tool for solving everyday problems. Happy coding!

[<img src="{{ site.url }}{{ site.baseurl }}/assets/images/share_message.png" alt="Feel free to share!">](https://alexilyenko.github.io/)
