---
layout: post
title: android广播
category: 技术
tags: [java, android]
keywords: java, android, broadcast
description: 
---
> 记录平时少用，但非常重要的android广播信息

#wifi密码错误广播

```java
private BroadcastReceiver mReceiver = new BroadcastReceiver() {
    String action = intent.getAction();
    if (action.equals(WifiManager.SUPPLICANT_STATE_CHANGED_ACTION)) {
        int errorCode = intent.getIntExtra(WifiManager.EXTRA_SUPPLICANT_ERROR, Integer.MIN_VALUE);
        if (errorCode == WifiManager.ERROR_AUTHENTICATING) {
            // means wifi password is error.
        }
    }
}
```
参考：
    https://developer.android.com/reference/android/net/wifi/WifiManager.html#EXTRA_SUPPLICANT_ERROR
    https://developer.android.com/reference/android/net/wifi/WifiManager.html#ERROR_AUTHENTICATING
