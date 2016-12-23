# PushSDK3 Eclipse 快速开始

# 目录<a name="index"/>

* [一.准备工作](#prepare_setting)
    * [1.1 PushSDK资源引用配置说明](#pushsdk_res_setting)
        * [1.1.1 Lib引用配置说明](#lib_setting)
        * [1.1.2 layout引用配置说明](#layout_setting)
        * [1.1.3 proguard配置说明](#proguard_setting)
    * [1.2 必要的配置](#nessary_setting)
        * [1.2.1 权限配置](#permission_setting)
        * [1.2.2 必要组件配置](#nessary_component_setting)
        * [1.2.3 注册消息接收Receiver](#pushmessage_receiver_manifest_setting)
        * [1.2.4 实现自有的PushReceiver,实现消息接收，注册与反注册回调](#pushmessage_receiver_code_setting)    
* [二.调用新版注册](#start_register)


# 一.准备工作<a name="prepare_setting"/>

**NOTE:** 请按照项目需求选取用的引用方式引用aar包

## 1.1 pushSDK内部版引用配置说明<a name="pushsdk_internal"/>

### 1.1.1 Lib引用配置说明<a name="lib_setting">
  PushSDK3.0精简了依赖关系，现在只需要一个jar包，无需引入其他第三方类库
  **NOTE:** 请将PushSDK的jar，拷贝到工程的lib目录即可，jar包具体见PushDemo的lib目录
  
### 1.1.2 layout引用配置说明<a name="layout_setting">
  PushSDK需要依赖三个layout文件，需要工程手动配置
  将sdk的res目录下的三个layout文件，拷贝工程的```res/layout```目录下
  
### 1.1.3 proguard配置说明<a name="proguard_setting">        
  由于jar不能打包混淆规则，你需要手动配置pushSDK的混淆规则，具体如下:
  
```
    -dontwarn com.meizu.cloud.pushsdk.**
    -keep class com.meizu.cloud.pushsdk.**{*;}
```

## 1.2 必要的配置<a name="nessary_setting"/>

### 1.2.1 兼容flyme5以下版本推送兼容配置<a name="permission_setting"/>
    **NOTE:** jar无法打包pushSDK的所需的权限，你需要收到配置一下权限
```
    <!--PushSDK运行必须配置权限-->
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.READ_PHONE_STATE" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />
    <uses-permission android:name="android.permission.WRITE_SETTINGS" />
    <uses-permission android:name="android.permission.VIBRATE" />
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.ACCESS_DOWNLOAD_MANAGER"/>
    <uses-permission android:name="android.permission.DOWNLOAD_WITHOUT_NOTIFICATION" />
    <uses-permission android:name="android.permission.DISABLE_KEYGUARD" />
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
```

    **NOTE：** 如果你需要支持flyme5以下版本推送，以下内容可以选配
```
    <!-- 兼容flyme5.0以下版本，魅族内部集成pushSDK必填，不然无法收到消息-->
    <uses-permission android:name="com.meizu.flyme.push.permission.RECEIVE"></uses-permission>
    <permission android:name="包名.push.permission.MESSAGE" android:protectionLevel="signature"/>
    <uses-permission android:name="包名.push.permission.MESSAGE"></uses-permission>
    
    <!--  兼容flyme3.0配置权限-->
    <uses-permission android:name="com.meizu.c2dm.permission.RECEIVE" />
    <permission android:name="你的包名.permission.C2D_MESSAGE"
                    android:protectionLevel="signature"></permission>
    <uses-permission android:name="你的包名.permission.C2D_MESSAGE"/>

```

#### 1.2.2 必要组件配置<a name="nessary_component_setting">

```
        <!-- 必要配置,提高push消息送达率 -->
        <service
            android:name="com.meizu.cloud.pushsdk.NotificationService"
            android:exported="true"/>

        <!-- 必要配置，pushSDK内部组件 -->
        <receiver android:name="com.meizu.cloud.pushsdk.SystemReceiver" >
            <intent-filter>
                <action android:name="com.meizu.cloud.pushservice.action.PUSH_SERVICE_START"/>
                <category android:name="android.intent.category.DEFAULT" />
            </intent-filter>
        </receiver>
```

#### 1.2.2 注册消息接收Receiver<a name="pushmessage_receiver_manifest_setting"/>

```xml
    <!-- push应用定义消息receiver声明 -->
    <receiver android:name="your.package.MyPushMsgReceiver">
        <intent-filter>
            <!-- 接收push消息 -->
            <action android:name="com.meizu.flyme.push.intent.MESSAGE" />
            <!-- 接收register消息 -->
            <action android:name="com.meizu.flyme.push.intent.REGISTER.FEEDBACK" />
            <!-- 接收unregister消息-->
            <action android:name="com.meizu.flyme.push.intent.UNREGISTER.FEEDBACK"/>
            <!-- 兼容低版本Flyme3推送服务配置 -->
            <action android:name="com.meizu.c2dm.intent.REGISTRATION" />
            <action android:name="com.meizu.c2dm.intent.RECEIVE" />
            <category android:name="包名"></category>
        </intent-filter>
    </receiver>
```
#### 1.2.3 实现自有的PushReceiver,实现消息接收，注册与反注册回调<a name="pushmessage_receiver_code_setting"/>

```
	public class MyPushMsgReceiver extends MzPushMessageReceiver {

	    @Override
	    public void onRegister(Context context, String pushid) {
		       //应用在接受返回的pushid
	    }

	    @Override
	    public void onMessage(Context context, String s) {
		      //接收服务器推送的消息
	    }

	    @Override
	    public void onUnRegister(Context context, boolean b) {
		      //调用PushManager.unRegister(context）方法后，会在此回调反注册状态
	    }

	    //设置通知栏小图标
	    @Override
	    public PushNotificationBuilder onUpdateNotificationBuilder(PushNotificationBuilder pushNotificationBuilder) {
		      pushNotificationBuilder.setmStatusbarIcon(R.drawable.mz_stat_share_weibo);
	    }

	    @Override
	    public void onPushStatus(Context context,PushSwitchStatus pushSwitchStatus) {
		      //检查通知栏和透传消息开关状态回调
	    }

	    @Override
	    public void onRegisterStatus(Context context,RegisterStatus registerStatus) {
		     Log.i(TAG, "onRegisterStatus " + registerStatus);
         //新版订阅回调
	    }

	    @Override
	    public void onUnRegisterStatus(Context context,UnRegisterStatus unRegisterStatus) {
		     Log.i(TAG,"onUnRegisterStatus "+unRegisterStatus);
           //新版反订阅回调
	    }

	    @Override
	    public void onSubTagsStatus(Context context,SubTagsStatus subTagsStatus) {
		     Log.i(TAG, "onSubTagsStatus " + subTagsStatus);
		     //标签回调
	    }

	    @Override
	    public void onSubAliasStatus(Context context,SubAliasStatus subAliasStatus) {
		     Log.i(TAG, "onSubAliasStatus " + subAliasStatus);
         //别名回调
	    }
	}
```


# 二. 调用新版注册
**Note:** 至此pushSDK 已经集成完毕，现在你需要在你的Application中调用新版的[register](#register)方法,
```
   /**
     * @param context
     * @param appId
     *         push 平台申请的应用id
     * @param appKey
     *         push 平台申请的应用key
     * */
     public static void register(Context context,String appId,String appKey);
```

并在你的Receiver中成功回调onRegisterStatus(RegisterStatus registerStatus)方法就可以了，
你现在可以到[新版Push平台](http://push.meizu.com) 找到你的应用推送消息就可以了;以下内容是pushSDK提供的api汇总,具体功能详见api具体说明,请根据需求选用合适的功能





