# Android 版在线打分 sdk 介绍

在使用在线打分服务前请先申请获取 appId 和 appSecret, 其中 appId appSecret 是客户端的身份标识，在客户端SDK初始化时使用。android sdk 以 aar gradle 方式提供，demo 在相关网页上提供下载。

Android 版在线打分 sdk 支持对 readLoud 题型进行打分，录音过程中 sdk 会与服务器进行通讯，当录音结束后服务端处理完毕，则会提供打分报告，并且sdk会在本地生成当前录音文件(wav格式)


# sdk 集成

## gradle 添加依赖

```
// release aar deploy on jcenter
compile('com.liulishuo.engzo:online-scorer:1.1.1@aar') {
    transitive = true
}

```

## api level 支持

min api level 15

## 权限要求

### Normal Permissions
```
<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/>
```
### Dangerous Permissions
```
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
<uses-permission android:name="android.permission.RECORD_AUDIO" />

```

当 targetSdkVersion 为 23 的时候需要开发者手动处理 runtime permission，否则会造成异常



## Supported ABIs

sdk 默认打包所有现在支持的 abi, 倘若需要去除掉不用的可以采用如下方法

```

defaultConfig {
    applicationId "com.liulishuo.engzo.scorerdemo"
    minSdkVersion 15
    targetSdkVersion 25
    versionCode 1
    versionName "1.0"

    ndk {
        abiFilters "armeabi-v7a", "x86"
    }
}
```

# 基本使用

## 初始化
请先在 Application 里进行初始化操作：
```
public class DemoApp extends Application {

    @Override
    public void onCreate() {
        super.onCreate();
        OnlineScorer.init(this, "test", "test");
        OnlineScorer.setDebugEnable(true);
    }

}
```

## 使用示例
```


// 创建 "i will study english very hard" 这句话的练习
ReadLoudExercise readLoudExercise = new ReadLoudExercise();
readLoudExercise.setReftext("i will study english very hard");
// 并且指定音频质量为8，该参数只影响传输给服务端的音频质量，不影响本地录音文件
readLoudExercise.setQuality(8);  

// 创建打分录音器
final OnlineScorerRecorder onlineScorerRecorder = new OnlineScorerRecorder(readLoudExercise, "/sdcard/test.wav");

// OnRecordListener 与 OnProcessStopListener 开始录音后保证成对出现，且 OnProcessStopListener 的回调在 OnRecordListener 的回调之后。
// 录音完成的回调，开发者可以用这个监听录音的完成，在里头一般做更新录音按钮状态的操作
onlineScorerRecorder.setOnRecordStopListener(new OnlineScorerRecorder.OnRecordListener() {
    @Override
    public void onRecordStop(Throwable error, OnlineScorerRecorder.Result result) {
        //result里面可以获取录音时长
    }
});    

// 录音处理完成的回调，开发者可以监听该回调，获取打分报告与录音文件
onlineScorerRecorder.setOnProcessStopListener(new OnlineScorerRecorder.OnProcessStopListener() {
    @Override
    public void onProcessStop(Throwable error, String filePath, String report) {
        
    }
});  


//开始录音打分
onlineScorerRecorder.startRecord();
//结束录音打分
onlineScorerRecorder.stopRecord();

```

### 开启调试
开启调试只需要调用 `OnlineScorer.setDebugEnable(true);` 即可，建议在 Application 中开启，类似 Demo 中的做法。

### 查看日志
开启调试后，可以在控制台过滤 `OnlineScorerRecorder` ,也可以在 OnProcessStopListener 接口返回时获取日志文件的目录:
```
 OnlineScorer.requestLogDir(
                         new RequestLogCallback() {
                             @Override
                             public void onDirResponse(@Nullable File logDir) {
                                 
                             }
                         });
                             
```
# 异常处理

## onRecordStop

 可能会出现录音相关的异常，比如录音初始化错误，具体可查看权限是否获取


## onProcessStop

主要包含 ScorerException 与其他异常，其中 ScorerException 是对 Server 返回 errorMessage 的封装，一般来说开发者只需要关心 ScorerException，其中 status code > 0 属于 client 端的异常，status code < 0 属于 server 端的异常。
当出现 ScorerException 异常的时候可以将当前录音的 filePath 保存下来，供而后重试打分，具体可参考 demo 中的做法。

```

/**
 * 2 - 客户端 网络数据传输错误
 * 1 - 客户端 response timeout
 * 0 - 成功
 * -1 - 参数有误
 * -20 - 认证失败
 * -30 - 请求过于频繁
 * -31 - 余额不足
 * -41 - 排队超时
 * -99 - 计算资源不可用
 */
public static class ScorerException extends Exception {
    private int status;
    private String msg;

    public ScorerException(int status, String msg) {
        super(String.format("response error status = %d msg = %s", status, msg));
        this.status = status;
        this.msg = msg;
    }

    public int getStatus() {
        return status;
    }

    public String getMsg() {
        return msg;
    }
}
```

# 打分报告格式

```

{

    "fluency": 99,

    "integrity": 100,

    "locale": "en",

    "overall": 100,

    "pronunciation": 100,

    "version": "2.1.0",

    "words": [

        {

            "scores": {

                "pronunciation": 100

            },

            "word": "i"

        },

        {

            "scores": {

                "pronunciation": 100

            },

            "word": "will"

        },

        {

            "scores": {

                "pronunciation": 100

            },

            "word": "study"

        },

        {

            "scores": {

                "pronunciation": 100

            },

            "word": "english"

        },

        {

            "scores": {

                "pronunciation": 100

            },

            "word": "very"

        },

        {

            "scores": {

                "pronunciation": 100

            },

            "word": "hard"

        }

    ]

}

```


## ChangeLog

### 1.1.2

1. 修复 SpeexEncoder 调用错误
2. 修复了 start 中未初始化话 CountDownLatch 导致而后打分失败的问题
3. ReadLoudExercise 添加 targetAudience 字段
4. OnlineScorer.init 支持对 url 的注入



