# Android-OpenCV
An Android Studio project which has a module that contains OpenCV SDK files ported and configured to 
use CMake and Gradle plugin 2.3.1 or above, making it easy to include OpenCV into Android applications.

[![Android Arsenal](https://img.shields.io/badge/Android%20Arsenal-Android--OpenCV-brightgreen.svg?style=flat)](https://android-arsenal.com/details/3/6360)

## Integration

Currently the doc contains details of integrating the ```opencv``` module into an exiting git repo 
of an Android Studio project using ```git submodule```. 

**Note: if your project is not versioned by git, it will not work.** You could enable git by running 
the following command in the project root directory, but careful not do so if your project is 
versioned by other VCS tools:
```
git init
```

Sooner or later I'll include steps to include the ```opencv``` module without using 
```git submodule``` and for projects that are not versioned by git. It is better to understand how 
```git submodule``` works via this link: 
[Git - Submodules](https://git-scm.com/book/en/v2/Git-Tools-Submodule). For team managed git 
repositories, read the section **Cloning a Project with Submodules** carefully.

Within the project directory, open bash/cmd and run the following command:
```
git submodule add https://github.com/ahasbini/Android-OpenCV.git android-opencv
```

Then within the project directory, edit ```settings.gradle``` file and append the following and do 
Gradle Sync <img src="https://developer.android.com/studio/images/buttons/toolbar-sync-gradle.png" width="16px" height="16px"/>:
```
include ':android-opencv:opencv'
```

Within the Project View in Android Studio, right click and choose "Configure project subset" and 
check ```opencv```. Then do a Gradle Sync 
Gradle Sync <img src="https://developer.android.com/studio/images/buttons/toolbar-sync-gradle.png" width="16px" height="16px"/>. 
You should see ```opencv``` as a module as the image below:

![Project View](https://i.stack.imgur.com/qgXf7.png)

Then within the ```app``` module directory (or application module that you're developing), create or 
alter the ```CMakeLists.txt``` file and add the following lines (Note that if you you are not 
developing a native library, you can skip this step):
```
set(OpenCV_DIR "../android-opencv/opencv/src/sdk/native/jni")
find_package(OpenCV REQUIRED)
message(STATUS "OpenCV libraries: ${OpenCV_LIBS}")
target_link_libraries(YOUR_TARGET_LIB ${OpenCV_LIBS})
```

Then within the ```app``` module directory (or application module that you're developing), alter 
it's ```build.gradle``` file as such (Note that if you you are not developing a native library, 
only the ```compile``` line is needed):
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
                //Choose the abi architectures that you would like to compile to
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
    compile project(':android-opencv:opencv')
}
```

After final step, OpenCV would be included within the application and their APIs could be accessed 
from within the Java and Native C/C++ code. To check if OpenCV is working properly, add the 
following in an ```Application```, ```Activity``` or any class to load OpenCV (make sure that the 
class will be accessed at runtime):
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

Then build and launch the application. Once the class is loaded, the logcat will display OpenCV 
messages as below (the first error is normal):
```
10-21 16:53:28.399 D/OpenCV/StaticHelper: Trying to get library list
10-21 16:53:28.419 E/OpenCV/StaticHelper: OpenCV error: Cannot load info library for OpenCV
10-21 16:53:28.419 D/OpenCV/StaticHelper: Library list: ""
10-21 16:53:28.419 D/OpenCV/StaticHelper: First attempt to load libs
10-21 16:53:28.419 D/OpenCV/StaticHelper: Trying to init OpenCV libs
10-21 16:53:28.420 D/OpenCV/StaticHelper: Trying to load library opencv_java3
10-21 16:53:28.428 I/OpenCV: calling android_getCpuFeatures() ...
10-21 16:53:28.430 I/OpenCV: calling android_getCpuFeatures() ... Done (1f7ff)
10-21 16:53:28.441 D/OpenCV/StaticHelper: Library opencv_java3 loaded
10-21 16:53:28.441 D/OpenCV/StaticHelper: First attempt to load libs is OK
10-21 16:53:28.444 I/OpenCV/StaticHelper: General configuration for OpenCV 3.3.0 =====================================
10-21 16:53:28.444 I/OpenCV/StaticHelper:   Version control:               3.3.0
10-21 16:53:28.444 I/OpenCV/StaticHelper:   Platform:
10-21 16:53:28.444 I/OpenCV/StaticHelper:     Timestamp:                   2017-08-04T00:30:10Z
10-21 16:53:28.444 I/OpenCV/StaticHelper:     Host:                        Linux 4.8.0-58-generic x86_64
10-21 16:53:28.444 I/OpenCV/StaticHelper:     Target:                      Linux 1 armv7-a
10-21 16:53:28.444 I/OpenCV/StaticHelper:     CMake:                       2.8.12.2
10-21 16:53:28.444 I/OpenCV/StaticHelper:     CMake generator:             Ninja
10-21 16:53:28.444 I/OpenCV/StaticHelper:     CMake build tool:            /usr/bin/ninja
10-21 16:53:28.444 I/OpenCV/StaticHelper:     Configuration:               Release
10-21 16:53:28.444 I/OpenCV/StaticHelper:   CPU/HW features:
10-21 16:53:28.444 I/OpenCV/StaticHelper:     Baseline:                    NEON NEON
10-21 16:53:28.444 I/OpenCV/StaticHelper:       requested:                 DETECT
10-21 16:53:28.444 I/OpenCV/StaticHelper:       required:                  NEON
10-21 16:53:28.444 I/OpenCV/StaticHelper:       disabled:                  VFPV3
10-21 16:53:28.444 I/OpenCV/StaticHelper:   C/C++:
10-21 16:53:28.445 I/OpenCV/StaticHelper:     Built as dynamic libs?:      NO
10-21 16:53:28.445 I/OpenCV/StaticHelper:     C++ Compiler:                /usr/bin/ccache /opt/android/android-ndk-r10e/toolchains/arm-linux-androideabi-4.8/prebuilt/linux-x86_64/bin/arm-linux-androideabi-g++ (ver 4.8)
```

Also, you could inspect the built ```apk``` under ```app_module_directory/build/outputs``` to see 
that the ```libopenc_java3.so``` is included within the abi architecture folders (drag the apk onto 
the text editor tabs of Android Studio):

![APK Inspection](https://i.stack.imgur.com/YRsH4.png)

## Updating OpenCV

I'll be monitoring OpenCV's SDK and releases on their website in order to re-port it with their 
updated files. Please post an issue if I haven't done so within a **day or two**.

In order to update the OpenCV module to the latest version simply open a bash/cmd in 
```project_root_directory/android-opencv/``` and run the command:
```
git pull
```

Once the command finishes, the ```opencv``` module will be updated to the latest commit on the repo. 
Do a Gradle Sync <img src="https://developer.android.com/studio/images/buttons/toolbar-sync-gradle.png" width="16px" height="16px"/>, 
refresh linked C++ projects (**Build > Refresh Linked C++ Projects**) and compile to make sure 
the update was successful.

## Checkout to a certain version of OpenCV

With each release of OpenCV SDK a tag with the OpenCV version will be created on the commit of the 
release. To checkout to a certain version of the OpenCV library, open a bash/cmd in 
```project_root_directory/android-opencv/``` and run the command:
```
git pull
git checkout version-number
```
Where ```version-number``` is the version of OpenCV you'd like to checkout to (example: 3.2.0). The 
minimum version of the OpenCV within the repo is 3.2.0.

Don't forget to do a Gradle Sync <img src="https://developer.android.com/studio/images/buttons/toolbar-sync-gradle.png" width="16px" height="16px"/>, 
refresh linked C++ projects (**Build > Refresh Linked C++ Projects**) and compile to make sure 
the checkout was successful.
