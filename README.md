## General guidelines

We start with a sensible set of code style guidelines from the official sources:

- Kotlin: https://kotlinlang.org/docs/reference/coding-conventions.html
- Java: https://google.github.io/styleguide/javaguide.html
- Android: https://source.android.com/setup/contribute/code-style

A general rule of thumb is to make the code as concise as possible, but make sure to still keep the source readable. On top of the default guidelines, we apply a custom set of rules that we consider useful:

- Column width: **120** characters
- Indentation: **2** characters
- Continuation indentation: **2** characters
- Indentation type: **Space** _(not Tab)_
- Field/property notation: **None** (don't use the _`m`_ prefix as seen in Hungarian notation)
    - ```java
      String mName; // BAD
      String name; // GOOD
      ```
- Maximum joined blank lines: **1**. In general, don't use unnecessary line breaks, only use them to separate code blocks or logical code groups
- For tests annotated with `@Test`, don't use the `test` prefix in function name as it is unnecessary
- Do not use any abbreviations at all, make all declarations simple and clear:
    - ```java
      LinearLayout llMain;                 // BAD
      LinearLayout mainContainer;          // GOOD
      ```
    - ```kotlin
      lateinit var txtName: TextView;      // BAD
      lateinit var nameTextView: TextView; // GOOD
      ```
- In case of a single short annotation, we write them inline with the function/method or field/parameter declaration, for example:
    - ```java
      @Autowired String appName;
      @Test void runningWithScissors() {
        // test code here
      }
      ```
    - ```kotlin
      @Volatile lateinit var dataStore: Storage
      @UiThread fun escapeTheBugs() = runWithScissors()
      ```
- Try to cover as much as your production code with tests. We don't need to go overboard, but we strive to have all code units properly tested. Each project will have its own testing rules so make sure you follow that same framework

## Kotlin-specific guidelines

We have additional rules that apply to the Kotlin programming language:

- If the function or property type can be easily inferred from the context, don't specify the type explicitly, like so:
    - ```kotlin
      val hintText = "Don't click here"
      val user = UserModel("Mark", 31)
      val lock: ConcurrencyLock = Locks.createNewLock(this).withAcquire().setup()
      ```
- Companion objects should be at the top of the class, instead of at the bottom
- Constants are declared at the top of the file, more precisely
    - Privately visible constants (`private const val`) are placed outside of the class declaration, in `UPPER_SNAKE_CASE`, and separated by a blank line from the class declaration
    - Publicly visible constants (`const val`) are placed in the companion objects, also in `UPPER_SNAKE_CASE`
    - Non-constant properties are written in normal `camelCase`, even `val` properties
    - Example of these:
    - ```kotlin
      private const val MY_PRIVATE_KEY = "private_key"
      // leave one empty line
      class MyClass {
        companion object {
          const val MY_PUBLIC_KEY = "public_key"
          val keysMerged = "$MY_PUBLIC_KEY/$MY_PRIVATE_KEY/suffix"
        }
      }
      ```
- Classes (data and regular) with too many constructor arguments should be newlined at the opening parenthesis, like so:
    - ```kotlin
      data class PersonalInfo(
        private val name: String,
        private val age: Int = 30,
        val hasProtection: Boolean = true,
        val isValid: Boolean = false
      ) {
        // class code here
      }
      ```
- The same goes for methods with many arguments, for example:
    - ```kotlin
      fun <T> computeUsingArguments(
        argument0: T,
        argument1: String,
        argument2: Int = 0,
        argument3: String = "<empty>",
        argument4: String? = null,
        errorCallback: (Throwable) -> Unit
      ): T {
        // function code here
      }
      ```
- When appropriate, replace `null` checks with a `let` block
- When appropriate, use the `apply` scoping function as an inline function body to return the created object after invoking functions on it. A good example is having setters in a builder class:
    - ```kotlin
      fun withUsername(username: String) = apply {
        this.username = username
      }
      ```
- In case of argument ambiguity, use named arguments to make the code more readable
- When appropriate, use extension functions to make the code more readable
- Use Kotlin's collection utility functions to make the code more readable
- Use Kotlin's stream API and built-in functions like `filter`, `map`, etc.
- If possible, replace builders with constructors (with named arguments and default argument values if necessary)
- Be careful with `this` keyword and out-of-scope variables when using Kotlin's scoping functions 

To help out everyone involved, we use this graphic to figure out Kotlin's scoping functions:

![Kotlin Scoping Functions](https://i.imgur.com/Ny9rJQR.png)

## Converting Java to Kotlin guidelines

We convert the file using IDEA's built-in Java to Kotlin converter. After converting, we fix the following things:

- Remove `internal` modifiers (keep only when really necessary)
- Use `lateinit` where it's possible to do so
- Remove generics where it's possible to do so (Kotlin is better at types)
- Use `given` instead of `when` (or see project's docs for other rules)
- Add `@JvmStatic` for companion (or plain) object functions which are used from Java code
- Use default values in functions instead of overloading
- Use `const` for constants in and out of companion objects 
- Optimize imports and reformat file according to our style guides
- Convert easily extractable functions with a single parameter to extension functions if appropriate
- Replace multiple immediate setter function invocations with an `apply` block on the target
- Replace multiple usages of a long-named variable with a `with` around it
- In interfaces and abstract classes, use `val` properties with getters only for truly immutable data, otherwise declare a getter function
- Replace incorrectly converted delegated getter properties with functions if necessary
- In case of performance concerns, add `lazy` delegation if appropriate
- Use function expressions for one-line functions
- Remove single arguments from lambdas if they are not used, or replace them with `it` if used
- Remove extra parentheses around lambda arguments if they are the only argument (pull lambdas out of parentheses)
- Use Kotlin's `emptyList()` and `listOf()`/`mutableListOf()` initializer functions instead of Java's `Collections` methods
- Remove unnecessary type declarations generated by the converter
- Use Kotlin's built-in `copy` function when converting `AutoValue` classes to data classes

In case of doubt, contact the maintainers or check the project docs. For most projects, we will try to use a `lint` tool to verify that everything we do is per the guidelines above.

## Android-specific guidelines

We have additional rules that apply to the Android platform:

- Refer the inherited class in the subclasses, ie: use `Fragment`, `Activity`, `Adapter` suffixes.
    - ```kotlin
      class MainPage : Fragment()                    // BAD
      class MainFragment : Fragment()                // GOOD
      ```
    - ```java
      public class MainPage extends Activity { }     // BAD
      public class MainActivity extends Activity { } // GOOD
      ```
- Use a layered naming structure for String resources:
    - ```xml
      <string name="login_name">Your name:</string>
      ```
- Where possible, use Kotlin Android Extensions:
    - https://kotlinlang.org/docs/tutorials/android-plugin.html
- Because we mostly use Kotlin Android Extensions, and this plugin generates synthetic delegated properties for finding views, we use `camelCase` naming convention (instead of Google-proposed `snake_case`) for view IDs in XML layouts
- In order to avoid conflict and confusion, we use `{feature_name}{description}{view}` format, for example `loginNameLabel`, or `registrationBirthdayInput`
- Closing tags should always be on a new line in XML layouts:
    - ```xml
      <TextView
        android:id="@+id/text_view"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        /> <!-- Closing tag is on a new line -->
      ```
    - The reasoning behind this decision is: if you add attributes to XML in a way that the last attribute changes (or format code around the last attribute), the closing tag `/>` goes on a new line. This means that in code diff you will see an extra modification line, thus messing up the history (and `blame` results) for the future -- this is something we want to avoid

## Applying guidelines

To apply these guidelines, you can download our styling configuration files and import them to your IDE. Both Java and Kotlin style configurations are in this repository, named `java_code_style.xml` and `kotlin_code_style.xml`.

To import, go to your IntelliJ IDEA (or Android Studio) Preferences, then Editor -> Code Style -> Java (or Kotlin, respectively) and on top of your right pane there will be an import button available. Just select the appropriate file and save. Verify from the dropdown menu that the appropriate style is being used.

## Android testing

We run a suite of tests:

- **Unit tests** for isolated units of code (we mock using Mockito)
- **Functional tests** for isolated feature checks and user interaction tests, such as testing individual fragments or activities (usually using Espresso and Robot framework). These run offline in most cases, and external dependencies are mocked to behave in an expected way
- **Acceptance tests**, a suite that builds on top of Functional tests, tests whole features in isolation. External dependencies are still mocked to behave in an expected way. We test all happy paths of all user stories
- **Integration** (or **end-to-end** tests) where we test how our apps work while communicating with other (external) apps and services. These run online, in a staging environment, and we test the most business-oriented user stories

To set up your device for functional/acceptance tests:

- Enable _"Don't keep activities"_ developer setting (this makes sure your state saving works too)
- Turn off _all animations_ from developer settings
- Enable _"Stay awake while charging"_ developer setting
- Set AM/PM time format to project's default (check project docs)
- Login with a Google account and update _Google Play Services_
- Disable _auto-update_ for all apps from Google Play settings
- Disable _auto-update_ for the Android OS
- Disable Wi-Fi (functional tests run offline)

For information on how to run test suites, refer to specific project documentation.