## Gradle Build Files in Android 第三章

### 了解`Build Types` 和 `Flavors`

#### 3.1 了解`Build Types` 

* **debug**和**release**

	Gradle Android 插件提供了两种build类型，**debug**和**release**，他们两者都可以在`buildTypes`节点中配置。例如默认配置：

				buildTypes {
				    release {
				      minifyEnabled false
				      proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
				    }
				}

	同样可以直接加入`debug`type ，两者配置都可以一样，这是用来区分打包类型。并且debug 默认 debuggable 为true。

* **minifyEnabled，shrinkResources**

	开发很久的项目中通常会存在很多已经不再使用的资源文件，但是人工去删除又比较麻烦，可以使用这两个帮助删除不再使用的资源文件。

				android {
				    buildTypes {
						release {
						minifyEnabled true
						shrinkResources true
						proguardFiles getDefaultProguardFile('proguard-android.txt'),
						  Turn on code shrinking
						'proguard-rules.pro'
						} }
				}

	这两个属性已经要同时设置才能生效。

* **设置后缀的属性**

	为了方便辨别打包后的包是release或debug，可以根据不同的打包类型设置区分。如修改版本名称，修改包名。

				android {
				// ... other properties ... buildTypes {
				        debug {
				            applicationIDsuffix '.debug'
				            versionNameSuffix '-debug'
				}
				// .. other build types ...
				} }

	这样配置后打包的debug包的包名会在后面增加`.debug`，版本名后也会相应的增加`-debug`,这样一个手机上可以很方便安装多个包。

#### 3.2 Flavors and Variants

**问题**

如何做到不同一次build同一个APP不同版本功能的包。

**解决方法**

`productFlavors`用于build不同版本的包，他可能用于免费版，收费版要看上去不一样。

例如

