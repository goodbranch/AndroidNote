### 一直以来我们都在使用Android提供的Support library，但是对于Support library 更多的信息没有了解，同时对于怎么选择support 的版本也没有明确的判断，下面将介绍Support library中每个包的特性以及新版本的更新说明。


#### 1.Support library 的特性

>Android Support Library package 包含了一些可以加入你项目中的libraries ，他们每一个都有他们支持的Android 系统版本范围和一些特性。
下面我们将会更加详细介绍每个包的重要特性以及支持的系统版本范围，这样能更加方便让你选择把哪一个library加入到自己的应用中，通常我们推荐你使用[v4 support](https://developer.android.com/topic/libraries/support-library/features.html#v4)和[v7 appcompat](https://developer.android.com/topic/libraries/support-library/features.html#v7-appcompat),因为这两个library支持很广的Android版本和我们推荐使用的用户界面模式。

>为了使用下面这些libraries，你必须在Android SDK installation 中下载他们，具体步骤可以查看[ Support Library Setup](https://developer.android.com/topic/libraries/support-library/setup.html#download),不过通常我们都是直接在build.gradle中这样写就可以了。

	 dependencies {
	    ...
	    compile "com.android.support:support-v4:24.1.1"
	}




#### 2.每个包的重要特性

##### 1.v4 Support Library
--
V4包是我们用的最广的，他支持从Android 1.6(API level 4)开始，他是所有支持包中APIS最多的，包含应用组件，UI， 无障碍，数据处理，网络连接，工具类，下面v4包中一些重要的类。

+ App Components
> + [Fragment](https://developer.android.com/reference/android/support/v4/app/Fragment.html)-提供碎片化一整套UI以及功能上的处理，使应用适配大屏幕还是小屏幕上。
> + [NotificationCompat](https://developer.android.com/reference/android/support/v4/app/NotificationCompat.html)-提供更加丰富的Notification特性。
> + [LocalBroadcastManager ](https://developer.android.com/reference/android/support/v4/content/LocalBroadcastManager.html)-可以简单的注册一个广播接收器并且发送只在应用内的广播（Context.sendBroad()发送的广播在整个手机上注册了Action的应用都可以接受到）。
+ User Interface
> + [ViewPager](https://developer.android.com/reference/android/support/v4/view/ViewPager.html)-我们常用的左右滑动控件
> + [PagerTitleStrip](https://developer.android.com/reference/android/support/v4/view/PagerTitleStrip.html)-
> + [PagerTitleStrip]()-
> + [PagerTabStrip]()-
> + [DrawerLayout]()- 
> + [SlidingPaneLayout ]()-
+ Accessibility
> + [ExploreByTouchHelper]()- 
> + [AccessibilityEventCompat]()- AccessibilityEvent的支持类
> + [AccessibilityNodeInfoCompat]()- AccessibilityNodeInfo的支持类
> + [AccessibilityNodeProviderCompat]()- AccessibilityNodeProvider的支持类
> + [AccessibilityDelegateCompat]()- View.AccessibilityDelegate的支持类
这里我们也发现当你使用的类不能支持你当前应用的最低版本，则可以在当前类名后面加上`Compat`看是否V4包中给我们提供了。
+ Content
> + [Loader](使用AsyncTaskLoader加载数据.md)- 在Activity和Fragment中实现异步加载数据同时伴随生命周期，可以看他的实现类`CursorLoader`和`AsyncTaskLoader`。
> + [FileProvider]()- 帮助我们在不同应用之间分享数据

##### 2.Multidex Support Library
--
> 当我们的工程项目在打包成应用是如果应用项目方法超过65536时则会报错，这时我们需要使用`Multidex Support Library`把应用打包分成多个DEX文件，使用方法如下：
 `com.android.support:multidex:1.0.0` 然后在我们自己的`xxApplication extends MultidexApplication` 就OK了。

##### 3.v7 Support Libraries
--
v7支持包中包含了一些library，他们的Android使用版本最低 Android 2.1(API level 7)，他们分别是以下7个：

###### 1.v7 appcompat library

> 这个library提供了`ActionBar`的UI设计模式，并且包含了支持`material design`的UI设计模式的实现。
注意：这个library必须结合V4一起使用

> 下面是v7 appcompat library中关键的一些类：

> + [ActionBar]()-
+ [AppCompatActivity]()-实现了v7 appcompat action bar UI设计，可以作为基类的Activity
+ [AppCompatDialog]()-实现了AppCompat themed 可以用作基类的Dialog
+ [ShareActionProvider]()-提供标准的发送邮件或者分享到社交应用的组件。

>在项目中这样加入：`com.android.support:appcompat-v7:24.1.1`

###### 2.[v7 cardview library](https://developer.android.com/reference/android/support/v7/widget/CardView.html)

> 这个library提供了`CardView`控件，可以让我们像卡片一样展示，这些卡片充分的使用的`material design` ,并且在TV apps中使用非常普遍。

> 可以这样加入项目：`com.android.support:cardview-v7:24.1.1`


###### 3.[v7 gridlayout library](https://developer.android.com/reference/android/support/v7/widget/package-summary.html)

> 为我们提供了`GridLayout`网格布局控件

> 在项目中这样加入：com.android.support:gridlayout-v7:24.1.1

###### 4.[v7 mediarouter library](https://developer.android.com/reference/android/support/v7/media/package-summary.html)

> 这个library中提供了` MediaRouter`,` MediaRouteProvider`和一些能支持`Google Cast` 的media 类。

> 在v7这个library中提供了让我们从当前设备录制媒体频道和流发送到外置屏幕，扬声器，和其他一些远程设备上，并且这个library还提供了搜索选择远程设备的APIS以及检测媒体状态等等，

> 在项目中这样加入：com.android.support:mediarouter-v7:24.1.1


###### 5.[v7 palette library](https://developer.android.com/reference/android/support/v7/graphics/Palette.html)

> 提供一个可以从Image中获取颜色的`Palette`,比如一个音乐播放器会通过这个类把专辑封面主要颜色获得，然后把这些颜色制作一个颜色协调的标题卡片。

> 在项目中这样加入：com.android.support:palette-v7:24.1.1


###### 6.v7 recyclerview library

> 这个library中提供了` RecyclerView`,可以想ListView那样高效展示数据。

> 在项目中这样加入：com.android.support:recyclerview-v7:24.1.1


###### 7.v7 Preference Support Library

> 在我们的preference包下提供了SharePreference 数据绑定方式的APIS，例如`CheckBoxPreference` 和 `ListPreference`，

> 在项目中这样加入：com.android.support:preference-v7:24.1.1

##### 4.v8 Support Library
--
顾名思义v8最低支持android 2.1(API level 8) 这个library中也有他独有的一些特性。

###### 1.v8 renderscript library

> 


##### 5.v13 Support Library
--

最低版本支持Android 3.2(API level 13) ,这个library中提供了为Fragment提供了FragmentCompat以及额外的支持类。

在项目中这样加入：`com.android.support:support-v13:24.1.1`

##### 6.v14 Preference Support Library
--



在项目中这样加入：`com.android.support:preference-v14:24.1.1`


##### 7.v17 Preference Support Library for TV
--

在项目中这样加入：`com.android.support:preference-leanback-v17:24.1.1`


##### 8.v17 Leanback Library
--

在项目中这样加入：`com.android.support:leanback-v17:24.1.1`


##### 9.Annotations Support Library
--

在项目中这样加入：`com.android.support:support-annotations:24.1.1`


##### 10.Design Support Library
--

在项目中这样加入：`com.android.support:design:24.1.1`


##### 11.Custom Tabs Support Library
--

在项目中这样加入：`com.android.support:customtabs:24.1.1`


##### 12.Percent Support Library
--

在项目中这样加入：`com.android.support:percent:24.1.1`


##### 13.App Recommendation Support Library for TV
--

在项目中这样加入：`com.android.support:recommendation:24.1.1`







