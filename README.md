## 一：简介

快捷支付涉及到方方面面，同时安全问题既是用户所关心的，也是制作者不容忽视的重要部分。
比如涉及到支付和金钱的app，商户端app等等，都需要进行实名认证，实名认证可以通过银行卡认证和身份证认证。
通常办法是通过上传照片，并且手动输入基本信息进行认证，这种方法不仅操作复杂，而且容易出错。
网上很多资源，识别率低，速度慢，用户体验很差。今天我就和大家介绍一下我使用的方法，可以快速、高效的识别中国身份证信息。

## 二：原理

在拍摄框中右上区域加了一个人像区域提示框，并将该区域设为扫描人脸的区域，只有该区域扫描到身份证上的人脸时（确保用户的确将身份证人像对准了拍摄框中的人像框），才执行读取身份证信息的操作。

## 三：项目展示

扫描身份证面截图

![头像面扫描](http://upload-images.jianshu.io/upload_images/2829694-20b78f45be90b3de.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![国徽面扫描](http://upload-images.jianshu.io/upload_images/2829694-8cd81ffdb6f4daac.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

信息展示页面

![头像页信息](http://upload-images.jianshu.io/upload_images/2829694-5dc5685f6c54a9df.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![国徽页信息](http://upload-images.jianshu.io/upload_images/2829694-856239c3727e49d6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 四： 使用流程介绍

#### 4.1 身份验证

由于使用的是百度云的图像文字识别sdk，首先需要去百度云[[管理控制台](https://console.bce.baidu.com/ai/?fromai=1&_=1488766023093#/ai/ocr/app/list)
](https://console.bce.baidu.com/ai/?_=1515636851708&fromai=1#/ai/ocr/app/list)获得API Key / Secret Key。

![管理控制台](http://upload-images.jianshu.io/upload_images/2829694-d962a45ac612e492.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![管理控制台](http://upload-images.jianshu.io/upload_images/2829694-97842b3398dd0700.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 4.2 下载demo

1. 将下载的demo中libs目录下的ocr-sdk.jar文件拷贝到工程libs目录中，并加入工程依赖 
2. 将libs目录下armeabi，arm64-v8a，armeabi-v7a，x86文件夹按需添加到android studio工程src/main/jniLibs目录中， eclipse用户默认为libs目录。
3. 在Android studio中以模块方式导入下载包中的identify文件夹,模块方式导入流程如下：
首先要在顶层工程目录下的settings.gradle文件中include模块名
```
include ':app',':identify'
```
然后直接复制粘贴identify文件导入到项目的文件夹中！！！然后Build>Rebuild Project，重新构建项目。

#### 4.3 权限配置

1.  在工程AndroidManifest.xml文件中添加如下权限：
```
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.CAMERA"/>
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
```
2. 在Proguard配置文件中增加, 防止release发布时打包报错：
```
-keep class com.baidu.ocr.sdk.**{*;}
-dontwarn com.baidu.ocr.**
```
#### 4.4 核心函数介绍

1. 初始化函数
```
OCR.getInstance().initAccessTokenWithAkSk(new OnResultListener<AccessToken>() {
            @Override
            public void onResult(AccessToken result) {

            }

            @Override
            public void onError(OCRError error) {
                error.printStackTrace();
                Log.d("onError", "msg: " + error.getMessage());
            }
        }, getApplicationContext(), "你注册的appkey", "你注册的sk");
``` 
2. 身份证拍照

```
Intent intent = new Intent(MainActivity.this, CameraActivity.class);
intent.putExtra(CameraActivity.KEY_OUTPUT_FILE_PATH,
    FileUtil.getSaveFile(getApplication()).getAbsolutePath());
intent.putExtra(CameraActivity.KEY_CONTENT_TYPE, CameraActivity.CONTENT_TYPE_ID_CARD_FRONT);
startActivityForResult(intent, REQUEST_CODE_CAMERA);
```
3. 图像回调
```
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    super.onActivityResult(requestCode, resultCode, data);

    if (requestCode == REQUEST_CODE_CAMERA && resultCode == Activity.RESULT_OK) {
        if (data != null) {
            String contentType = data.getStringExtra(CameraActivity.KEY_CONTENT_TYPE);
            String filePath = FileUtil.getSaveFile(getApplicationContext()).getAbsolutePath();
            if (!TextUtils.isEmpty(contentType)) {
                if (CameraActivity.CONTENT_TYPE_ID_CARD_FRONT.equals(contentType)) {
                    recIDCard(IDCardParams.ID_CARD_SIDE_FRONT, filePath);
                } else if (CameraActivity.CONTENT_TYPE_ID_CARD_BACK.equals(contentType)) {
                    recIDCard(IDCardParams.ID_CARD_SIDE_BACK, filePath);
                }
            }
        }
    }
}
```
4. 数据解析
```
private void recIDCard(String idCardSide, String filePath) {
    IDCardParams param = new IDCardParams();
    param.setImageFile(new File(filePath));
    param.setIdCardSide(idCardSide);
    param.setDetectDirection(true);
    OCR.getInstance().recognizeIDCard(param, new OnResultListener<IDCardResult>() {
        @Override
        public void onResult(IDCardResult result) {
            if (result != null) {
                Log.d("onResult", "result: " + result.toString());
            }
        }

        @Override
        public void onError(OCRError error) {
            Log.d("onError", "error: " + error.getMessage());
        }
    });
}
```
到此身份证识别接入就结束了！！！

相同方式均可实现银行卡识别、驾驶证识别、行驶证识别、车牌识别、营业执照识别等等，如有需要demo可联系我。

> 希望可以帮助大家，可加微信：FBY-fan 拉你进群交流

> 如果哪里有什么不对或者不足的地方，还望读者多多提意见或建议

> 如需转载请联系我，经过授权方可转载，谢谢

***
欢迎关注公众号「网罗开发」

<img width="500" alt="网罗开发" src="https://user-images.githubusercontent.com/24238160/131977235-0938b244-820d-472d-a708-5b4a3ea39f6e.png">
