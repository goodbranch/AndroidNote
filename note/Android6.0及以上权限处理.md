### Android 6.0及以上权限处理
--
>为了保证Android系统的完整性以及用户的隐私性，所有的APP都运行在一个单独的沙盒中，如果想要访问沙盒之外的资源必须有明确指定是否有该权限，在Android 6.0之前，只要指明了权限系统一般都会自动授权，也有需要询问用户是否授权。但是在Android 6.0以及后对于危险权限必须用户同意后才能进行。
>
>
#### 1.定义应用所需权限
> `App Manifest` 中定义所需权限，只定义所需的，不使用的不要定义。
>
#### 2.处理运行时权限请求
>
##### 1.权限组概念
>
>###### 一个或者多个权限组成一个权限组，例如` READ_CONTACTS` , `WRITE_CONTACTS` 和 ` GET_ACCOUNTS` 同属`CONTACTS`权限组，申请其中任一个权限系统都会提示同样的提示，且用户允许其中一个权限则权限组中其他权限自动允许。
>
##### 2.权限分类
>###### 权限分成正常权限和危险权限，如果不会威胁到用户隐私的权限通常会在安装时自动允许，而危险权限即使你再`Mainifest`列出来也会显式要用户处理。下面表中的危险权限在你APP `targetSdkVersion ` 大于等于23是需要特殊处理。
>
>
>![Dangerous permissions and permission groups](https://raw.githubusercontent.com/goodbranch/AndroidNote/master/note/image/permission_1.png)
>![Dangerous permissions and permission groups](https://raw.githubusercontent.com/goodbranch/AndroidNote/master/note/image/permission_2.png)
>
>
#### 3. 检测权限和请求权限
>
>#####1. 判断是否有相应权限
>
> `int permissionCheck = ContextCompat.checkSelfPermission(thisActivity,Manifest.permission.ACCESS_FINE_LOCATION);`返回的结果 `permissionCheck` 如果是允许则返回`PackageManager.PERMISSION_GRANTED` 否则 返回`PERMISSION_DENIED`,如果是允许了则直接使用。
>
>##### 2.请求权限
>
> `ActivityCompat.shouldShowRequestPermissionRationale(activity, p);` 返回是否需要一个我们自己定义的解释，默认返回 `false` 当用户选择拒绝后返回true，拒绝的同是选择不再询问则返回`false`
>
>`ActivityCompat.requestPermissions(activity, ps, requestCode);`请求相应权限,效果如下：
>
>![requestPermissions](https://raw.githubusercontent.com/goodbranch/AndroidNote/master/note/image/permission_request_1.png)
>
>
>
#### demo
>
>[源码](https://github.com/goodbranch/AndroidPermission.git)
>
>[APK](https://raw.githubusercontent.com/goodbranch/AndroidNote/master/note/Apk/androidPermission.apk)