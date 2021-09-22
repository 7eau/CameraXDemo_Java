# Android CameraX 1.1.0 Java版本使用教程

教程参考[Android官方文档](https://developer.android.google.cn/codelabs/camerax-getting-started#0)，并将其教程代码转成`Java`语言。

## 1. 注意事项

- 最低支持API是21
- Android Studio 至少是3.6版本
- Java8环境

## 2. 创建项目

1. 新建Android项目，注意`Minimum SDK`至少是API21

2. 添加依赖（`1.0.1`版本至`1.1.0-alpha08`（博客编写时的最新版本）均支持。）

   ```
    def camerax_version = "1.1.0-alpha08"
   // CameraX core library using camera2 implementation
    implementation "androidx.camera:camera-camera2:$camerax_version"
   // CameraX Lifecycle Library
    implementation "androidx.camera:camera-lifecycle:$camerax_version"
   // CameraX View class
    implementation "androidx.camera:camera-view:1.0.0-alpha27"
   ```

3. 配置项目使用Java8。在`build.gradle(app)`文件中

   ```
   ...
   android {
   	...
   	buildTypes {
   		...	
   	}
   	compileOptions {
           sourceCompatibility JavaVersion.VERSION_1_8
           targetCompatibility JavaVersion.VERSION_1_8
       }
   }
   ```

4. Sync

5. 编辑`activity_main` layout 文件，用以下代码替换

   ```xml
   <?xml version="1.0" encoding="utf-8"?>
   <androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
       xmlns:app="http://schemas.android.com/apk/res-auto"
       xmlns:tools="http://schemas.android.com/tools"
       android:layout_width="match_parent"
       android:layout_height="match_parent"
       tools:context=".MainActivity">
   
       <Button
           android:id="@+id/camera_capture_button"
           android:layout_width="100dp"
           android:layout_height="100dp"
           android:layout_marginBottom="50dp"
           android:scaleType="fitCenter"
           android:text="Take Photo"
           app:layout_constraintLeft_toLeftOf="parent"
           app:layout_constraintRight_toRightOf="parent"
           app:layout_constraintBottom_toBottomOf="parent"
           android:elevation="2dp" />
       <androidx.camera.view.PreviewView
           android:id="@+id/viewFinder"
           android:layout_width="match_parent"
           android:layout_height="match_parent" />
   
   </androidx.constraintlayout.widget.ConstraintLayout>
   ```

6. 编辑`MainActivity`，用以下代码替换

   ```java
   package com.awo.newcameraxtest;
   
   import android.Manifest;
   import android.annotation.SuppressLint;
   import android.content.pm.PackageManager;
   import android.net.Uri;
   import android.os.Bundle;
   import android.util.Log;
   import android.widget.Button;
   import android.widget.Toast;
   
   import androidx.annotation.NonNull;
   import androidx.appcompat.app.AppCompatActivity;
   import androidx.camera.core.CameraSelector;
   import androidx.camera.core.ImageAnalysis;
   import androidx.camera.core.ImageCapture;
   import androidx.camera.core.ImageCaptureException;
   import androidx.camera.core.ImageProxy;
   import androidx.camera.core.Preview;
   import androidx.camera.lifecycle.ProcessCameraProvider;
   import androidx.camera.view.PreviewView;
   import androidx.core.app.ActivityCompat;
   import androidx.core.content.ContextCompat;
   
   import com.google.common.util.concurrent.ListenableFuture;
   
   import org.jetbrains.annotations.NotNull;
   
   import java.io.ByteArrayInputStream;
   import java.io.File;
   import java.nio.Buffer;
   import java.nio.ByteBuffer;
   import java.text.SimpleDateFormat;
   import java.util.Locale;
   import java.util.Objects;
   import java.util.concurrent.ExecutorService;
   import java.util.concurrent.Executors;
   
   public class MainActivity extends AppCompatActivity {
   
       private ImageCapture imageCapture;
       private File outputDirectory;
       private ExecutorService cameraExecutor;
   
   
       @Override
       protected void onCreate(Bundle savedInstanceState) {
           super.onCreate(savedInstanceState);
           setContentView(R.layout.activity_main);
   
           // 请求相机权限
           if (allPermissionsGranted()) {
               startCamera();
           } else {
               ActivityCompat.requestPermissions(this, Configuration.REQUIRED_PERMISSIONS,
                       Configuration.REQUEST_CODE_PERMISSIONS);
           }
   
           // 设置拍照按钮监听
           Button camera_capture_button = findViewById(R.id.camera_capture_button);
           camera_capture_button.setOnClickListener(v -> takePhoto());
   
           // 设置照片等保存的位置
           outputDirectory = getOutputDirectory();
   
           cameraExecutor = Executors.newSingleThreadExecutor();
   
   
       }
   
       private void takePhoto() {
           
       }
   
       private void startCamera() {
           
       }
   
       private boolean allPermissionsGranted() {
           for (String permission : Configuration.REQUIRED_PERMISSIONS) {
               if (ContextCompat.checkSelfPermission(this, permission)
                       != PackageManager.PERMISSION_GRANTED) {
                   return false;
               }
           }
           return true;
       }
   
       private File getOutputDirectory() {
           File mediaDir = new File(getExternalMediaDirs()[0], getString(R.string.app_name));
           boolean isExist = mediaDir.exists() || mediaDir.mkdir();
           return isExist ? mediaDir : null;
       }
   
       @Override
       protected void onDestroy() {
           super.onDestroy();
           cameraExecutor.shutdown();
       }
   
       static class Configuration {
           public static final String TAG = "CameraxBasic";
           public static final String FILENAME_FORMAT = "yyyy-MM-dd-HH-mm-ss-SSS";
           public static final int REQUEST_CODE_PERMISSIONS = 10;
           public static final String[] REQUIRED_PERMISSIONS = new String[]{Manifest.permission.CAMERA};
       }
   }
   ```

## 3. 申请相机权限

1. 打开`AndroidManifest.xml`文件，在`<application>`标签前加入权限申请代码

```xml
<uses-feature android:name="android.hardware.camera.any" />
<uses-permission android:name="android.permission.CAMERA" />
```

> `android.hardware.camera.any`：用以确保设备有相机，并且APP可以使用任何可用的相机
>
> `android.permission.CAMERA`：申请相机权限

2. 在`MainActivity`中添加以下方法

```java
@Override
public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
super.onRequestPermissionsResult(requestCode, permissions, grantResults);
if (requestCode == Configuration.REQUEST_CODE_PERMISSIONS) {
if (allPermissionsGranted()) {// 申请权限通过
startCamera();
} else {// 申请权限失败
Toast.makeText(this, "用户拒绝授予权限！", Toast.LENGTH_LONG).show();
finish();
}
}
}
```

3. 此时运行APP将首先发现系统弹出请求权限的对话框。


## 4. 实现预览用例

1. 复制以下代码到`startCamera()`方法中

```java
private void startCamera() {
// 将Camera的生命周期和Activity绑定在一起（设定生命周期所有者），这样就不用手动控制相机的启动和关闭。
ListenableFuture<ProcessCameraProvider> cameraProviderFuture = ProcessCameraProvider.getInstance(this);

cameraProviderFuture.addListener(() -> {
try {
// 将你的相机和当前生命周期的所有者绑定所需的对象
ProcessCameraProvider processCameraProvider = cameraProviderFuture.get();

// 创建一个Preview 实例，并设置该实例的 surface 提供者（provider）。
PreviewView viewFinder = (PreviewView)findViewById(R.id.viewFinder);
Preview preview = new Preview.Builder()
.build();
preview.setSurfaceProvider(viewFinder.getSurfaceProvider());

// 选择后置摄像头作为默认摄像头
CameraSelector cameraSelector = CameraSelector.DEFAULT_BACK_CAMERA;

// 重新绑定用例前先解绑
processCameraProvider.unbindAll();

// 绑定用例至相机
processCameraProvider.bindToLifecycle(MainActivity.this, cameraSelector,
preview);

} catch (Exception e) {
Log.e(Configuration.TAG, "用例绑定失败！" + e);
}
}, ContextCompat.getMainExecutor(this));

}
```

2. 现在运行APP，将看到一个预览画面


## 5. 实现拍照用例 

1. 复制以下代码到`takePhoto()`方法中

   ```java
    private void takePhoto() {
        // 确保imageCapture 已经被实例化, 否则程序将可能崩溃
        if (imageCapture != null) {
            // 创建带时间戳的输出文件以保存图片，带时间戳是为了保证文件名唯一
            File photoFile = new File(outputDirectory,
                                      new SimpleDateFormat(Configuration.FILENAME_FORMAT,
                                                           Locale.SIMPLIFIED_CHINESE).format(System.currentTimeMillis()) + ".jpg");
   
            // 创建 output option 对象，用以指定照片的输出方式
            ImageCapture.OutputFileOptions outputFileOptions = new ImageCapture.OutputFileOptions
                .Builder(photoFile)
                .build();
   
            // 执行takePicture（拍照）方法
            imageCapture.takePicture(outputFileOptions,
                                     ContextCompat.getMainExecutor(this),
                                     new ImageCapture.OnImageSavedCallback() {// 保存照片时的回调
                                         @Override
                                         public void onImageSaved(@NonNull ImageCapture.OutputFileResults outputFileResults) {
                                             Uri savedUri = Uri.fromFile(photoFile);
                                             String msg = "照片捕获成功! " + savedUri;
                                             Toast.makeText(getBaseContext(), msg, Toast.LENGTH_SHORT).show();
                                             Log.d(Configuration.TAG, msg);
                                         }
   
                                         @Override
                                         public void onError(@NonNull ImageCaptureException exception) {
                                             Log.e(Configuration.TAG, "Photo capture failed: " + exception.getMessage());
                                         }
                                     });
        }
    }
   ```

   2. 在`startCamera()`方法中，加入以下语句

      ```java
      // 创建拍照所需的实例
      imageCapture = new ImageCapture.Builder().build();
      ```

   3. 并绑定拍照用例

      ```java
      processCameraProvider.bindToLifecycle(MainActivity.this, cameraSelector,
                              preview,
                              imageCapture);
      ```

   4. 现在你运行APP，点击拍照按钮，将可以看到APP能够拍照并将照片保存。


## 6. 实现预览帧分析

1. 在`MainActivity`中加入以下代码

   ```java
   private static class MyAnalyzer implements ImageAnalysis.Analyzer{
       @SuppressLint("UnsafeOptInUsageError")
       @Override
       public void analyze(@NonNull ImageProxy image) {
           Log.d(Configuration.TAG, "Image's stamp is " + Objects.requireNonNull(image.getImage()).getTimestamp());
           image.close();
       }
   }
   ```

   - 这里我改写了官方教程的分析类，因为这部分代码我实在是不熟悉如何转成Java代码。
   - 这个分析类就是打印每一个预览帧画面的时间戳。

2. 在`startCamera()`中`imageCapture`部分后加入以下代码

   ```java
    // 设置预览帧分析
   ImageAnalysis imageAnalysis = new ImageAnalysis.Builder()
       .build();
   imageAnalysis.setAnalyzer(cameraExecutor, new MyAnalyzer());
   ```

3. 绑定分析用例

   ```java
   // 绑定用例至相机
   processCameraProvider.bindToLifecycle(MainActivity.this, cameraSelector,
                                         preview,
                                         imageCapture,
                                         imageAnalysis);
   ```

现在`startCamera()`的代码应该和下方代码一致：

```java
private void startCamera() {
    // 将Camera的生命周期和Activity绑定在一起（设定生命周期所有者），这样就不用手动控制相机的启动和关闭。
    ListenableFuture<ProcessCameraProvider> cameraProviderFuture = ProcessCameraProvider.getInstance(this);

    cameraProviderFuture.addListener(() -> {
        try {
            // 将你的相机和当前生命周期的所有者绑定所需的对象
            ProcessCameraProvider processCameraProvider = cameraProviderFuture.get();

            // 创建一个Preview 实例，并设置该实例的 surface 提供者（provider）。
            PreviewView viewFinder = (PreviewView)findViewById(R.id.viewFinder);
            Preview preview = new Preview.Builder()
                .build();
            preview.setSurfaceProvider(viewFinder.getSurfaceProvider());

            // 选择后置摄像头作为默认摄像头
            CameraSelector cameraSelector = CameraSelector.DEFAULT_BACK_CAMERA;

            // 创建拍照所需的实例
            imageCapture = new ImageCapture.Builder().build();

            // 设置预览帧分析
            ImageAnalysis imageAnalysis = new ImageAnalysis.Builder()
                .build();
            imageAnalysis.setAnalyzer(cameraExecutor, new MyAnalyzer());

            // 重新绑定用例前先解绑
            processCameraProvider.unbindAll();

            // 绑定用例至相机
            processCameraProvider.bindToLifecycle(MainActivity.this, cameraSelector,
                                                  preview,
                                                  imageCapture,
                                                  imageAnalysis);

        } catch (Exception e) {
            Log.e(Configuration.TAG, "用例绑定失败！" + e);
        }
    }, ContextCompat.getMainExecutor(this));

}
```

4. 现在运行APP，并查看`Logcat`日志，可以看到输出的预览图像的时间戳信息。

## 恭喜你！

以上就是所有使用`CameraX`的教程，接下来根据自己的需要修改或查看最新API文档即可。

