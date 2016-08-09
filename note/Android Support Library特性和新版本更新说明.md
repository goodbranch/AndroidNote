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
> + Fragment  