
### 运行monkeyrunner报错：

Please set ANDROID_SWT to point to the folder containing swt.jar for your platform.

### 原因

monkeyrunner 找不到swt.jar,而swt.jar存在sdk tools/lib/[x86|x86_64]目录中。

### 修复如下：

step 1

修改`monkeyrunner.bat`

注释这一段：

	if exist %frameworkdir%\%jarfile% goto JarFileOk
	  set frameworkdir=lib

	if exist %frameworkdir%\%jarfile% goto JarFileOk
	 set frameworkdir=..\framework

添加：

	set frameworkdir=..\lib

step 2

step 1修改后运行还是会报错：

Exception in thread "main" java.lang.IllegalArgumentException: java.io.IOExcepti
on: Cannot run program "..\framework\adb.exe": CreateProcess error=2, 系统找不到
指定的文件。

所以继续在android sdk tools目录下创建`framework`目录并且把`adb.exe`复制进去。



然后就可以正常使用了。




