## 为什么要做进程保活

对于很多应用来说推送是非常重要，而ios能使用系统方式非常好的实现，但Android系统Push GCM不能在中国使用，这也就导致push优化畸形发展，为了能及时收到推送很多开发者费劲心机让自己的应用一直在后台与服务端保持长连接，尽管有些一天也就2条Push，同时还不让用户关闭,这种方式对用户来说非常不好，导致手机运行越来越慢同时耗电，非常不提倡。值得高兴的是google官方和自定义系统已经在解决这些问题，所以这种做法也将慢慢走向终点。下面是一些并不很友好的解决思路供大家学习，希望不要去使用。


## linux进程守护

早期大部分应用都在linux开启一个进程实现应用杀不死，因为Android系统漏洞在杀死主进程时不会把子进程杀死，不过幸好在5.0以上这种方式已经失效了，Android官方也是已经知道了这个漏洞，5.0之上自动会把当前uid下的所有进程都杀死，具体代码可以查看`ActivityManagerService`

![](https://raw.githubusercontent.com/goodbranch/AndroidNote/master/note/push/killProcess-1.png)

![](https://raw.githubusercontent.com/goodbranch/AndroidNote/master/note/push/killProcess-2.png)

[参考linux实现](https://github.com/droidwolf/andmon) linux守护介绍在网上已经有很多能实现了，可以自行google。

## App之间互相唤醒

通常App之间会通过直接启动Service的方式启动别的，例如反编译优酷你可以看到下面这段代码：

![优酷唤醒](https://raw.githubusercontent.com/goodbranch/AndroidNote/master/note/push/youku-wakeuputil.png)

这种直接指定包名的方式完全没有漏洞导致我们也不能利用起来。但是看到这里你或许会想到如果通过action方式启动可以被利用，在百度`AndroidManifest.xml`有这么一段配置，通过查看源码发现他就是利用过滤这两个action方式启动自家的应用。

	 <intent-filter>
	        <action android:name="com.baidu.android.pushservice.action.METHOD"/>
	        <action android:name="com.baidu.android.pushservice.action.BIND_SYNC"/>
	 </intent-filter>

这是百度地图中广播接收器过滤的一个action，然后我们看代码中`CommandService`他是怎么用的，

![百度反射广播-1](https://raw.githubusercontent.com/goodbranch/AndroidNote/master/note/push/baiduditu-action-1.png)

通过反射把广播发给其他应用，不过包名一定要是**com.baidu**开头的才行，这里我也始终没明白是怎么做到的。我试了下好像是不能反射其他应用的类吧，一脸懵逼。但是奇怪的是就按照百度地图这么写，百度地图或爱奇艺启动时我就能收到广播，然后再启动我的PushService.

	 <receiver android:enabled="@bool/enablePushService" android:name="com.baidu.android.pushservice.RegistrationReceiver" android:process=":bdservice_v1">
	            <intent-filter>
	                <action android:name="com.baidu.android.pushservice.action.METHOD"/>
	                <action android:name="com.baidu.android.pushservice.action.BIND_SYNC"/>
	            </intent-filter>
	            <intent-filter>
	                <action android:name="android.intent.action.PACKAGE_REMOVED"/>
	                <data android:scheme="package"/>
	            </intent-filter>
	        </receiver>


这种方式目前除了在一加手机6.0上发现不行，其他ok的。

## 系统账号同步

在设置中的账号看到下面的场景，但是你点进去其实没有任何操作，其实这些做法就是为了唤醒应用。

![Android 账号系统](https://raw.githubusercontent.com/goodbranch/AndroidNote/master/note/push/system_account_1.png)


[google账号同步demo](https://github.com/googlesamples/android-BasicSyncAdapter/),目前账号同步方式唤醒应用是最有效的，但是6.0后华为，小米上同步被关闭，具体代码实现看官方提供的demo。

## 实现前台进程

进程如果在后台，那么系统就会在某些时机清理你。可以看下官方关于进程生命周期的定义：

![进程](https://raw.githubusercontent.com/goodbranch/AndroidNote/master/note/push/process_1.png)
![进程](https://raw.githubusercontent.com/goodbranch/AndroidNote/master/note/push/process_2.png)

所以如果想要我们的进程不被系统杀死，则应该提高自己进程的优先级，只有高优先级系统才不会再内存不足或者后台省电保护中杀死你，如音乐播放器在通知栏的显示，你在后台不管听多久都能正常运行，这就是一个最直观的供学习案例，下面是三种参考做法。

* Android 4.3 以下像QQ一样使用空的Notification实现前台进程
 
定义Service在另外的进程中`android:process=":push"`：

	    <service
	        android:name=".view.push.wakeup.DaemonService"
	        android:enabled="true"
	        android:exported="true"
	        android:label="PushService"
	        android:process=":push">

然后在 `onStartCommand`使这个Service成为前台进程。

	startForeground(NOTIFICATION_ID, new Notification());

Android早期的版本这种方式可以很好保证进程不被杀死。

* Activity模拟前端进程，4.3以上空Notification方式被修复了，这样设置已经没有用了，那么定义一个一像素大小全透明的Activity，在屏幕关闭后启动，屏幕打开是关闭，同时把他从最近进程列表中移除。

	 public class DaemonActivity extends FragmentActivity {

	  public static final String CLOSE_ACTION = "close";

	  private static Intent newIntent(Context context) {
	    Intent intent = new Intent(context, DaemonActivity.class);
	    intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
	    return intent;
	  }


	  public static void startActivity(Context context) {

	    if (!mStart) {
	      context.startActivity(newIntent(context));
	    }
	  }


	  public static boolean mStart = false;

	  @Override
	  protected void onCreate(Bundle savedInstanceState) {
	    super.onCreate(savedInstanceState);
	    EventBus.getDefault().register(this);

	    mStart = true;

	    View view = new View(getApplicationContext());
	    view.setLayoutParams(new ViewGroup.LayoutParams(1, 1));

	    setContentView(view);

	    view.setOnTouchListener(new View.OnTouchListener() {
	      @Override
	      public boolean onTouch(View v, MotionEvent event) {
	        finish();
	        return true;
	      }
	    });

	    setFinish();
	  }

设置不会在最近列表中和历史记录中显示：

	  <activity
	        android:name=".view.push.weakup.DaemonActivity"
	        android:alwaysRetainTaskState="true"
	        android:autoRemoveFromRecents="true"
	        android:excludeFromRecents="true"
	        android:icon="@drawable/translusent_bg"
	        android:label=" "
	        android:launchMode="singleInstance"
	        android:noHistory="true"
	        android:process=":push"
	        android:showOnLockScreen="true"
	        android:taskAffinity="android.task.push"
	        android:theme="@style/DaemonActivity"/>

设置他特有的样式

	 <style name="DaemonActivity" parent="@style/Theme.AppCompat.Dialog.Alert">

	    <item name="android:windowIsTranslucent">true</item>
	    <item name="android:windowBackground">@android:color/transparent</item>
	    <item name="android:colorBackgroundCacheHint">@null</item>
	    <item name="android:windowNoTitle">true</item>
	    <item name="android:backgroundDimEnabled">false</item>
	    <item name="android:windowAnimationStyle">@android:style/Animation.Translucent</item>
	    <item name="android:windowContentOverlay">@null</item>
	  </style>




然后在屏幕开关时控制他

* 注册一个监听屏幕开关的广播接收器

		  private void registerDaemonBroadcast(int type) {
		    if (mReceiver != null) {
		      return;
		    }
		    IntentFilter intentFilter = new IntentFilter();
		    intentFilter.addAction(Intent.ACTION_SCREEN_OFF);
		    intentFilter.addAction(Intent.ACTION_SCREEN_ON);
		    intentFilter.addAction(Intent.ACTION_USER_PRESENT);
		    mReceiver = new DaemonBroadcastReceiver(type);
		    registerReceiver(mReceiver, intentFilter);
		  }

* 在onReceive中处理

		  @Override
		  public void onReceive(Context context, Intent intent) {

		    String action = intent == null ? "" : intent.getAction();

		    switch (action) {
		      case Intent.ACTION_SCREEN_ON:
		      case Intent.ACTION_USER_PRESENT:

		        DaemonService.setIsScreenOn(true);

		        dealAction(context, true);
		        break;
		      case Intent.ACTION_SCREEN_OFF:
		        DaemonService.setIsScreenOn(false);
		        dealAction(context, false);
		        break;
		    }

		  }


		  private void dealAction(Context context, boolean screenOn) {


		    if ((mType & DAEMON_USEACTIVITY) == DAEMON_USEACTIVITY) {
		      //如果使用空Activity保活则在屏幕开关相应的打开Activity或者关闭

		      if (screenOn) {
		        EventBus.getDefault().post(new CloseActionEvent(DaemonActivity.CLOSE_ACTION));
		      } else {
		        DaemonActivity.startActivity(context);
		      }

		    }

		    if ((mType & DAEMON_USENOTIFICATION) == DAEMON_USENOTIFICATION) {
		      //如果使用notification保活则在屏幕开关相应的打开notification或者关闭
		      Intent daemon = new Intent(SCREEN_ACTION);
		      daemon.setClass(context, DaemonService.InnerService.class);

		      if (screenOn) {
		        context.stopService(daemon);
		      } else {
		        context.startService(daemon);
		      }
		    }
		  }

* 音乐播放器的方式

音乐播放器在后台运行时系统也不会杀死，可以参考这种方式实现，这种方式跟空Notification是一样，不过不能再使用空的了，必须要有实际的内容才能，所以又得依靠屏幕的开关随时显示notification或关闭,不过在5.0之后通知会直接显示在锁屏界面上，这种方式用户会看见logo闪现，所以并不好。控制方式看第二点。

	  MediaSessionCompat mediaSessionCompat = new MediaSessionCompat(context, context.getPackageName());
	      mediaSessionCompat.setActive(true);
	      MediaStyle mediaStyle = new MediaStyle();
	      mediaStyle.setMediaSession(mediaSessionCompat.getSessionToken());


	      MediaControllerCompat mediaControllerCompat = new MediaControllerCompat(context, mediaSessionCompat);
	      mediaControllerCompat.getTransportControls().play();

	      NotificationCompat.Builder mBuilder =
	          new NotificationCompat.Builder(context)
	              .setColor(context.getResources().getColor(R.color.xx))
	              .setSmallIcon(R.drawable.notification_icon)
	              .setOngoing(true)
	              .setAutoCancel(false)
	              .setContentTitle(context.getString(R.string.xx))
	              .setContentText(context.getString(R.string.xx))
	              .setGroup(context.getString(R.string.xx))
	              .setStyle(mediaStyle)
	              .setCategory(NotificationCompat.CATEGORY_SERVICE)
	              .setVisibility(NotificationCompat.VISIBILITY_SECRET)
	              .setPriority(Notification.PRIORITY_MAX);

	      mBuilder.addAction(R.drawable.xx, context.getString(R.string.xx),
	          PendingIntent.getActivity(context, 1,
	              new Intent(context, xx.class), PendingIntent.FLAG_CANCEL_CURRENT));
	      mNotification = mBuilder.build();
	      mNotification.visibility = Notification.VISIBILITY_SECRET;
	    }
	    return mNotification;


## 利用第三方Push

关于第三方Push可以先看下我上一篇文章：[JPush，友盟，百度云，个推Push服务在保活上的对比](https://github.com/goodbranch/AndroidNote/blob/master/note/JPush%EF%BC%8C%E5%8F%8B%E7%9B%9F%EF%BC%8C%E7%99%BE%E5%BA%A6%E4%BA%91%EF%BC%8C%E4%B8%AA%E6%8E%A8Push%E6%9C%8D%E5%8A%A1%E5%9C%A8%E4%BF%9D%E6%B4%BB%E4%B8%8A%E7%9A%84%E5%AF%B9%E6%AF%94.md)

* 极光Push

官方文档中我们可以看到极光推送有互相拉起功能，配置如下：

	   <!-- since 1.8.0 option 可选项。用于同一设备中不同应用的JPush服务相互拉起的功能。 -->
	        <!-- 若不启用该功能可删除该组件，将不拉起其他应用也不能被其他应用拉起 -->
	         <service
	             android:name="cn.jpush.android.service.DaemonService"
	             android:enabled="true"
	             android:exported="true">
	             <intent-filter >
	                 <action android:name="cn.jpush.android.intent.DaemonService" />
	                 <category android:name="您应用的包名"/>
	             </intent-filter>
	         </service>

但实际如果你看懂了百度的那种做法你就明白，你完全可以不用集成极光推送就可以做到所有集成了极光的App启动时把你的App拉起，这个拉起功能最重要的就是` <action android:name="cn.jpush.android.intent.DaemonService" />`你只要配置了这个就行。

可以用珍爱，云购全球试下，不过这种方式在华为系统上是完全需要用户授权才能启动别的应用。

* 华为，小米Push

系统级别的Push给我们提供稳定的Push服务，额外加入这两个，可以极大提高你得Push到达率。如果你本身有自己的Push服务，华为Push你可以只使用它的透传功能，把自己的Push进程启动。


## 总结

系统级别Push服务是最好的选择，GCM还不能使用前，小米，华为是现在最好的选择，对用户也没什么影响。
