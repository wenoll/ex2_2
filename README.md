# 构建Android CameraX应用

## 实验内容（1/2）

• CameraX是Android最新的支持开发相机应用的Jetpack 库（API level 21以上）

• 本实验将按照教程完成CameraX APP的构建

• 要求上传代码至Github，并撰写详细的Readme文档。

## 实验内容（2/2）

• 学习Android中布局的用法

• Android硬件权限的获取

• 掌握CameraX库的基本用法

• Preview（预览）将摄像头画面实时显示到界面上。

• ImageCapture（拍照）支持高质量静态图片捕捉。

• VideoCapture（录像）用于录制视频。

• （可选扩展）ImageAnalysis（图像分析）用于实时处理每一帧

画面，比如做机器学习推理、二维码识别等。

## 实验步骤：

### 创建项目

首先创建一个新项目，选择“Empty Activity”。

将项目命名为“CameraXApp”，软件包名称更改为“com.android.example.cameraxapp”。选择[Kotlin](https://so.csdn.net/so/search?q=Kotlin&spm=1001.2101.3001.7020)语言开发，设定最低支持的API Level 21（**CameraX 所需的最低级别**）

### 添加 Gradle 依赖

打开项目的模块（Module）的build.gradle 文件，并添加 CameraX 依赖项：

```kotlin
	implementation ("androidx.constraintlayout:constraintlayout:2.1.4")
    implementation(libs.androidx.appcompat)
    // CameraX core library using the camera2 implementation
    val camerax_version = "1.5.0-alpha06"
    // The following line is optional, as the core library is included indirectly by camera-camera2
    implementation("androidx.camera:camera-core:${camerax_version}")
    implementation("androidx.camera:camera-camera2:${camerax_version}")
    // If you want to additionally use the CameraX Lifecycle library
    implementation("androidx.camera:camera-lifecycle:${camerax_version}")
    // If you want to additionally use the CameraX VideoCapture library
    implementation("androidx.camera:camera-video:${camerax_version}")
    // If you want to additionally use the CameraX View class
    implementation("androidx.camera:camera-view:${camerax_version}")
    // If you want to additionally add CameraX ML Kit Vision Integration
    implementation("androidx.camera:camera-mlkit-vision:${camerax_version}")
    // If you want to additionally use the CameraX Extensions library
    implementation("androidx.camera:camera-extensions:${camerax_version}")
```

### 创建项目布局

#### 本项目中，涉及以下的功能：

CameraX PreviewView（用于预览相机图片/视频）。
	用于控制图片拍摄的标准按钮。
	用于开始/停止视频拍摄的标准按钮。
	用于放置 2 个按钮的垂直指南。

![屏幕截图 2025-04-30 092141](https://github.com/user-attachments/assets/59ee6819-e51f-4365-ba24-8cb5f9048fed)

### 编写 MainActivity.kt 代码

```kotlin
package com.android.example.cameraxapp

import android.Manifest
import android.content.ContentValues
import android.content.pm.PackageManager
import android.os.Build
import android.os.Bundle
import android.provider.MediaStore
import androidx.appcompat.app.AppCompatActivity
import androidx.camera.core.ImageCapture
import androidx.camera.video.Recorder
import androidx.camera.video.Recording
import androidx.camera.video.VideoCapture
import androidx.core.app.ActivityCompat
import androidx.core.content.ContextCompat
import com.android.example.cameraxapp.databinding.ActivityMainBinding
import java.util.concurrent.ExecutorService
import java.util.concurrent.Executors
import android.widget.Toast
import androidx.camera.lifecycle.ProcessCameraProvider
import androidx.camera.core.Preview
import androidx.camera.core.CameraSelector
import android.util.Log
import androidx.camera.core.ImageAnalysis
import androidx.camera.core.ImageCaptureException
import androidx.camera.core.ImageProxy
import androidx.camera.video.FallbackStrategy
import androidx.camera.video.MediaStoreOutputOptions
import androidx.camera.video.Quality
import androidx.camera.video.QualitySelector
import androidx.camera.video.VideoRecordEvent
import androidx.core.content.PermissionChecker
import java.nio.ByteBuffer
import java.text.SimpleDateFormat
import java.util.Locale

typealias LumaListener = (luma: Double) -> Unit

class MainActivity : AppCompatActivity() {
   private lateinit var viewBinding: ActivityMainBinding

   private var imageCapture: ImageCapture? = null

   private var videoCapture: VideoCapture<Recorder>? = null
   private var recording: Recording? = null

   private lateinit var cameraExecutor: ExecutorService

   override fun onCreate(savedInstanceState: Bundle?) {
       super.onCreate(savedInstanceState)
       viewBinding = ActivityMainBinding.inflate(layoutInflater)
       setContentView(viewBinding.root)

       // Request camera permissions
       if (allPermissionsGranted()) {
           startCamera()
       } else {
           ActivityCompat.requestPermissions(
               this, REQUIRED_PERMISSIONS, REQUEST_CODE_PERMISSIONS)
       }

       // Set up the listeners for take photo and video capture buttons
       viewBinding.imageCaptureButton.setOnClickListener { takePhoto() }
       viewBinding.videoCaptureButton.setOnClickListener { captureVideo() }

       cameraExecutor = Executors.newSingleThreadExecutor()
   }

   private fun takePhoto() {}

   private fun captureVideo() {}

   private fun startCamera() {}

   private fun allPermissionsGranted() = REQUIRED_PERMISSIONS.all {
       ContextCompat.checkSelfPermission(
           baseContext, it) == PackageManager.PERMISSION_GRANTED
   }

   override fun onDestroy() {
       super.onDestroy()
       cameraExecutor.shutdown()
   }

   companion object {
       private const val TAG = "CameraXApp"
       private const val FILENAME_FORMAT = "yyyy-MM-dd-HH-mm-ss-SSS"
       private const val REQUEST_CODE_PERMISSIONS = 10
       private val REQUIRED_PERMISSIONS =
           mutableListOf (
               Manifest.permission.CAMERA,
               Manifest.permission.RECORD_AUDIO
           ).apply {
               if (Build.VERSION.SDK_INT <= Build.VERSION_CODES.P) {
                   add(Manifest.permission.WRITE_EXTERNAL_STORAGE)
               }
           }.toTypedArray()
   }
}
```

## 请求必要的权限

```kotlin
<uses-feature android:name="android.hardware.camera.any" />
<uses-permission android:name="android.permission.CAMERA" />
<uses-permission android:name="android.permission.RECORD_AUDIO" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"
   android:maxSdkVersion="28" />
```

复制代码到MainActivity.kt中:

```kotlin
override fun onRequestPermissionsResult(
   requestCode: Int, permissions: Array<String>, grantResults:
   IntArray) {
   if (requestCode == REQUEST_CODE_PERMISSIONS) {
       if (allPermissionsGranted()) {
           startCamera()
       } else {
           Toast.makeText(this,
               "Permissions not granted by the user.",
               Toast.LENGTH_SHORT).show()
           finish()
       }
   }
}
```

![屏幕截图 2025-04-30 094058](https://github.com/user-attachments/assets/b71d6d88-32aa-4f2a-8be2-54b83626f0ac)


## 实现 Preview 用例

![屏幕截图 2025-04-30 094610](https://github.com/user-attachments/assets/bc8fb35c-e6ea-401e-b7ea-4e1c16ade37b)


## 实现 ImageCapture 用例（拍照功能）

![屏幕截图 2025-04-30 095207](https://github.com/user-attachments/assets/db424dcb-7d31-44ed-8e0e-0ede697832b7)

![屏幕截图 2025-04-30 095223](https://github.com/user-attachments/assets/5a72dbc8-748e-4895-bd01-1f6ba00d5a2b)


## 实现 ImageAnalysis 用例

![屏幕截图 2025-04-30 095951](https://github.com/user-attachments/assets/511187f7-d038-436f-a117-662a672fbd55)


## 实现 VideoCapture 用例（拍摄视频）

![屏幕截图 2025-04-30 101222](https://github.com/user-attachments/assets/14092072-937f-42fa-ac84-2529f392a13a)

![屏幕截图 2025-04-30 101246](https://github.com/user-attachments/assets/622cae07-2b32-43bc-89ac-95044120c6f4)

![屏幕截图 2025-04-30 101416](https://github.com/user-attachments/assets/39f281f8-d0a2-4a98-9ba1-0f89473514a0)

## 扩展实验

• 基础实验我们包含了Preview（预览）, ImageCapture（拍照）, VideoCapture（拍视频）和ImageAnalysis（图像分析）等四个

功能。

• 前面的VideoCapture步骤演示了Preview和VideoCapture的组 合，这是所有设备都支持的。

• 扩 展 实 验 可 以 尝 试 考 虑 更 多 不 同 的 组 合 ， 如 Preview + VideoCapture + ImageCapture 或 者 Preview + VideoCapture + ImageAnalysis等。

### Preview + VideoCapture + ImageCapture

![屏幕截图 2025-04-30 104008](https://github.com/user-attachments/assets/a33314e1-6230-4595-9f59-0b2a08e57ecd)

![屏幕截图 2025-04-30 104017](https://github.com/user-attachments/assets/fcfbef41-1502-4011-b026-fc784fc0ace8)

![屏幕截图 2025-04-30 104052](https://github.com/user-attachments/assets/e718228a-4b20-4d88-ba4c-9efc3d868b2c)


### Preview + VideoCapture + ImageAnalysis
![屏幕截图 2025-04-30 104319](https://github.com/user-attachments/assets/a92bfb65-c58a-42d1-a051-de21b1d65235)

![屏幕截图 2025-04-30 104336](https://github.com/user-attachments/assets/211dfa64-ada4-4751-94fe-73d9dafd0f7e)

