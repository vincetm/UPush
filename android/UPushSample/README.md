## U-Push SDK Sample

友盟推送SDK集成的示例工程说明

### 账号申请
在[U-Push官网](http://message.umeng.com/)添加应用申请Appkey。  
详细路径：U-Push官网->应用->新建应用->创建新应用。

### 集成说明
1. app/build.gradle中替换您的应用id
```groovy
android {
    defaultConfig {
        applicationId "您的应用id"
    }
}
```

2. PushConstants类中替换Appkey、MessageSecret和Channel等
```java
class PushConstants {
    /**
     * 应用申请的Appkey
     */
    public static final String APP_KEY = "应用申请的Appkey";

    /**
     * 应用申请的UmengMessageSecret
     */
    public static final String MESSAGE_SECRET = "应用申请的UmengMessageSecret";

    /**
     * 设置您打包时的渠道名称
     */
    public static final String CHANNEL = "Umeng";
}
```

3. PushHelper类初步封装PushSDK的初始化
```java
class PushHelper {
    /**
     * SDK预初始化
     * @param context 应用上下文
     */
    public static void preInit(Context context) {
        ...
    }
    
    /**
     * SDK初始化。
     * 合规说明：需在用户已确认同意"隐私政策协议"后调用
     * @param context 应用上下文
     */
    public static void init(Context context) {
        ...
    }

    /**
     * 注册设备推送通道（小米、华为等设备的推送）
     * @param context 应用上下文
     */
    private static void registerDeviceChannel(Context context) {
        ...
    }
}
```

4. MyApplication类onCreate()方法中调用预初始化或初始化
```java
public class MyApplication extends Application {

    @Override
    public void onCreate() {
        super.onCreate();
        initUmengSDK();
    }

    /**
     * 初始化友盟SDK
     */
    private void initUmengSDK() {
        //日志开关
        UMConfigure.setLogEnabled(true);
        //预初始化
        PushHelper.preInit(this);
        //是否同意隐私政策
        boolean agreed = MyPreferences.getInstance(this).hasAgreePrivacyAgreement();
        if (!agreed) {
            return;
        }
        boolean isMainProcess = UMUtils.isMainProgress(this);
        if (isMainProcess) {
            //启动优化：建议在子线程中执行初始化
            new Thread(new Runnable() {
                @Override
                public void run() {
                    PushHelper.init(getApplicationContext());
                }
            }).start();
        } else {
            //若不是主进程（":channel"结尾的进程），直接初始化sdk，不可在子线程中执行
            PushHelper.init(getApplicationContext());
        }
    }

}
```

5. MainActivity类中，用户同意隐私政策协议后，执行初始化
```java
public class MainActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        if (hasAgreedAgreement()) {
            PushAgent.getInstance(this).onAppStart();
        } else {
            showAgreementDialog();
        }
    }

    private boolean hasAgreedAgreement() {
        return MyPreferences.getInstance(this).hasAgreePrivacyAgreement();
    }

    private void showAgreementDialog() {
        AlertDialog.Builder builder = new AlertDialog.Builder(this);
        builder.setTitle(R.string.agreement_title);
        builder.setMessage(R.string.agreement_msg);
        builder.setPositiveButton(R.string.agreement_ok, new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialog, int which) {
                dialog.dismiss();
                //用户点击隐私协议同意按钮后，初始化PushSDK
                MyPreferences.getInstance(getApplicationContext()).setAgreePrivacyAgreement(true);
                PushHelper.init(getApplicationContext());
                ...
            }
        });
        builder.setNegativeButton(R.string.agreement_cancel, new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialog, int which) {
                dialog.dismiss();
                ...
                finish();
            }
        });
        builder.create().show();
    }
}
```
6. push包依赖情况如下
```
+--- com.umeng.umsdk:push:6.5.1
|    +--- com.umeng.umsdk:alicloud-httpdns:1.3.2.3.2
|    +--- com.umeng.umsdk:alicloud-utils:2.0.0
|    +--- com.umeng.umsdk:alicloud_beacon:1.0.5
|    +--- com.umeng.umsdk:agoo-accs:3.4.2.7.1
|    +--- com.umeng.umsdk:agoo_aranger:1.0.6
|    +--- com.umeng.umsdk:agoo_networksdk:3.5.8.4
|    +--- com.umeng.umsdk:agoo_tnet4android:3.1.14.10.2
|    \--- com.umeng.umsdk:utdid:1.5.2.3
```
移除重复组件的方法，如utdid：
```groovy
    api('com.umeng.umsdk:push:6.5.1') {
        exclude group: 'com.umeng.umsdk', module: 'utdid'
    }
```

**具体使用，请参考UPushSample工程**

