---
layout: post
title: 常用代码片断
category: 技术
tags: [java, android]
keywords: java, android, code
description: 
---

> 此文章用于记录遇到的一些常用的代码片断, 方便后续重复使用

## android 文件浏览器列表

```java
package xiongqi.venom;

import android.content.Intent;
import android.os.Bundle;
import android.os.Environment;
import android.support.annotation.Nullable;
import android.support.v7.app.AppCompatActivity;
import android.support.v7.widget.ListViewCompat;
import android.support.v7.widget.Toolbar;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.AdapterView;
import android.widget.BaseAdapter;
import android.widget.ImageView;
import android.widget.TextView;

import com.orhanobut.logger.Logger;

import java.io.File;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.Comparator;
import java.util.Stack;

import rx.Observable;
import rx.Observer;
import rx.android.schedulers.AndroidSchedulers;
import rx.functions.Func1;
import rx.schedulers.Schedulers;

/**
 * Created by quinn on 16-8-25.
 */
public class FileListActivity extends AppCompatActivity {


    ListViewCompat fileListView = null;
    private Stack<String> filePathStack = null;

    private AdapterView.OnItemClickListener fileListItemClickListener = new AdapterView.OnItemClickListener() {
        @Override
        public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
            File clickFile = (File) fileListView.getAdapter().getItem(position);
            if  (clickFile.isDirectory()) {
                loadFileListObservable(clickFile);
            } else {
                Intent fileSelectIntent = new Intent();
                fileSelectIntent.putExtra("file_path", clickFile.getAbsolutePath());
                setResult(RESULT_OK, fileSelectIntent);
                finish();
            }
        }
    };

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Logger.init(FileListActivity.class.getSimpleName());
        Logger.d("fileListActivity on create");
        setContentView(R.layout.activity_file_list);
        initToolBar();
        fileListView = (ListViewCompat) findViewById(R.id.file_list);
        fileListView.setOnItemClickListener(fileListItemClickListener);
        Logger.d("external storage state :%s, storage dir :%s", Environment.getExternalStorageState(), Environment.getExternalStorageDirectory());
        filePathStack = new Stack<String>();
        loadFileListObservable(Environment.getExternalStorageDirectory());
    }

    void initToolBar() {
        Toolbar toolbar = (Toolbar) findViewById(R.id.file_list_toolbar);
        setSupportActionBar(toolbar);
        toolbar.setTitle(null);
        toolbar.findViewById(R.id.file_list_toolbar_back).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Logger.d("user click back navigation button, current position: %s", filePathStack.peek());
                if (!filePathStack.peek().equals(Environment.getExternalStorageDirectory().getAbsolutePath())) {
                    loadFileListObservable(new File(filePathStack.pop()).getParentFile());
                } else {
                    setResult(RESULT_CANCELED);
                    finish();
                }
            }
        });
    }

    void loadFileListObservable(final File file) {
        Observable<File> observable = Observable.just(file);
        observable.subscribeOn(Schedulers.io())
                .map(new Func1<File, File[]>() {
                    @Override
                    public File[] call(File file) {
                        return file.listFiles();
                    }
                })
                .filter(new Func1<File[], Boolean>() {
                    @Override
                    public Boolean call(File[] files) {
                        return files != null;
                    }
                })
                .map(new Func1<File[], File[]>() {
                    @Override
                    public File[] call(File[] files) {
                        ArrayList<File> fileArrayList = new ArrayList<File>();
                        for (File temp : files) {
                            if (!temp.isHidden()) {
                                fileArrayList.add(temp);
                            }
                        }
                        return fileArrayList.toArray(new File[fileArrayList.size()]);
                    }
                })
                .flatMap(new Func1<File[], Observable<FileAdapter>>() {
                    @Override
                    public Observable<FileAdapter> call(File[] files) {
                        Logger.i("get files null ? %s ", files == null);
                        Arrays.sort(files, COMPARATOR);
                        return Observable.just(new FileAdapter(files));
                    }
                })
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Observer<FileAdapter>() {
                    @Override
                    public void onCompleted() {}

                    @Override
                    public void onError(Throwable e) {
                        Logger.e(e, "get error when subscribe FileAdapter");
                    }

                    @Override
                    public void onNext(FileAdapter kosyniFileAdapter) {
                        filePathStack.push(file.getAbsolutePath());
                        fileListView.setAdapter(kosyniFileAdapter);
                    }
                });
    }

    private final FileComparator COMPARATOR = new FileComparator();
    private class FileComparator implements Comparator<File> {

        @Override
        public int compare(File lhs, File rhs) {
            if ((lhs.isDirectory() && !rhs.isDirectory())
                    || (!lhs.isDirectory() && rhs.isDirectory())) {
                return lhs.isDirectory() ? -1 : 1;
            } else if (lhs.isDirectory() && rhs.isDirectory()) {
                return lhs.getName().compareTo(rhs.getName());
            } else {
                return lhs.getName().compareTo(rhs.getName());
            }
        }
    }

    private class FileAdapter extends BaseAdapter {

        private File[] files = null;

        public FileAdapter(File[] files) {
            this.files = files;
        }

        @Override
        public int getCount() {
            return this.files.length;
        }

        @Override
        public Object getItem(int position) {
            return this.files[position];
        }

        @Override
        public long getItemId(int position) {
            return position;
        }

        @Override
        public View getView(int position, View convertView, ViewGroup parent) {
            LayoutInflater inflater = LayoutInflater.from(FileListActivity.this);
            ViewHolder viewHolder = null;
            if (convertView == null) {
                convertView = inflater.inflate(R.layout.item_activity_filelist_item, null);
                viewHolder = new ViewHolder();
            } else {
                viewHolder = (ViewHolder) convertView.getTag(R.id.activity_filelist_file_title);
            }

            viewHolder.fileName = (TextView) convertView.findViewById(R.id.file_title);
            viewHolder.isDir = (ImageView) convertView.findViewById(R.id.is_dir);
            viewHolder.fileName.setText(this.files[position].getName());
            viewHolder.isDir.setVisibility(this.files[position].isDirectory() ? View.VISIBLE : View.GONE);
            convertView.setTag(R.id.activity_filelist_file_title, viewHolder);
            return convertView;
        }

        private class ViewHolder {
            public TextView fileName;
            public ImageView isDir;
        }
    }
}
```

## vim 里替换^M符号

```shell
:%s/^M//g (注意命令中的^M需要用CTRL+V、CTRL+M的方式输入)
```

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

## android 4.4 全屏模式设置

将如下代码写在onCreate()的setContentView()之前.

```java
View decorView = getWindow().getDecorView();
decorView.setSystemUiVisibility( View.SYSTEM_UI_FLAG_LAYOUT_STABLE
	| View.SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION
	| View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN
	| View.SYSTEM_UI_FLAG_HIDE_NAVIGATION
	| View.SYSTEM_UI_FLAG_FULLSCREEN
	| View.SYSTEM_UI_FLAG_IMMERSIVE_STICKY);
```
