# Android-OpenCV
An Android Studio project which has a module that contains OpenCV SDK files ported and configured to use CMake and Gradle plugin 2.3.1 or above, making it easy to include OpenCV into Android applications.

## Integration

Currently the doc contains details of integrating the ```opencv``` module into an exiting git repo of an Android Studio project using ```git submodule```. Sooner or later I'll inclde steps to include the ```opencv``` module without using ```git submodule```. It is better to understand how ```git submodule``` works via this link: [Git - Submodules](https://git-scm.com/book/en/v2/Git-Tools-Submodule). For team managed git repos, read the section **Cloning a Project with Submodules** carefully.

Within the porject directory, open bash/cmd and run the following command:
```
git submodule add https://github.com/ahasbini/Android-OpenCV.git android-opencv
```

Then within the project directory, edit ```settings.gradle``` file and append the following and do gradle sync:
```
include ':android-opencv:opencv'
```

Within the Project View in Android Studio, right click and choose "Configure project subset" and check ```opencv```. Then do a gradle sync. You should see ```opencv``` as a module as the image below:

![Project View](https://i.stack.imgur.com/qgXf7.png)

Then within the ```app``` module directory (or application module that you're developing), create or alter the ```CMakeLists.txt``` file and add the following lines (Note that if you you are not developing a native library, you can skip this step):
```
set(OpenCV_DIR "../opencv/src/sdk/native/jni")
find_package(OpenCV REQUIRED)
message(STATUS "OpenCV libraries: ${OpenCV_LIBS}")
target_link_libraries(YOUR_TARGET_LIB ${OpenCV_LIBS})
```

Then within the ```app``` module directory (or application module that you're developing), alter it's ```build.gradle``` file as such (Note that if you you are not developing a native library, only the ```compile``` line is needed):
```
apply plugin: 'com.android.application'
...
android {
    ...
    defaultConfig {
        ...
        externalNativeBuild {
            cmake {
                cppFlags "-frtti -fexceptions"
                abiFilters 'x86', 'x86_64', 'armeabi', 'armeabi-v7a', 'arm64-v8a', 'mips', 'mips64'
            }
        }
    }
    buildTypes {
        ...
    }
    externalNativeBuild {
        cmake {
            path "CMakeLists.txt"
        }
    }
}

dependencies {
    ...
    compile project(':opencv')
}
```

After final step, OpenCV would be included within the application and thier APIs could be accessed from within the Java and Native C/C++ code. To check if OpenCV is working properly, add the following in an ```Application```, ```Activity``` or any class to load OpenCV:
```
public class MyClass {

    static {
        if (BuildConfig.DEBUG) {
            OpenCVLoader.initDebug();
        }
    }

    ...
}
```

Then build and launch the application. Once the class is loaded, the logcat will display OpenCV messages as below (the first error is normal):
```
05-10 10:42:31.451 D/OpenCV/StaticHelper: Trying to get library list
05-10 10:42:31.452 E/OpenCV/StaticHelper: OpenCV error: Cannot load info library for OpenCV
05-10 10:42:31.452 D/OpenCV/StaticHelper: Library list: ""
05-10 10:42:31.452 D/OpenCV/StaticHelper: First attempt to load libs
05-10 10:42:31.452 D/OpenCV/StaticHelper: Trying to init OpenCV libs
05-10 10:42:31.452 D/OpenCV/StaticHelper: Trying to load library opencv_java3
05-10 10:42:32.031 D/OpenCV/StaticHelper: Library opencv_java3 loaded
05-10 10:42:32.031 D/OpenCV/StaticHelper: First attempt to load libs is OK
05-10 10:42:32.045 I/OpenCV/StaticHelper: General configuration for OpenCV 3.2.0 =====================================
05-10 10:42:32.045 I/OpenCV/StaticHelper:   Version control:               3.2.0
05-10 10:42:32.045 I/OpenCV/StaticHelper:   Platform:
05-10 10:42:32.045 I/OpenCV/StaticHelper:     Timestamp:                   2016-12-23T13:04:49Z
05-10 10:42:32.045 I/OpenCV/StaticHelper:     Host:                        Linux 4.8.0-25-generic x86_64
05-10 10:42:32.045 I/OpenCV/StaticHelper:     Target:                      Linux 1 x86_64
05-10 10:42:32.045 I/OpenCV/StaticHelper:     CMake:                       2.8.12.2
05-10 10:42:32.045 I/OpenCV/StaticHelper:     CMake generator:             Ninja
05-10 10:42:32.045 I/OpenCV/StaticHelper:     CMake build tool:            /usr/bin/ninja
05-10 10:42:32.045 I/OpenCV/StaticHelper:     Configuration:               Release
05-10 10:42:32.045 I/OpenCV/StaticHelper:   C/C++:
05-10 10:42:32.045 I/OpenCV/StaticHelper:     Built as dynamic libs?:      NO
05-10 10:42:32.045 I/OpenCV/StaticHelper:     C++ Compiler:                /usr/bin/ccache /opt/android/android-ndk-r10e/toolchains/x86_64-4.9/prebuilt/linux-x86_64/bin/x86_64-linux-android-g++ (ver 4.9)
```

Also, you could inspect the built ```apk``` under ```app_module_directory/build/outputs``` to see that the ```libopenc_java3.so``` is included within the abi architecture folders (drag the apk onto the text editor tabs of Android Studio):

![APK Inspection](https://i.stack.imgur.com/YRsH4.png)

## Updating OpenCV

I'll be monitoring OpenCV's SDK and releases on their website in order to re-port it with their updated files. Please post an issue if I haven't done so within a **day or two**.

In order to update the OpenCV module to the latest version simply open a bash/cmd in ```project_root_directory/android-opencv/``` and run the command:
```
git pull
```

Once the command finishes, the ```opencv``` module will be updated to the latest commit on the repo. Do a gradle sync and compile to make sure the update was successful.

## Checkout to a certain version of OpenCV

With each release of OpenCV SDK a tag with the OpenCV version will be created on the commit of the release. To checkout to a certain version of the OpenCV library, open a bash/cmd in ```project_root_directory/android-opencv/``` and run the command:
```
git pull
git checkout version-number
```
Where ```version-number``` is the version of OpenCV you'd like to checkout to (example: 3.2.0). The minimum version of the OpenCV within the repo is 3.2.0.
