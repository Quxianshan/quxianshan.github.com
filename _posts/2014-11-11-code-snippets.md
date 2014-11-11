---
layout: post
title: 常用代码片断
category: 技术
tags: [java, android]
keywords: java, android, code
description: 
---

> 此文章用于记录遇到的一些常用的代码片断, 方便后续重复使用

## 读取android节点信息

```java
public void readNodeInfo(String node){
	String result = "";
	try{
		FileReader mReader = new FileReader(node);
		BufferedReader mBuffer = new BufferedReader(mReader, 1024);
		result = mBuffer.readLine(); // for single line
		// for multi-line
		//String temp = "";
		//while((temp = mBuffer.readLine()) != null){
		//	result += (temp + "\n");
		//}
		mBuffer.close();
	}catch(IOException e){
		e.printStackTrace();
	}
	Log.d("quinn", "node info is : " + result);
}
```

## 执行shell命令并读取执行结果

```java
public void execCommand(String commandStr){
    try{
        Process mProcess = Runtime.getRuntime().exec("sh"); 
	// or Runtime.getRuntime().exec("su"); if root needed.
	OutputStream out = mProcessgetOutputStream();
        DataOutputStream mDataOutputStream = new DataOutputStream(out);
        mDataOutputStream.write(commandStr.getBytes());
        mDataOutputStream.flush();
        // if commandStr is simple command:
	// 	like echo .. we just close mDataOutputStream here:
        mDataOutputStream.close();
        // if commandStr is complicated command:
	//	like logcat .. we need use while loop to read:
	InputStreamReader inStream = new InputStreamReader(mProcess.getInputStream());
        BufferedReader inBuffer = new BufferedReader(inStream);
	InputStreamReader errStream = new InputStreamReader(mProcess.getErrorStream());
        BufferedReader errBuffer = new BufferedReader(errStream);
        String inStr = "", errStr = "";
        Log.d("quinn", "normal log:");
        while((inStr = inBuffer.readLine()) != null){
            Log.d("quinn", inStr);            
        }
        Log.d("quinn", "err log:");
        while((errStr = errBuffer.readLine()) != null){
            Log.d("quinn", errStr);
        }
        // close resources here
        inBuffer.close();
        errBuffer.close();
    }catch(IOException e){
	Log.d(TAG, "some err hanppends:" + e.toString());
    }
}
```

## android 执行shell命令截图:

```bash
screencap /mnt/sdcard/screenshot.png
```
或者执行
```bash
cat /dev/graphics/fb0 > /mnt/sdcard/screenshot.raw
ffmpeg -vcodec rawvideo -f rawvideo -pix_fmt rgb32 -s 800x480 -i screenshot.raw -f image2 -vcodec png screenshot.png
```

## jni模板
```cpp
#include <stdio.h>
#include <stdlib.h>
#include <android_runtime/AndroidRuntime.h>
#include <jni.h>
#include <JNIHelp.h>
namespace android {
	static const char *classPathName = "class_path_name";
	
	static JNINativeMethod methods[] = {
        {"java native function", "return type", cpp native function},
	};

	//  the code below stays the same

	extern "C" int registerNativeMethods (JNIEnv* env, const char* className, JNINativeMethod* gMethods, int numMethods) {
        jclass clazz;
        clazz = env->FindClass(className);

        if (clazz == NULL) {
            ALOGE("Native registration unable to find class '%s'", className);
            return JNI_FALSE;
        }
    
        if (env->RegisterNatives(clazz, gMethods, numMethods) < 0){
            ALOGE("RegisterNatives failed for '%s'", className);
            return JNI_FALSE;
        }

        return JNI_TRUE;
    
    }

    // Register native methods for all classes we know about.
    // returns JNI_TURE on success
    extern "C" int registerNatives (JNIEnv* env) {
        if(!registerNativeMethods(env, classPathName, methods, sizeof(methods) / sizeof(methods[0]))) {
            return JNI_FALSE;
        }

        return JNI_TRUE;
    }

    // this is called by the VM when the shared library is first loaded.
    typedef union {
        JNIEnv* env;
        void* venv;
    } UnionJNIEnvToVoid;

    extern "C" jint JNI_OnLoad(JavaVM* vm, void* reserved) {
        UnionJNIEnvToVoid uenv;
        uenv.venv = NULL;
        jint result = -1;
        JNIEnv* env = NULL;

        ALOGI("JNI_Onload");
        if (vm->GetEnv(&uenv.venv, JNI_VERSION_1_4) != JNI_OK) {
            ALOGE("ERROR: GetEnv failed.");
            goto bail;
        }
        env = uenv.env;

        if (registerNatives(env) != JNI_TRUE) {
            ALOGE("ERROR: registerNatives failed.");
            goto bail;
        }

        result = JNI_VERSION_1_4;
    bail:
        return result;
    }
}
```