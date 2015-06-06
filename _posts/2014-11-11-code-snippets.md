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

## android计算场景声音

```java
static final int SAMPLE_RATE_IN_HZ = 8000;
final int BUFFER_SIZE = AudioRecord.getMinBufferSize(SAMPLE_RATE_IN_HZ,
	AudioFormat.CHANNEL_IN_DEFAULT, AudioFormat.ENCODING_PCM_16BIT) * 10;
private AudioRecord mAudioRecord = new AudioRecord(MediaRecorder.AudioSource.MIC,
	SAMPLE_RATE_IN_HZ, AudioFormat.CHANNEL_IN_DEFAULT,
	AudioFormat.ENCODING_PCM_16BIT, BUFFER_SIZE);
short[] buffer = new short[BUFFER_SIZE]; // data buffer read from AudioRecord
int length = 0; // data length
long values = 0L; // data value
if (mAudioRecord == null) {
	throw new RuntimeException("init AudioRecord failed");
}
mAudioRecord.startRecording();
while (true) {
	values = 0L;
	length = mAudioRecord.read(buffer, 0, BUFFER_SIZE);
	if (length != 0) {
		for (int i = 0; i < buffer.length; i+=2) {
			values += ((buffer[i] | (buffer[i+1] << 8)) / length);
		}
		double mean =1.0d * values * values;
		double volume = 10 * Math.log10(mean / 32767 / 32767);
		// volume is current db.
	}
}
```

##android 通过反射添加外部应用资源

```java
public static final String CLAZZ_DRAWABLE 	= "third_part_app_package_name.R$drawable";
public static final String CLAZZ_RAW 		= "third_part_app_package_name.R$raw";
try {
	Field imgField = Class.forName(CLAZZ_DRAWABLE).getField("图片资源名，如ic_launcher");
	int imageResId = imgFiled.getInt(imgField)); // 图片资源resId
	Filed rawField = Class.forName(CLAZZ_RAW).getField("资源名");
	int rawResId = rawField.getInt(rawField)); // raw 资源Id
} catch (Exception e) {
	Log.e(TAG, "some error happens when expression update : " + e.toString());
}
```

## vim cscope 数据库生成

```shell
find . -iname "*.java" -o -iname "*.aidl" -o -iname "*.cpp" -o -iname "*.c" -o -iname "*.h" > cscope.files
cscope -Rbq
```
