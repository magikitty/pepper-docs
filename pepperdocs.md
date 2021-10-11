# PepperDocs

This document contains instructions for setting up your environment for developing code for the Pepper robot in the Android Studio IDE. This document also includes some problems you may come across and how to fix them. There are also some useful tips and limitations we faced during development.

## References

- [Pepper](https://us.softbankrobotics.com/pepper)
- [Follow Pepper SDK Installation Guide](https://developer.softbankrobotics.com/pepper-qisdk/getting-started/installing-pepper-sdk-plug) (instructions also for Windows and Mac)

## Linux Setup

These steps have been tested on Debian 10 (Buster) and Ubuntu 20 (Focal Fossa).

### Enable Virtualization

- [Enable hardware acceleration](https://developer.android.com/studio/run/emulator-acceleration):
  - Boot into your BIOS (find instructions specific to your machine online)
  - Intel: Enable `Intel Virtualization Technology extensions` e.g. `VT`, `VT-x`, `vmx`
  - AMD: Enable `AMD Virtualization extensions` e.g. `AMD-V`, `SVM`

### Install Android Studio

- Check whether your OS is 64-bit architecture, if so:
  - Execute `sudo apt-get install libc6:i386 libncurses5:i386 libstdc++6:i386 lib32z1 libbz2-1.0:i386`
- Download [Android Studio](https://developer.android.com/studio/)
- Extract Android Studio into your desired location
  - e.g. for tar.gz file `tar -xvzf <filename>` into `/user/local/lib` or other desired location

### Install Latest Android SDK, Tools and Pepper SDK

- Tools -> SDK Manager -> Check "Show Package Details"
- Select latest release version of Android -> Edit -> Next
- Click SDK Tools -> Select highest version corresponding  to SDK major version installed -> Apply
- File -> Settings -> Plugins -> Search "Pepper" -> Select "Pepper SDK" -> Click Install -> Click Restart IDE
- Once Android Studio has opened again, select to install the Pepper SDK tools
- In File -> Settings -> Tools -> Pepper plugin settings -> Debug options select "Enable debug" to get helpful
  debugging messages

### Pepper SDK Confirmation

- View -> Appearance -> Toolbar
- In Robot SDK Manager select all components of the latest API version -> Apply -> OK

## Issues and Fixes

Below are some issues you may encounter and ways to fix them.

- Build error: `Manifest merger failed: Attribute application@appComponentFactory`
  - Add the following lines to the `AndroidManifest.xml` file's `<application>` element:
    - `tools:replace="android:appComponentFactory"`
    - `android:appComponentFactory="androidx.core.app.CoreComponentFactory"`
  - Add `xmlns:tools="http://schemas.android.com/tools"` to the `AndroidManifest.xml` file's `<xml>` tag
  - Add `android.enableJetifier=true` to the `gradle.properties` file
- Build error: `Could not find org.jetbrains.kotlin:kotlin-gradle-plugin:1.5.0-release-764.` (or some other version)
  - Replace the version in `build.gradle` file's `ext.kotlin_version = "<VERSION>"` to the latest that can be found at [mavenrepository.com](https://mvnrepository.com/artifact/org.jetbrains.kotlin/kotlin-gradle-plugin) e.g. `ext.kotlin_version = "1.5.0"`
- Runtime error on start emulator: `./robot_viewer: /home/username/.local/share/Softbank Robotics/RobotSDK/API7/tools/bin/../lib/libz.so.1: version 'ZLIB_1.2.9' not found (required by /lib/x86_64-linux-gnu/libpng16.so.16)`
  - Backup your current `libz.so.1` file with `mv libz.so.1 libz.so.1.old`
  - Create a new version of `libz.so.1` as a symbolic/soft link to `/lib/x86_64-linux-gnu` with: `ln -s /lib/x86_64-linux-gnu/libz.so.1 libz.so.1`
- `Adb connection Error:EOF`: remove wrong versions/unused avd emulators in Android Studio with AVD Manager
- Imports don't work: Once you have the required tools and SDK you should be able to import necessary libraries by
  mousing over functions and clicking `Import` or by pressing `Alt+Shift+Enter`. If you do not get the option to import
  libraries go to File -> Sync project with gradle files and then try again

## Quick Start Using Kotlin

First create a new robot project following the [instructions](https://developer.softbankrobotics.com/pepper-qisdk/getting-started/creating-robot-application).

Then add the following code to `MainActivity.kt`.

This program makes Pepper say "Hello Human" when the app is launched. Test it with your emulator and Pepper to make sure that your environment and IDE are configured correctly.

```kotlin
import android.os.Bundle
import com.aldebaran.qi.sdk.QiContext
import com.aldebaran.qi.sdk.QiSDK
import com.aldebaran.qi.sdk.RobotLifecycleCallbacks
import com.aldebaran.qi.sdk.`object`.conversation.Say
import com.aldebaran.qi.sdk.builder.SayBuilder
import com.aldebaran.qi.sdk.design.activity.RobotActivity

class MainActivity : RobotActivity(), RobotLifecycleCallbacks {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // Set layout to render UI from activity_main.xml
        setContentView(R.layout.activity_main)
        // Register the RobotLifecycleCallbacks to this Activity.
        QiSDK.register(this, this)
    }

    override fun onDestroy() {
        // Unregister the RobotLifecycleCallbacks for this Activity.
        super.onDestroy()
        QiSDK.unregister(this, this)
    }

    override fun onRobotFocusGained(qiContext: QiContext) {
        // The robot focus is gained.

        // Create say action with context, set text, build say action
        val say: Say = SayBuilder.with(qiContext).withText("Hello human!").build()
        // Execute say action
        say.run()
    }

    override fun onRobotFocusLost() {
        // The robot focus is lost.
    }

    override fun onRobotFocusRefused(reason: String) {
        // The robot focus is refused.
    }
}
```

## Connecting to Pepper

- [Instructions](https://developer.softbankrobotics.com/pepper-qisdk/getting-started/running-application#running-application)
  for running an app on Pepper
- Only one computer may be connected to Pepper at a time
- To connect to Pepper you should be on the same network as Pepper (you can check Pepper's network from its tablet)
- On Pepper's tablet go to Settings and make sure that Developer mode is activated
- Activate ADB from Settings -> Developer options -> Debugging -> ADB on Pepper's tablet
- Make sure that your run configuration in Android Studio is `app`

## Tips

- Pepper cannot move autonomously if it is charging or if the socket cover is up
- Pepper can be pushed around by people if it is charging or if the socket cover is up
- Pepper's shoulders will light up blue when Pepper is listening to human speech
- There is a speech bar on Pepper's tablet screen (can be enabled if not already enabled)
  - A blue bar indicates Pepper is listening, a grey bar indicates not listening
  - A question mark will appear in the bar if Pepper heard some speech, but cannot understand what it was
  - If Pepper understood what was said, the speech bar will display what Pepper heard

## Limitations

Below are some limitations/problems we came across and could not fix. This limits Pepper's functionality. However, there may be solutions that we did not find.

- The wildcard operator `*` does not work
  - This operator should allow Pepper to understand (and optionally store) whatever speech it hears from a person, 
     even if the dialogue option has not been predefined
  - Without being able to use the wildcard all anticipated speech from a person must be hardcoded into e.g. the 
     topic file for a chat, so Pepper knows exactly what it expects to hear
  - This makes developing deep and natural dialogue difficult, since all possible dialogue options cannot be anticipated
- Pepper's emotion detection is not always accurate
  - Pepper's emotion detection is a built-in function and emotions can be determined by combining Pepper's observed 
     excitement state and pleasure state
  - Since these functions are built-in we did not find a way to make the emotion detection more accurate
- Pepper does not always understand what a person says even if Pepper has been programmed for the expected response.
   A person may have to repeat themselves several times before Pepper understands what they are saying.