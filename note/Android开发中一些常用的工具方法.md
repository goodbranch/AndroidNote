##1. 获取debug状态

```
/**
* 检测是否处于Debug模式
*
* @return boolean
*/
 public static boolean isDebugMode(Context context) {
	ApplicationInfo info =    context.getApplicationInfo();
			    if (info != null && (info.flags &    ApplicationInfo.FLAG_DEBUGGABLE) != 0) {
	      return true;
    }
    return false;
    }
```

 
## 2. 得到当前版本信息

```
  /**
   * 获取软件版本号
   */
  public static String getVersonName(Context context) {
    String versionName = null;
    PackageManager pm = context.getPackageManager();
    PackageInfo info = null;

    try {
      info = pm.getPackageInfo(context.getApplicationContext().getPackageName(), 0);
    } catch (NameNotFoundException e) {
      e.printStackTrace();
    }

    if (info != null) {
      versionName = info.versionName;
    }
    return versionName;
  }
```

## 3. 得到系统名

```
 public static String getOsName() {
	 String MANUFACTURER = Build.MANUFACTURER;
     return MANUFACTURER;
  }
```

## 4. 判断当前字符是数字

```
public static boolean isNumeric(String str) {
    if (TextUtils.isEmpty(str)) {
      return false;
    }
    Pattern pattern = Pattern.compile("[0-9]*");
    Matcher isNum = pattern.matcher(str);
    return isNum.matches();
    }
```

  
## 5. 验证手机号码

```
/**
   * 验证手机格式
   */
  public static boolean isMobileNumber(String mobileNumber) {
    /*
     * 移动：134、135、136、137、138、139、150、151、157(TD)、140(TD)、158、159、187、188
     * 联通：130、131、132、152、155、156、185、186 电信：133、153、180、189、（1349卫通）
     * 总结起来就是第一位必定为1，第二位必定为3或5或8，其他位置的可以为0-9 虚拟运营商：170
     */
    // String regex = "[1][34578]\\d{9}";//
    // "[1]"代表第1位为数字1，"[358]"代表第二位可以为3、5、8中的一个，"\\d{9}"代表后面是可以是0～9的数字，有9位。
    String regex = "[1]\\d{10}";
    if (TextUtils.isEmpty(mobileNumber)) {
      return false;
    } else {
      return mobileNumber.matches(regex);
    }
  }
```

## 6.  Bitmap颜色转换

```
 public static Bitmap translateBitmap(Bitmap bm, int r, int g, int b) {

    if (bm == null)
      return null;

    try {
      int w = bm.getWidth();
      int h = bm.getHeight();
      int size = w * h;
      int[] array = new int[size];
      bm.getPixels(array, 0, w, 0, 0, w, h);

      for (int i = 0; i < h; i++) {
        for (int j = 0; j < w; j++) {
          if (array[i * w + j] != 0) {
            int color = array[i * w + j];

            int alpha = ((color & 0xff000000) >> 24);
            int red = r;
            int green = g;
            int blue = b;

            int pix = (alpha << 24) | (red << 16) | (green << 8) | blue;
            array[i * w + j] = pix;

          }
        }

      }
      if (bm.isMutable()) //此情况可以直接改变像素点
      {
        try {
          bm.setPixels(array, 0, w, 0, 0, w, h);
        } catch (Exception e) {I
          //设置像素点出错
        }
      } else {
        bm = Bitmap.createBitmap(array, w, h, Config.ARGB_8888);
      }
      return bm;
    } catch (OutOfMemoryError e) {
      e.printStackTrace();
    }

    return bm;
  }

```

## 7. 通过View生成Bitmap

```
  /**
   * Returns a bitmap showing a screenshot of the view passed in.
   */
  private Bitmap getBitmapFromView(View v) {
    Bitmap bitmap = Bitmap.createBitmap(v.getWidth(), v.getHeight(), Bitmap.Config.ARGB_8888);
    Canvas canvas = new Canvas(bitmap);
    v.draw(canvas);
    return bitmap;
  }
```

## 8. dp,px 转换

```
  /**
     * Converts an unpacked complex data value holding a dimension to its final floating 
     * point value. The two parameters <var>unit</var> and <var>value</var>
     * are as in {@link #TYPE_DIMENSION}.
     *  
     * @param unit The unit to convert from.
     * @param value The value to apply the unit to.
     * @param metrics Current display metrics to use in the conversion -- 
     *                supplies display density and scaling information.
     * 
     * @return The complex floating point value multiplied by the appropriate 
     * metrics depending on its unit. 
     */
    public static float applyDimension(int unit, float value,
                                       DisplayMetrics metrics)
    {
        switch (unit) {
        case COMPLEX_UNIT_PX:
            return value;
        case COMPLEX_UNIT_DIP:
            return value * metrics.density;
        case COMPLEX_UNIT_SP:
            return value * metrics.scaledDensity;
        case COMPLEX_UNIT_PT:
            return value * metrics.xdpi * (1.0f/72);
        case COMPLEX_UNIT_IN:
            return value * metrics.xdpi;
        case COMPLEX_UNIT_MM:
            return value * metrics.xdpi * (1.0f/25.4f);
        }
        return 0;
    }
```
##### 这个方法在TypedValue中

## 9. 判断网络状态是否可以用

```
 public static boolean isNetWorkAvailable(Context mContext) {
    ConnectivityManager mConnectivity = (ConnectivityManager) mContext.getSystemService(Context.CONNECTIVITY_SERVICE);
    NetworkInfo nInfo = mConnectivity.getActiveNetworkInfo();
    if (nInfo == null) {
      return false;
    } else {
      return nInfo.isConnected();
    }


  }
```
##10.得到当前设备名,例如，vivoX3,HUAWEIP7-L10

```
  private String getModel() {

    String model = android.os.Build.MODEL;

    if (model != null) {

      model = model.trim().replaceAll("\\s*", "");
    } else {

      model = "";
    }

    return model;
  }
```

## 11. 获取`AndroidManifest.xml` 中`meta`中的数据
```
   PackageManager packageManager = mContext.getPackageManager();
   String packageName = mContext.getPackageName();
   ApplicationInfo applicationInfo=packageManager
   .getApplicationInfo(packageName,PackageManager.GET_META_DATA);
   
   applicationInfo.metaData.getXX(x,x);
``` 
## 12. 打开Apk文件进行安装
```
    Intent intent = new Intent();
    intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
    intent.setAction(android.content.Intent.ACTION_VIEW);
    intent.setDataAndType(Uri.fromFile(file),
        "application/vnd.android.package-archive");
    context.startActivity(intent);
```
## 13.打开应用市场
```
     Uri uri = Uri.parse("market://details?id=" + this.getPackageName());
     Intent intent = new Intent(Intent.ACTION_VIEW, uri);
     try {
          //可能手机上没有安装应用市场
          startActivity(entry.jumpIntent);
         } catch (ActivityNotFoundException e) {
           
         }
```

### 后续过程中再补充
