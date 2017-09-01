# ChangeAppIcon
Android实现动态修改应用桌面图标和名称

---------------------------------
*等下要在下面插入一些图片
 ![image](https://github.com/Vension/ChangeAppIcon/blob/master/ScreenShots/Screenshot_1504233093.png?raw=true)
 ![image](https://github.com/Vension/ChangeAppIcon/blob/master/ScreenShots/device-2017-09-01-104923.png?raw=true)
 ![image](https://github.com/Vension/ChangeAppIcon/blob/master/ScreenShots/device-2017-09-01-104938.png?raw=true)
---------------------------------------
我们知道，我们每写一个 Activity就要在AndroidManifest进行配置一下，我们才可以正常的启动它，除此之外，我们还可以对它设置一个别名，也就是用<activity-alias>标签，这个标签的属性，和<activity>的属性一致，可以做一个简单的分析：

android:icon="@mipmap/app_logo"
android:label="@string/app_name"
上面的两个属性是用来设置图标和标签。

android:name=".newsLuncherActivity"
虽然说别名的name可以任意去写，但我想说的是，还是尽量设置成一个activity,尽量和一个<activity>的name保持一致，如果不设置成一个activity的名字，我发现部分手机会有问题，比我现在我手上的测试机。

android:enabled="false"
这个是否是显示别名，默认是true。

android:targetActivity=".MainActivity"
这个就比较重要了，指定别名启动的activity,一定要与原来启动入口activity的name保持一致，并且要在<activity>的标签下面。
具体实现如下：

<application
 android:allowBackup="true"
 android:icon="@mipmap/ic_launcher"
 android:label="@string/app_name"
 android:supportsRtl="true"
 android:theme="@style/AppTheme">
 <activity android:name=".MainActivity">
 <intent-filter>
  <action android:name="android.intent.action.MAIN" />
  <category android:name="android.intent.category.LAUNCHER" />
 </intent-filter>
 </activity>
</application>
上面呢是默认的图标，及默认的activity入口 。

<application
 android:allowBackup="true"
 android:icon="@mipmap/ic_launcher"
 android:label="@string/app_name"
 android:supportsRtl="true"
 android:theme="@style/AppTheme">
 <activity android:name=".MainActivity">
 <intent-filter>
  <action android:name="android.intent.action.MAIN" />
  <category android:name="android.intent.category.LAUNCHER" />
 </intent-filter>
 </activity>
 <activity-alias
 android:name=".newsLuncherActivity"
 android:enabled="false"
 android:icon="@mipmap/app_logo"
 android:label="@string/app_name"
 android:targetActivity=".MainActivity">
 <intent-filter>
  <action android:name="android.intent.action.MAIN" />
  <category android:name="android.intent.category.LAUNCHER" />
 </intent-filter>
 </activity-alias>
</application>
上面是添加<activity-alias>标签后，具体启动方式，我们可以这样做一个控制，服务器端设置一个开关，当请求到要更改桌面图标时，我们就可以通过 PackageManager 对象提供的 setComponentEnabledSetting()方法关闭当前 Component 组件，并启动别名对应的 Component 组件即可，为了使得图标能够快速更换，我们可以加上重启Luncher应用代码，name是自己定义个类名，记住一定要传全路径，如：

com.ming.abner.changelauncher.newsLuncherActivity
 private void changeLuncher(String name) {
 PackageManager pm = getPackageManager();
 pm.setComponentEnabledSetting(getComponentName(),
  PackageManager.COMPONENT_ENABLED_STATE_DISABLED, PackageManager.DONT_KILL_APP);
 pm.setComponentEnabledSetting(new ComponentName(MainActivity.this, name),
  PackageManager.COMPONENT_ENABLED_STATE_ENABLED, PackageManager.DONT_KILL_APP);
 //Intent 重启 Launcher 应用
 Intent intent = new Intent(Intent.ACTION_MAIN);
 intent.addCategory(Intent.CATEGORY_HOME);
 intent.addCategory(Intent.CATEGORY_DEFAULT);
 List<ResolveInfo> resolves = pm.queryIntentActivities(intent, 0);
 for (ResolveInfo res : resolves) {
  if (res.activityInfo != null) {
  ActivityManager am = (ActivityManager) getSystemService(ACTIVITY_SERVICE);
  am.killBackgroundProcesses(res.activityInfo.packageName);
  }
 }
 }
}
<activity-alias>我们可以定义多个，对于不同时候，我们就可以动态去更换不同的图标。
记得添加权限：
<uses-permission android:name="android.permission.KILL_BACKGROUND_PROCESSES"
