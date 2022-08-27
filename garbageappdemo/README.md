> 原文博客：[Doi技术团队](http://blog.doiduoyi.com)<br/>
> 链接地址：[https://blog.doiduoyi.com/authors/1584446358138](https://blog.doiduoyi.com/authors/1584446358138)<br/>
> 初心：记录优秀的Doi技术团队学习经历<br/>
>本文链接：[基于MNN在Android手机上实现图像分类](https://blog.doiduoyi.com/articles/1599314168812.html)<br/>

# 前言

[comment]: <> (/*)

[comment]: <> (MNN是一个轻量级的深度神经网络推理引擎，在端侧加载深度神经网络模型进行推理预测。目前，MNN已经在阿里巴巴的手机淘宝、手机天猫、优酷等20多个App中使用，覆盖直播、短视频、搜索推荐、商品图像搜索、互动营销、权益发放、安全风控等场景。此外，IoT等场景下也有若干应用。)

[comment]: <> (下面就介绍如何使用MNN在Android设备上实现图像分类。)


[comment]: <> (# 编译库和转换模型)

[comment]: <> (## 编译MNN的Android动态库)

[comment]: <> (1. 在 `https://developer.android.com/ndk/downloads/`下载安装NDK，建议使用最新稳定版本)

[comment]: <> (2. 在 .bashrc 或者 .bash_profile 中设置 NDK 环境变量，例如：`export ANDROID_NDK=/Users/username/path/to/android-ndk-r14b`)

[comment]: <> (3. `cd /path/to/MNN`)

[comment]: <> (4. `./schema/generate.sh`)

[comment]: <> (5. `cd project/android`)

[comment]: <> (6. 编译armv7动态库：`mkdir build_32 && cd build_32 && ../build_32.sh`)

[comment]: <> (7. 编译armv8动态库：`mkdir build_64 && cd build_64 && ../build_64.sh`)

[comment]: <> (## 模型转换)

[comment]: <> (执行下面命令，得到模型转换工具 `MNNConvert`。)

[comment]: <> (```bash)

[comment]: <> (cd MNN/)

[comment]: <> (./schema/generate.sh)

[comment]: <> (mkdir build)

[comment]: <> (cd build)

[comment]: <> (cmake .. -DMNN_BUILD_CONVERTER=true && make -j4)

[comment]: <> (```)

[comment]: <> (通过以下命令可以把其他框架的模型转换为MNN模型。)

[comment]: <> (**TensorFlow -> MNN**)

[comment]: <> (把Tensorflow的冻结图模型转换为MNN模型，bizCode指定标记码，这个随便吧。如果冻结图转换不成功，可以使用下面的Tensorflow Lite模型，这个通常会成功。)

[comment]: <> (```bash)

[comment]: <> (./MNNConvert -f TF --modelFile XXX.pb --MNNModel XXX.mnn --bizCode biz)

[comment]: <> (```)

[comment]: <> (**TensorFlow Lite -> MNN**)

[comment]: <> (把Tensorflow Lite的模型转换为MNN模型，bizCode指定标记码。)

[comment]: <> (```bash)

[comment]: <> (./MNNConvert -f TFLITE --modelFile XXX.tflite --MNNModel XXX.mnn --bizCode biz)

[comment]: <> (```)

[comment]: <> (**Caffe -> MNN**)

[comment]: <> (把Caffe的模型转换为MNN模型，bizCode指定标记码。)

[comment]: <> (```bash)

[comment]: <> (./MNNConvert -f CAFFE --modelFile XXX.caffemodel --prototxt XXX.prototxt --MNNModel XXX.mnn --bizCode biz)

[comment]: <> (```)

[comment]: <> (**ONNX -> MNN**)

[comment]: <> (把ONNX 的模型转换为MNN模型，bizCode指定标记码。)

[comment]: <> (```bash)

[comment]: <> (./MNNConvert -f ONNX --modelFile XXX.onnx --MNNModel XXX.mnn --bizCode biz)

[comment]: <> (```)

[comment]: <> (# Android应用开发)

[comment]: <> (把生成的C++的头文件放在 `app/include/MNN/`目录下，把生成的动态库文件放在 `app/src/main/jniLibs/`目录下，在 `app/src/main/cpp/`目录下编写JNI的C++代码，`com.yeyupiaoling.mnnclassification.mnn`包下放JNI的java代码和MNN的相关工具类，将转换的模型放在`assets`目录下。)

[comment]: <> (## MNN工具)

[comment]: <> (编写一个[MNNClassification.java]&#40;https://github.com/yeyupiaoling/ClassificationForAndroid/blob/master/MNNClassification/app/src/main/java/com/yeyupiaoling/mnnclassification/mnn/MNNClassification.java&#41;工具类，关于MNN的操作都在这里完成，如加载模型、预测。在构造方法中，通过参数传递的模型路径加载模型，在加载模型的时候配置预测信息，例如是否使用CPU或者GPU，同时获取网络的输入输出层。同时MNN还提供了很多的图像预处理工具，对图像的预处理非常简单。要注意的是图像的均值 `dataConfig.mean`和标准差 `dataConfig.normal`，还有图片的输入通道顺序 `dataConfig.dest`，因为在训练的时候图像预处理可能不一样的，有些读者出现在电脑上准确率很高，但在手机上准确率很低，多数情况下就是这个图像预处理做得不对。)

[comment]: <> (```java)

[comment]: <> (public MNNClassification&#40;String modelPath&#41; throws Exception {)

[comment]: <> (    dataConfig = new MNNImageProcess.Config&#40;&#41;;)

[comment]: <> (    dataConfig.mean = new float[]{128.0f, 128.0f, 128.0f};)

[comment]: <> (    dataConfig.normal = new float[]{0.0078125f, 0.0078125f, 0.0078125f};)

[comment]: <> (    dataConfig.dest = MNNImageProcess.Format.RGB;)

[comment]: <> (    imgData = new Matrix&#40;&#41;;)

[comment]: <> (    File file = new File&#40;modelPath&#41;;)

[comment]: <> (    if &#40;!file.exists&#40;&#41;&#41; {)

[comment]: <> (        throw new Exception&#40;"model file is not exists!"&#41;;)

[comment]: <> (    })

[comment]: <> (    try {)

[comment]: <> (        mNetInstance = MNNNetInstance.createFromFile&#40;modelPath&#41;;)

[comment]: <> (        MNNNetInstance.Config config = new MNNNetInstance.Config&#40;&#41;;)

[comment]: <> (        config.numThread = NUM_THREADS;)

[comment]: <> (        config.forwardType = MNNForwardType.FORWARD_CPU.type;)

[comment]: <> (        mSession = mNetInstance.createSession&#40;config&#41;;)

[comment]: <> (        mInputTensor = mSession.getInput&#40;null&#41;;)

[comment]: <> (    } catch &#40;Exception e&#41; {)

[comment]: <> (        e.printStackTrace&#40;&#41;;)

[comment]: <> (        throw new Exception&#40;"load model fail!"&#41;;)

[comment]: <> (    })

[comment]: <> (})

[comment]: <> (```)

[comment]: <> (为了兼容图片路径和Bitmap格式的图片预测，这里创建了两个重载方法，它们都是通过调用 `predict&#40;&#41;`)

[comment]: <> (```java)

[comment]: <> (public int predictImage&#40;String image_path&#41; throws Exception {)

[comment]: <> (    if &#40;!new File&#40;image_path&#41;.exists&#40;&#41;&#41; {)

[comment]: <> (        throw new Exception&#40;"image file is not exists!"&#41;;)

[comment]: <> (    })

[comment]: <> (    FileInputStream fis = new FileInputStream&#40;image_path&#41;;)

[comment]: <> (    Bitmap bitmap = BitmapFactory.decodeStream&#40;fis&#41;;)

[comment]: <> (    int result = predictImage&#40;bitmap&#41;;)

[comment]: <> (    if &#40;bitmap.isRecycled&#40;&#41;&#41; {)

[comment]: <> (        bitmap.recycle&#40;&#41;;)

[comment]: <> (    })

[comment]: <> (    return result;)

[comment]: <> (})

[comment]: <> (public int predictImage&#40;Bitmap bitmap&#41; throws Exception {)

[comment]: <> (    return predict&#40;bitmap&#41;;)

[comment]: <> (})

[comment]: <> (```)

[comment]: <> (这里创建一个获取最大概率值，并把下标返回的方法，其实就是获取概率最大的预测标签。)

[comment]: <> (```java)

[comment]: <> (public static int getMaxResult&#40;float[] result&#41; {)

[comment]: <> (    float probability = 0;)

[comment]: <> (    int r = 0;)

[comment]: <> (    for &#40;int i = 0; i < result.length; i++&#41; {)

[comment]: <> (        if &#40;probability < result[i]&#41; {)

[comment]: <> (            probability = result[i];)

[comment]: <> (            r = i;)

[comment]: <> (        })

[comment]: <> (    })

[comment]: <> (    return r;)

[comment]: <> (})

[comment]: <> (```)

[comment]: <> (这个方法就是MNN执行预测的最后一步，通过执行 `mSession.run&#40;&#41;`对输入的数据进行预测并得到预测结果，通过解析获取到最大的概率的预测标签，并返回。到这里MNN的工具就完成了。)

[comment]: <> (```java)

[comment]: <> (private float[] predict&#40;Bitmap bmp&#41; throws Exception {)

[comment]: <> (    imgData.reset&#40;&#41;;)

[comment]: <> (    imgData.postScale&#40;inputWidth / &#40;float&#41; bmp.getWidth&#40;&#41;, inputHeight / &#40;float&#41; bmp.getHeight&#40;&#41;&#41;;)

[comment]: <> (    imgData.invert&#40;imgData&#41;;)

[comment]: <> (    MNNImageProcess.convertBitmap&#40;bmp, mInputTensor, dataConfig, imgData&#41;;)

[comment]: <> (    try {)

[comment]: <> (        mSession.run&#40;&#41;;)

[comment]: <> (    } catch &#40;Exception e&#41; {)

[comment]: <> (        throw new Exception&#40;"predict image fail! log:" + e&#41;;)

[comment]: <> (    })

[comment]: <> (    MNNNetInstance.Session.Tensor output = mSession.getOutput&#40;null&#41;;)

[comment]: <> (    float[] result = output.getFloatData&#40;&#41;;)

[comment]: <> (    Log.d&#40;TAG, Arrays.toString&#40;result&#41;&#41;;)

[comment]: <> (    int l = getMaxResult&#40;result&#41;;)

[comment]: <> (    return new float[]{l, result[l]};)

[comment]: <> (})

[comment]: <> (```)

[comment]: <> (## 选择图片预测)

[comment]: <> (本教程会有两个页面，一个是选择图片进行预测的页面，另一个是使用相机实时预测并显示预测结果。以下为 `activity_main.xml`的代码，通过按钮选择图片，并在该页面显示图片和预测结果。)

[comment]: <> (```xml)

[comment]: <> (<?xml version="1.0" encoding="utf-8"?>)

[comment]: <> (<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android")

[comment]: <> (    xmlns:app="http://schemas.android.com/apk/res-auto")

[comment]: <> (    xmlns:tools="http://schemas.android.com/tools")

[comment]: <> (    android:layout_width="match_parent")

[comment]: <> (    android:layout_height="match_parent")

[comment]: <> (    android:orientation="vertical")

[comment]: <> (    tools:context=".MainActivity">)

[comment]: <> (    <ImageView)

[comment]: <> (        android:id="@+id/image_view")

[comment]: <> (        android:layout_width="match_parent")

[comment]: <> (        android:layout_height="400dp" />)

[comment]: <> (    <TextView)

[comment]: <> (        android:id="@+id/result_text")

[comment]: <> (        android:layout_width="match_parent")

[comment]: <> (        android:layout_height="wrap_content")

[comment]: <> (        android:layout_below="@id/image_view")

[comment]: <> (        android:text="识别结果")

[comment]: <> (        android:textSize="16sp" />)


[comment]: <> (    <LinearLayout)

[comment]: <> (        android:layout_width="match_parent")

[comment]: <> (        android:layout_height="wrap_content")

[comment]: <> (        android:layout_alignParentBottom="true")

[comment]: <> (        android:orientation="horizontal">)

[comment]: <> (        <Button)

[comment]: <> (            android:id="@+id/select_img_btn")

[comment]: <> (            android:layout_width="0dp")

[comment]: <> (            android:layout_height="wrap_content")

[comment]: <> (            android:layout_weight="1")

[comment]: <> (            android:text="选择照片" />)


[comment]: <> (        <Button)

[comment]: <> (            android:id="@+id/open_camera")

[comment]: <> (            android:layout_width="0dp")

[comment]: <> (            android:layout_height="wrap_content")

[comment]: <> (            android:layout_weight="1")

[comment]: <> (            android:text="实时预测" />)

[comment]: <> (    </LinearLayout>)

[comment]: <> (</RelativeLayout>)

[comment]: <> (```)

[comment]: <> (在 `MainActivity.java`中，进入到页面我们就要先加载模型，我们是把模型放在Android项目的assets目录的，我们需要把模型复制到一个缓存目录，然后再从缓存目录加载模型，同时还有读取标签名，标签名称按照训练的label顺序存放在assets的 `label_list.txt`，以下为实现代码。)

[comment]: <> (```java)

[comment]: <> (classNames = Utils.ReadListFromFile&#40;getAssets&#40;&#41;, "label_list.txt"&#41;;)

[comment]: <> (String classificationModelPath = getCacheDir&#40;&#41;.getAbsolutePath&#40;&#41; + File.separator + "mobilenet_v2.mnn";)

[comment]: <> (Utils.copyFileFromAsset&#40;MainActivity.this, "mobilenet_v2.mnn", classificationModelPath&#41;;)

[comment]: <> (try {)

[comment]: <> (    mnnClassification = new MNNClassification&#40;classificationModelPath&#41;;)

[comment]: <> (    Toast.makeText&#40;MainActivity.this, "模型加载成功！", Toast.LENGTH_SHORT&#41;.show&#40;&#41;;)

[comment]: <> (} catch &#40;Exception e&#41; {)

[comment]: <> (    Toast.makeText&#40;MainActivity.this, "模型加载失败！", Toast.LENGTH_SHORT&#41;.show&#40;&#41;;)

[comment]: <> (    e.printStackTrace&#40;&#41;;)

[comment]: <> (    finish&#40;&#41;;)

[comment]: <> (})

[comment]: <> (```)

[comment]: <> (添加两个按钮点击事件，可以选择打开相册读取图片进行预测，或者打开另一个Activity进行调用摄像头实时识别。)

[comment]: <> (```java)

[comment]: <> (Button selectImgBtn = findViewById&#40;R.id.select_img_btn&#41;;)

[comment]: <> (Button openCamera = findViewById&#40;R.id.open_camera&#41;;)

[comment]: <> (imageView = findViewById&#40;R.id.image_view&#41;;)

[comment]: <> (textView = findViewById&#40;R.id.result_text&#41;;)

[comment]: <> (selectImgBtn.setOnClickListener&#40;new View.OnClickListener&#40;&#41; {)

[comment]: <> (    @Override)

[comment]: <> (    public void onClick&#40;View v&#41; {)

[comment]: <> (        // 打开相册)

[comment]: <> (        Intent intent = new Intent&#40;Intent.ACTION_PICK&#41;;)

[comment]: <> (        intent.setType&#40;"image/*"&#41;;)

[comment]: <> (        startActivityForResult&#40;intent, 1&#41;;)

[comment]: <> (    })

[comment]: <> (}&#41;;)

[comment]: <> (openCamera.setOnClickListener&#40;new View.OnClickListener&#40;&#41; {)

[comment]: <> (    @Override)

[comment]: <> (    public void onClick&#40;View v&#41; {)

[comment]: <> (        // 打开实时拍摄识别页面)

[comment]: <> (        Intent intent = new Intent&#40;MainActivity.this, CameraActivity.class&#41;;)

[comment]: <> (        startActivity&#40;intent&#41;;)

[comment]: <> (    })

[comment]: <> (}&#41;;)

[comment]: <> (```)

[comment]: <> (当打开相册选择照片之后，回到原来的页面，在下面这个回调方法中获取选择图片的Uri，通过Uri可以获取到图片的绝对路径。如果Android8以上的设备获取不到图片，需要在 `AndroidManifest.xml`配置文件中的 `application`添加 `android:requestLegacyExternalStorage="true"`。拿到图片路径之后，调用 `TFLiteClassificationUtil`类中的 `predictImage&#40;&#41;`方法预测并获取预测值，在页面上显示预测的标签、对应标签的名称、概率值和预测时间。)

[comment]: <> (```java)

[comment]: <> (@Override)

[comment]: <> (protected void onActivityResult&#40;int requestCode, int resultCode, @Nullable Intent data&#41; {)

[comment]: <> (    super.onActivityResult&#40;requestCode, resultCode, data&#41;;)

[comment]: <> (    String image_path;)

[comment]: <> (    if &#40;resultCode == Activity.RESULT_OK&#41; {)

[comment]: <> (        if &#40;requestCode == 1&#41; {)

[comment]: <> (            if &#40;data == null&#41; {)

[comment]: <> (                Log.w&#40;"onActivityResult", "user photo data is null"&#41;;)

[comment]: <> (                return;)

[comment]: <> (            })

[comment]: <> (            Uri image_uri = data.getData&#40;&#41;;)

[comment]: <> (            image_path = getPathFromURI&#40;MainActivity.this, image_uri&#41;;)

[comment]: <> (            try {)

[comment]: <> (                // 预测图像)

[comment]: <> (                FileInputStream fis = new FileInputStream&#40;image_path&#41;;)

[comment]: <> (                imageView.setImageBitmap&#40;BitmapFactory.decodeStream&#40;fis&#41;&#41;;)

[comment]: <> (                long start = System.currentTimeMillis&#40;&#41;;)

[comment]: <> (                float[] result = mnnClassification.predictImage&#40;image_path&#41;;)

[comment]: <> (                long end = System.currentTimeMillis&#40;&#41;;)

[comment]: <> (                String show_text = "预测结果标签：" + &#40;int&#41; result[0] +)

[comment]: <> (                        "\n名称：" +  classNames[&#40;int&#41; result[0]] +)

[comment]: <> (                        "\n概率：" + result[1] +)

[comment]: <> (                        "\n时间：" + &#40;end - start&#41; + "ms";)

[comment]: <> (                textView.setText&#40;show_text&#41;;)

[comment]: <> (            } catch &#40;Exception e&#41; {)

[comment]: <> (                e.printStackTrace&#40;&#41;;)

[comment]: <> (            })

[comment]: <> (        })

[comment]: <> (    })

[comment]: <> (})

[comment]: <> (```)

[comment]: <> (上面获取的Uri可以通过下面这个方法把Url转换成绝对路径。)

[comment]: <> (```java)

[comment]: <> (// get photo from Uri)

[comment]: <> (public static String getPathFromURI&#40;Context context, Uri uri&#41; {)

[comment]: <> (    String result;)

[comment]: <> (    Cursor cursor = context.getContentResolver&#40;&#41;.query&#40;uri, null, null, null, null&#41;;)

[comment]: <> (    if &#40;cursor == null&#41; {)

[comment]: <> (        result = uri.getPath&#40;&#41;;)

[comment]: <> (    } else {)

[comment]: <> (        cursor.moveToFirst&#40;&#41;;)

[comment]: <> (        int idx = cursor.getColumnIndex&#40;MediaStore.Images.ImageColumns.DATA&#41;;)

[comment]: <> (        result = cursor.getString&#40;idx&#41;;)

[comment]: <> (        cursor.close&#40;&#41;;)

[comment]: <> (    })

[comment]: <> (    return result;)

[comment]: <> (})

[comment]: <> (```)

[comment]: <> (## 摄像头实时预测)

[comment]: <> (在调用相机实时预测我就不再介绍了，原理都差不多，具体可以查看[https://github.com/yeyupiaoling/ClassificationForAndroid/tree/master/TFLiteClassification]&#40;https://github.com/yeyupiaoling/ClassificationForAndroid/tree/master/TFLiteClassification&#41;中的源代码。核心代码如下，创建一个子线程，子线程中不断从摄像头预览的 `AutoFitTextureView`上获取图像，并执行预测，并在页面上显示预测的标签、对应标签的名称、概率值和预测时间。每一次预测完成之后都立即获取图片继续预测，只要预测速度够快，就可以看成实时预测。)

[comment]: <> (```java)

[comment]: <> (private Runnable periodicClassify =)

[comment]: <> (        new Runnable&#40;&#41; {)

[comment]: <> (            @Override)

[comment]: <> (            public void run&#40;&#41; {)

[comment]: <> (                synchronized &#40;lock&#41; {)

[comment]: <> (                    if &#40;runClassifier&#41; {)

[comment]: <> (                        // 开始预测前要判断相机是否已经准备好)

[comment]: <> (                        if &#40;getApplicationContext&#40;&#41; != null && mCameraDevice != null && mnnClassification != null&#41; {)

[comment]: <> (                            predict&#40;&#41;;)

[comment]: <> (                        })

[comment]: <> (                    })

[comment]: <> (                })

[comment]: <> (                if &#40;mInferThread != null && mInferHandler != null && mCaptureHandler != null && mCaptureThread != null&#41; {)

[comment]: <> (                    mInferHandler.post&#40;periodicClassify&#41;;)

[comment]: <> (                })

[comment]: <> (            })

[comment]: <> (        };)

[comment]: <> (// 预测相机捕获的图像)

[comment]: <> (private void predict&#40;&#41; {)

[comment]: <> (    // 获取相机捕获的图像)

[comment]: <> (    Bitmap bitmap = mTextureView.getBitmap&#40;&#41;;)

[comment]: <> (    try {)

[comment]: <> (        // 预测图像)

[comment]: <> (        long start = System.currentTimeMillis&#40;&#41;;)

[comment]: <> (        float[] result = mnnClassification.predictImage&#40;bitmap&#41;;)

[comment]: <> (        long end = System.currentTimeMillis&#40;&#41;;)

[comment]: <> (        String show_text = "预测结果标签：" + &#40;int&#41; result[0] +)

[comment]: <> (                "\n名称：" +  classNames[&#40;int&#41; result[0]] +)

[comment]: <> (                "\n概率：" + result[1] +)

[comment]: <> (                "\n时间：" + &#40;end - start&#41; + "ms";)

[comment]: <> (        textView.setText&#40;show_text&#41;;)

[comment]: <> (    } catch &#40;Exception e&#41; {)

[comment]: <> (        e.printStackTrace&#40;&#41;;)

[comment]: <> (    })

[comment]: <> (})

[comment]: <> (```)

[comment]: <> (本项目中使用的了读取图片的权限和打开相机的权限，所以不要忘记在 `AndroidManifest.xml`添加以下权限申请。)

[comment]: <> (```bash)

[comment]: <> (<uses-permission android:name="android.permission.CAMERA"/>)

[comment]: <> (<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>)

[comment]: <> (<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>)

[comment]: <> (```)

[comment]: <> (如果是Android 6 以上的设备还要动态申请权限。)

[comment]: <> (```java)

[comment]: <> (    // check had permission)

[comment]: <> (    private boolean hasPermission&#40;&#41; {)

[comment]: <> (        if &#40;Build.VERSION.SDK_INT >= Build.VERSION_CODES.M&#41; {)

[comment]: <> (            return checkSelfPermission&#40;Manifest.permission.CAMERA&#41; == PackageManager.PERMISSION_GRANTED &&)

[comment]: <> (                    checkSelfPermission&#40;Manifest.permission.READ_EXTERNAL_STORAGE&#41; == PackageManager.PERMISSION_GRANTED &&)

[comment]: <> (                    checkSelfPermission&#40;Manifest.permission.WRITE_EXTERNAL_STORAGE&#41; == PackageManager.PERMISSION_GRANTED;)

[comment]: <> (        } else {)

[comment]: <> (            return true;)

[comment]: <> (        })

[comment]: <> (    })

[comment]: <> (    // request permission)

[comment]: <> (    private void requestPermission&#40;&#41; {)

[comment]: <> (        if &#40;Build.VERSION.SDK_INT >= Build.VERSION_CODES.M&#41; {)

[comment]: <> (            requestPermissions&#40;new String[]{Manifest.permission.CAMERA,)

[comment]: <> (                    Manifest.permission.READ_EXTERNAL_STORAGE,)

[comment]: <> (                    Manifest.permission.WRITE_EXTERNAL_STORAGE}, 1&#41;;)

[comment]: <> (        })

[comment]: <> (    })

[comment]: <> (```)

[comment]: <> (*/)
**效果图：**
![在这里插入图片描述](https://s1.ax1x.com/2020/09/05/wVLG6g.jpg)
