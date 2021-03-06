---
layout: post
title: android 常用设置修改
category: 技术
tags: [settings, android]
keywords: settings, android
description: 
---

> 记录经常使用的一些android设置

## 默认开启以太网设置
文件路径：android/frameworks/base/packages/SettingsProvider/res/values/defaults.xml     

```bash
<!--  0 == mobile, 1 == wifi. -->
<integer name="def_network_preference">9</integer>
```     
由xml文件中的注释可知，0代表默认的网络类型为移动数据，1代表默认的网络类型为wifi，以太网的定义为9。详细的定义配置参见：     
android/frameworks/base/core/res/res/values/config.xml     

```bash
<!-- XXXXX NOTE THE FOLLOWING RESOURCES USE THE WRONG NAMING CONVENTION.
    Please don't copy them, copy anything else. -->
<!-- This string array should be overridden by the device to present a list of network
    attributes.  This is used by the connectivity manager to decide which networks can coexist
    based on the hardware -->
<!-- An Array of "[Connection name],[ConnectivityManager.TYPE_xxxx],
    [associated radio-type],[priority],[restoral-timer(ms)],[dependencyMet]  -->
<!-- the 5th element "resore-time" indicates the number of milliseconds to delay
    before automatically restore the default connection.  Set -1 if the connection
    does not require auto-restore. -->
<!-- the 6th element indicates boot-time dependency-met value. -->
<string-array translatable="false" name="networkAttributes">
    <item>"wifi,1,1,1,-1,true"</item>
    <item>"ethernet,9,9,0,-1,true"</item>
    <item>"mobile,0,0,0,-1,true"</item>
    <item>"mobile_mms,2,0,2,60000,true"</item>
    <item>"mobile_supl,3,0,2,60000,true"</item>
    <item>"mobile_hipri,5,0,3,60000,true"</item>
    <item>"mobile_fota,10,0,2,60000,true"</item>
    <item>"mobile_ims,11,0,2,60000,true"</item>
    <item>"mobile_cbs,12,0,2,60000,true"</item>
    <item>"wifi_p2p,13,1,0,-1,true"</item>
</string-array>
```
以上是网络类型的基本配置，那如何应用这些配置呢？首先需要让这些配置生效，详见文件：    
frameworks/base/packages/SettingsProvider/src/com/android/providers/settings/DatabaseHelper.java

```java
private void loadGlobalSettings(SQLiteDatabase db) {
    ...
    loadIntegerSetting(stmt, Settings.Global.NETWORK_PREFERENCE,
        R.integer.def_network_preference);
    ...
```
DatabaseHelper会将定义的值写到数据库当中。在ConnectivityService.java中读取并设置网络模式:    
frameworks/base/services/java/com/android/server/ConnectivityService.java

```java
public ConnectivityService(Context context, INetworkManagementService netManager,
    INetworkStatsService statsService, INetworkPolicyManager policyManager,
    NetworkFactory netFactory) {
    ...
    mNetworkPreference = getPersistedNetworkPreference();
    ...
}

...

private int getPersistedNetworkPreference() {
    final ContentResolver cr = mContext.getContentResolver();
    final int networkPrefSetting = Settings.Global
        .getInt(cr, Settings.Global.NETWORK_PREFERENCE, -1);
    if (networkPrefSetting != -1) {
        return networkPrefSetting;
    }
    return ConnectivityManager.DEFAULT_NETWORK_PREFERENCE;
}
```
