---
layout: post
title: Camera预览数据转换成JPEG
category: 技术
tags: [java, android]
keywords: java, android, code
description: 
---

>将camera preview数据直接保存成jpeg文件。

```java
/* callback function */
private Camera.PreviewCallback mOneShotPreviewCallback = new Camera.PreviewCallback() {
    @Override
    public void onPreviewFrame(byte[] data, Camera camera) {
        if (data.length <= 0) return;
        Size size = camera.getParameters().getPreviewSize();
        final int width = size.width;
        final int height = size.height;
        final YuvImage image = new YuvImage(data, ImageFormat.NV21, width, height, null);
        ByteArrayOutputStream os = new ByteArrayOutputStream(data.length);
        if (!image.compressToJpeg(new Rect(0, 0, width, height), 50, os)) {
            return;
        }
        byte[] tempByteArray = os.toByteArray();
        // write this tempByteArray to file.
    }
}
/* how to use it. */
mCamera.setOneShotPreviewCallback(mOneShotPreviewCallback);
```
