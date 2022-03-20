# Android CameraX 1.1.0 Java版本使用教程

教程参考[Android官方文档](https://developer.android.com/codelabs/camerax-getting-started#0)，并将其教程代码转成`Java`语言。

## 1. 注意事项

- 最低支持API是21
- Android Studio 至少是3.6版本
- Java8环境

## 2. 创建项目

1. 新建Android项目，注意`Minimum SDK`至少是API21

2. 在`app`模块的`build.gradle`文件中添加依赖（`1.1.0-beta01`）

   ```
   def camerax_version = "1.1.0-beta01"
   implementation "androidx.camera:camera-core:${camerax_version}"
   implementation "androidx.camera:camera-camera2:${camerax_version}"
   implementation "androidx.camera:camera-lifecycle:${camerax_version}"
   implementation "androidx.camera:camera-video:${camerax_version}"
   
   implementation "androidx.camera:camera-view:${camerax_version}"
   implementation "androidx.camera:camera-extensions:${camerax_version}"
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

4. 此代码实验室使用 [ViewBinding](https://developer.android.com/topic/libraries/view-binding)，因此请使用以下命令启用它（在 `android{}` 块的末尾）：

   ```
   buildFeatures {
      viewBinding true
   }
   ```

4. Sync

6. 编辑`activity_main` layout 文件，用以下代码替换

   ```xml
   <?xml version="1.0" encoding="utf-8"?>
   <androidx.constraintlayout.widget.ConstraintLayout
      xmlns:android="http://schemas.android.com/apk/res/android"
      xmlns:app="http://schemas.android.com/apk/res-auto"
      xmlns:tools="http://schemas.android.com/tools"
      android:layout_width="match_parent"
      android:layout_height="match_parent"
      tools:context=".MainActivity">
   
      <androidx.camera.view.PreviewView
          android:id="@+id/viewFinder"
          android:layout_width="match_parent"
          android:layout_height="match_parent" />
   
      <Button
          android:id="@+id/image_capture_button"
          android:layout_width="110dp"
          android:layout_height="110dp"
          android:layout_marginBottom="50dp"
          android:layout_marginEnd="50dp"
          android:elevation="2dp"
          android:text="@string/take_photo"
          app:layout_constraintBottom_toBottomOf="parent"
          app:layout_constraintLeft_toLeftOf="parent"
          app:layout_constraintEnd_toStartOf="@id/vertical_centerline" />
   
      <Button
          android:id="@+id/video_capture_button"
          android:layout_width="110dp"
          android:layout_height="110dp"
          android:layout_marginBottom="50dp"
          android:layout_marginStart="50dp"
          android:elevation="2dp"
          android:text="@string/start_capture"
          app:layout_constraintBottom_toBottomOf="parent"
          app:layout_constraintStart_toEndOf="@id/vertical_centerline" />
   
      <androidx.constraintlayout.widget.Guideline
          android:id="@+id/vertical_centerline"
          android:layout_width="wrap_content"
          android:layout_height="wrap_content"
          android:orientation="vertical"
          app:layout_constraintGuide_percent=".50" />
   
   </androidx.constraintlayout.widget.ConstraintLayout>
   ```

6. 编辑`MainActivity`，用以下代码替换

   ```java
   import android.Manifest;
   import android.annotation.SuppressLint;
   import android.content.ContentValues;
   import android.content.pm.PackageManager;
   import android.os.Build;
   import android.os.Bundle;
   import android.provider.MediaStore;
   import android.util.Log;
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
   import androidx.camera.video.MediaStoreOutputOptions;
   import androidx.camera.video.Quality;
   import androidx.camera.video.QualitySelector;
   import androidx.camera.video.Recorder;
   import androidx.camera.video.Recording;
   import androidx.camera.video.VideoCapture;
   import androidx.camera.video.VideoRecordEvent;
   import androidx.camera.view.PreviewView;
   import androidx.core.app.ActivityCompat;
   import androidx.core.content.ContextCompat;
   
   import com.google.common.util.concurrent.ListenableFuture;
   
   import java.io.File;
   import java.text.SimpleDateFormat;
   import java.util.Locale;
   import java.util.Objects;
   import java.util.concurrent.ExecutorService;
   import java.util.concurrent.Executors;
   
   public class MainActivity extends AppCompatActivity {
   
       private ActivityMainBinding viewBinding;
   
       private ImageCapture imageCapture = null;
   
       private VideoCapture videoCapture = null;
       private Recording recording = null;
   
       private ExecutorService cameraExecutor;
   
   
       @Override
       protected void onCreate(Bundle savedInstanceState) {
           super.onCreate(savedInstanceState);
           viewBinding = ActivityMainBinding.inflate(getLayoutInflater());
           setContentView(viewBinding.getRoot());
   
           // 请求相机权限
           if (allPermissionsGranted()) {
               startCamera();
           } else {
               ActivityCompat.requestPermissions(this, Configuration.REQUIRED_PERMISSIONS,
                       Configuration.REQUEST_CODE_PERMISSIONS);
           }
   
           // 设置拍照按钮监听
           viewBinding.imageCaptureButton.setOnClickListener(v -> takePhoto());
           viewBinding.videoCaptureButton.setOnClickListener(v -> captureVideo());
   
           cameraExecutor = Executors.newSingleThreadExecutor();
   
   
       }
   
       private void captureVideo() {}
   
       private void takePhoto() {}
   
       private void startCamera() {}
   
       private boolean allPermissionsGranted() {
           for (String permission : Configuration.REQUIRED_PERMISSIONS) {
               if (ContextCompat.checkSelfPermission(this, permission)
                       != PackageManager.PERMISSION_GRANTED) {
                   return false;
               }
           }
           return true;
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
           public static final int REQUEST_AUDIO_CODE_PERMISSIONS = 12;
           public static final String[] REQUIRED_PERMISSIONS =
                   Build.VERSION.SDK_INT <= Build.VERSION_CODES.P ?
                           new String[]{Manifest.permission.CAMERA,
                                   Manifest.permission.RECORD_AUDIO,
                                   Manifest.permission.WRITE_EXTERNAL_STORAGE} :
                           new String[]{Manifest.permission.CAMERA,
                                   Manifest.permission.RECORD_AUDIO};
       }
   }
   ```

## 3. 申请相机权限

1. 打开`AndroidManifest.xml`文件，在`<application>`标签前加入权限申请代码

```xml
<uses-feature android:name="android.hardware.camera.any" />
<uses-permission android:name="android.permission.CAMERA" />
<uses-permission android:name="android.permission.RECORD_AUDIO" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"
   android:maxSdkVersion="28" />
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
    } else if (requestCode == Configuration.REQUEST_AUDIO_CODE_PERMISSIONS) {
        if (ContextCompat.checkSelfPermission(this,
                                              "Manifest.permission.RECORD_AUDIO") != PackageManager.PERMISSION_GRANTED) {
            Toast.makeText(this, "未授权录制音频权限！", Toast.LENGTH_LONG).show();
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
                // 将你的相机绑定到APP进程的lifecycleOwner中
                ProcessCameraProvider processCameraProvider = cameraProviderFuture.get();

                // 创建一个Preview 实例，并设置该实例的 surface 提供者（provider）。
                PreviewView viewFinder = (PreviewView)findViewById(R.id.viewFinder);
                Preview preview = new Preview.Builder()
                        .build();
                preview.setSurfaceProvider(viewBinding.viewFinder.getSurfaceProvider());

                // 选择后置摄像头作为默认摄像头
                CameraSelector cameraSelector = CameraSelector.DEFAULT_BACK_CAMERA;

                // 重新绑定用例前先解绑
                processCameraProvider.unbindAll();
                
                processCameraProvider.bindToLifecycle(MainActivity.this, cameraSelector,
                        preview);

            } catch (Exception e) {
                Log.e(Configuration.TAG, "用例绑定失败！" + e);
            }
        }, /*在主线程运行*/ContextCompat.getMainExecutor(this));

    }
```

2. 现在运行APP，将看到一个预览画面


## 5. 实现拍照用例 

1. 复制以下代码到`takePhoto()`方法中

   ```java
    private void takePhoto() {
           // 确保imageCapture 已经被实例化, 否则程序将可能崩溃
           if (imageCapture != null) {
               // 创建一个 "MediaStore Content" 以保存图片，带时间戳是为了保证文件名唯一
               String name = new SimpleDateFormat(Configuration.FILENAME_FORMAT,
                       Locale.SIMPLIFIED_CHINESE).format(System.currentTimeMillis());
               ContentValues contentValues = new ContentValues();
               contentValues.put(MediaStore.MediaColumns.DISPLAY_NAME, name);
               contentValues.put(MediaStore.MediaColumns.MIME_TYPE, "image/jpeg");
               if (Build.VERSION.SDK_INT > Build.VERSION_CODES.P) {
                   contentValues.put(MediaStore.Images.Media.RELATIVE_PATH, "Pictures/CameraX-Image");
               }
   
               // 创建 output option 对象，用以指定照片的输出方式。
               // 在这个对象中指定有关我们希望输出如何的方式。我们希望将输出保存在 MediaStore 中，以便其他应用可以显示它
               ImageCapture.OutputFileOptions outputFileOptions = new ImageCapture.OutputFileOptions
                       .Builder(getContentResolver(),
                           MediaStore.Images.Media.EXTERNAL_CONTENT_URI,
                           contentValues)
                       .build();
   
               // 设置拍照监听，用以在照片拍摄后执行takePicture（拍照）方法
               imageCapture.takePicture(outputFileOptions,
                       ContextCompat.getMainExecutor(this),
                       new ImageCapture.OnImageSavedCallback() {// 保存照片时的回调
                           @Override
                           public void onImageSaved(@NonNull ImageCapture.OutputFileResults outputFileResults) {
                               String msg = "照片捕获成功! " + outputFileResults.getSavedUri();
                               Toast.makeText(getBaseContext(), msg, Toast.LENGTH_SHORT).show();
                               Log.d(Configuration.TAG, msg);
                           }
   
                           @Override
                           public void onError(@NonNull ImageCaptureException exception) {
                               Log.e(Configuration.TAG, "Photo capture failed: " + exception.getMessage());// 拍摄或保存失败时
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

## 7. 实现视频录制用例

CameraX在1.1.0-alpha10版本中添加了VideoCapture用例，并且从那时起一直在进行进一步的改进。请注意，`VideoCapture` API 支持许多视频捕获功能，因此为了保持此代码实验室的可管理性，此代码实验室仅演示如何将视频和音频捕获到 `MediaStore`.

1. 将此代码复制到 `captureVideo()`方法中：它控制 `VideoCapture` 用例的启动和停止。下面的要点将分解我们刚刚复制的代码。

   ```java
   @SuppressLint("CheckResult")
       private void captureVideo() {
           // 确保videoCapture 已经被实例化，否则程序可能崩溃
           if (videoCapture != null) {
               // 禁用UI，直到CameraX 完成请求
               viewBinding.videoCaptureButton.setEnabled(false);
   
               Recording curRecording = recording;
               if (curRecording != null) {
                   // 如果正在录制，需要先停止当前的 recording session（录制会话）
                   curRecording.stop();
                   recording = null;
                   return;
               }
   
               // 创建一个新的 recording session
               // 首先，创建MediaStore VideoContent对象，用以设置录像通过MediaStore的方式保存
               String name = new SimpleDateFormat(Configuration.FILENAME_FORMAT, Locale.SIMPLIFIED_CHINESE)
                       .format(System.currentTimeMillis());
               ContentValues contentValues = new ContentValues();
               contentValues.put(MediaStore.MediaColumns.DISPLAY_NAME, name);
               contentValues.put(MediaStore.MediaColumns.MIME_TYPE, "video/mp4");
               if (Build.VERSION.SDK_INT > Build.VERSION_CODES.P) {
                   contentValues.put(MediaStore.Video.Media.RELATIVE_PATH, "Movies/CameraX-Video");
               }
   
               MediaStoreOutputOptions mediaStoreOutputOptions = new MediaStoreOutputOptions
                       .Builder(getContentResolver(), MediaStore.Video.Media.EXTERNAL_CONTENT_URI)
                       .setContentValues(contentValues)
                       .build();
               // 申请音频权限
               if (ActivityCompat.checkSelfPermission(this, Manifest.permission.RECORD_AUDIO) != PackageManager.PERMISSION_GRANTED) {
                   ActivityCompat.requestPermissions(this,
                           new String[]{Manifest.permission.RECORD_AUDIO},
                           Configuration.REQUEST_CODE_PERMISSIONS);
               }
               Recorder recorder = (Recorder) videoCapture.getOutput();
               recording = recorder.prepareRecording(this, mediaStoreOutputOptions)
                   	.withAudioEnabled() // 开启音频录制
                   	// 开始新录制
                       .start(ContextCompat.getMainExecutor(this), videoRecordEvent -> {
                           if (videoRecordEvent instanceof VideoRecordEvent.Start) {
                               viewBinding.videoCaptureButton.setText(getString(R.string.stop_capture));
                               viewBinding.videoCaptureButton.setEnabled(true);// 启动录制时，切换按钮显示文本
                           } else if (videoRecordEvent instanceof VideoRecordEvent.Finalize) {//录制完成后，使用Toast通知
                               if (!((VideoRecordEvent.Finalize) videoRecordEvent).hasError()) {
                                   String msg = "Video capture succeeded: " +
                                           ((VideoRecordEvent.Finalize) videoRecordEvent).getOutputResults()
                                           .getOutputUri();
                                   Toast.makeText(getBaseContext(), msg, Toast.LENGTH_SHORT).show();
                                   Log.d(Configuration.TAG, msg);
                               } else {
                                   if (recording != null) {
                                       recording.close();
                                       recording = null;
                                       Log.e(Configuration.TAG, "Video capture end with error: " +
                                               ((VideoRecordEvent.Finalize) videoRecordEvent).getError());
                                   }
                               }
                               viewBinding.videoCaptureButton.setText(getString(R.string.start_capture));
                               viewBinding.videoCaptureButton.setEnabled(true);
                           }
                       });
           }
       }
   ```

   2. 在 `startCamera()`，将以下代码放在`preview`创建行之后。这将创建`VideoCapture`用例。

      ```java
      // 创建录像所需实例
      Recorder recorder = new Recorder.Builder()
          .setQualitySelector(QualitySelector.from(Quality.HIGHEST))
          .build();
      videoCapture = VideoCapture.withOutput(recorder);
      ```

   3. 将`videoCapture`实例绑定到`Lifecycle`中

      ```
      processCameraProvider.bindToLifecycle(MainActivity.this, cameraSelector,
                              preview,
                              imageCapture/*,
                              imageAnalysis*/,
                              videoCapture);
      ```

      Note：

      ```java
      /*
      此步骤具有 Camera2 设备文档中指定的设备级别要求:
          预览 + 视频捕获 + 图像捕获：LIMITED设备及以上。
          预览 + 视频捕获 + 图像分析：LEVEL_3（最高）设备添加到 Android 7（N）。
          预览 + 视频捕获 + 图像分析 + 图像捕获：不支持。
      */
      ```

   4. 现在运行代码，则可以录制一段视频，并在系统的图库中看到这段视频。

## 恭喜你！

以上就是所有使用`CameraX`的教程，接下来根据自己的需要修改或查看最新API文档即可。

