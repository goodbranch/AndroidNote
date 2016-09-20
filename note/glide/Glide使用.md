## Glide的使用

   Glide是google开发用于Android加载媒体的类库，包括图片,gif,video,已经在很多项目中使用，灵活快速。下面我们看如何使用它,如果有什么不对的请不吝指教


* ### 导入到项目中

		dependencies {
		    compile fileTree(include: ['*.jar'], dir: 'libs')
			testCompile 'junit:junit:4.12'
			compile 'com.android.support:appcompat-v7:24.1.1'
			compile 'com.github.bumptech.glide:glide:3.7.0'
			compile 'com.android.support:support-v4:24.1.1'
			}

    Glide同样需要*Android Support Library v4*，请记得加上**support-v4**，不过现在项目基本都包含**support-v4**。

* ### 加载图片

		  Glide.with(this).load("http://inthecheesefactory.com/uploads/source/glidepicasso/cover.jpg")
		          .asBitmap()
		          .into(mImageView);

* #### Glide的生命周期

    ![glide-with](https://raw.githubusercontent.com/goodbranch/AndroidNote/master/note/glide/glide-with-1.png)

    `Glide.with()` 不仅仅只是`Context`还可以是`Activity`,`Fragment`等，传入后自动适配，Glide加载图片是会随着`Activity`,`Fragment` 的生命周期，具体可以参考`LifecycleListener`，所以推荐使用`Activity`,`Fragment`.

* #### 设置图片大小

    `.override(200,200)`可以设置加载图片大小，但是实际大小不一定是200x200,通过源码分析下：

    在`BitmapRequestBuilder`中默认的`    private Downsampler downsampler = Downsampler.AT_LEAST;`

	     /**
	     * Load and scale the image uniformly (maintaining the image's aspect ratio) so that the smallest edge of the
	     * image will be between 1x and 2x the requested size. The larger edge has no maximum size.
	     */
	    public static final Downsampler AT_LEAST = new Downsampler() {
	        @Override
	        protected int getSampleSize(int inWidth, int inHeight, int outWidth, int outHeight) {
	            return Math.min(inHeight / outHeight, inWidth / outWidth);
	        }

	        @Override
	        public String getId() {
	            return "AT_LEAST.com.bumptech.glide.load.data.bitmap";
	        }
	    };


    然后在`Downsampler`中的`decode`方法中，获取的Bitmap大小变成1/sampleSize，倍数通过`getSampleSize`计算所得，


            options.inTempStorage = bytesForOptions;

            final int[] inDimens = getDimensions(invalidatingStream, bufferedStream, options);
            final int inWidth = inDimens[0];
            final int inHeight = inDimens[1];

            final int degreesToRotate = TransformationUtils.getExifOrientationDegrees(orientation);
            final int sampleSize = getRoundedSampleSize(degreesToRotate, inWidth, inHeight, outWidth, outHeight);

            final Bitmap downsampled =
                    downsampleWithSize(invalidatingStream, bufferedStream, options, pool, inWidth, inHeight, sampleSize,
                            decodeFormat);

            // BitmapFactory swallows exceptions during decodes and in some cases when inBitmap is non null, may catch
            // and log a stack trace but still return a non null bitmap. To avoid displaying partially decoded bitmaps,
            // we catch exceptions reading from the stream in our ExceptionCatchingInputStream and throw them here.
            final Exception streamException = exceptionStream.getException();
            if (streamException != null) {
                throw new RuntimeException(streamException);
            }

            Bitmap rotated = null;
            if (downsampled != null) {
                rotated = TransformationUtils.rotateImageExif(downsampled, pool, orientation);

                if (!downsampled.equals(rotated) && !pool.put(downsampled)) {
                    downsampled.recycle();
                }
            }


    inSampleSize 是 BitmapFactory.Options的属性，应该大家都知道。然后再看看怎么生成`.override(200,200)`，如果没有设置Glide默认是`FitCenter`，查看`FitCenter`可以看到图片截取方式。


		public class FitCenter extends BitmapTransformation {

		    public FitCenter(Context context) {
		        super(context);
		    }

		    public FitCenter(BitmapPool bitmapPool) {
		        super(bitmapPool);
		    }

		    @Override
		    protected Bitmap transform(BitmapPool pool, Bitmap toTransform, int outWidth, int outHeight) {
		        return TransformationUtils.fitCenter(toTransform, pool, outWidth, outHeight);
		    }

		    @Override
		    public String getId() {
		        return "FitCenter.com.bumptech.glide.load.resource.bitmap";
		    }
		}

    --

		 public static Bitmap fitCenter(Bitmap toFit, BitmapPool pool, int width, int height) {
		        if (toFit.getWidth() == width && toFit.getHeight() == height) {
		            if (Log.isLoggable(TAG, Log.VERBOSE)) {
		                Log.v(TAG, "requested target size matches input, returning input");
		            }
		            return toFit;
		        }
		        final float widthPercentage = width / (float) toFit.getWidth();
		        final float heightPercentage = height / (float) toFit.getHeight();
		        final float minPercentage = Math.min(widthPercentage, heightPercentage);

		        // take the floor of the target width/height, not round. If the matrix
		        // passed into drawBitmap rounds differently, we want to slightly
		        // overdraw, not underdraw, to avoid artifacts from bitmap reuse.
		        final int targetWidth = (int) (minPercentage * toFit.getWidth());
		        final int targetHeight = (int) (minPercentage * toFit.getHeight());

		        if (toFit.getWidth() == targetWidth && toFit.getHeight() == targetHeight) {
		            if (Log.isLoggable(TAG, Log.VERBOSE)) {
		                Log.v(TAG, "adjusted target size matches input, returning input");
		            }
		            return toFit;
		        }

		        Bitmap.Config config = getSafeConfig(toFit);
		        Bitmap toReuse = pool.get(targetWidth, targetHeight, config);
		        if (toReuse == null) {
		            toReuse = Bitmap.createBitmap(targetWidth, targetHeight, config);
		        }
		        // We don't add or remove alpha, so keep the alpha setting of the Bitmap we were given.
		        TransformationUtils.setAlpha(toFit, toReuse);

		        if (Log.isLoggable(TAG, Log.VERBOSE)) {
		            Log.v(TAG, "request: " + width + "x" + height);
		            Log.v(TAG, "toFit:   " + toFit.getWidth() + "x" + toFit.getHeight());
		            Log.v(TAG, "toReuse: " + toReuse.getWidth() + "x" + toReuse.getHeight());
		            Log.v(TAG, "minPct:   " + minPercentage);
		        }

		        Canvas canvas = new Canvas(toReuse);
		        Matrix matrix = new Matrix();
		        matrix.setScale(minPercentage, minPercentage);
		        Paint paint = new Paint(PAINT_FLAGS);
		        canvas.drawBitmap(toFit, matrix, paint);

		        return toReuse;
		    }

    这里算出图片的大小：

                final float widthPercentage = width / (float) toFit.getWidth();
		        final float heightPercentage = height / (float) toFit.getHeight();
		        final float minPercentage = Math.min(widthPercentage, heightPercentage);


    测试中我们使用的图片是1080x540,最终生成了200x100.设置`approximate()`对应`Downsampler.AT_LEAST`,`asIs()`对应`Downsampler.NONE`,`atMost()`对应`Downsampler.AT_MOST`，具体情况可查看`Downsampler`


 * #### 图片缓存策略

    测试加载的图片大小为1080x540，如果使用了`.override(200,200)`默认缓存一张200x100的图片,也就是默认存储结果`RESULT`，如果想要改变缓存的策略可以这样设置：

    `.diskCacheStrategy(DiskCacheStrategy.ALL)`

    DiskCacheStrategy 分别有以下几种选择，`ALL`缓存原图和截取后的图，`NONE` 不缓存，`SOURCE` 只缓存原图，`RESULT`缓存截取后的图：

			 public enum DiskCacheStrategy {
			    /** Caches with both {@link #SOURCE} and {@link #RESULT}. */
			    ALL(true, true),
			    /** Saves no data to cache. */
			    NONE(false, false),
			    /** Saves just the original data to cache. */
			    SOURCE(true, false),
			    /** Saves the media item after all transformations to cache. */
			    RESULT(false, true);

			    private final boolean cacheSource;
			    private final boolean cacheResult;

			    DiskCacheStrategy(boolean cacheSource, boolean cacheResult) {
			        this.cacheSource = cacheSource;
			        this.cacheResult = cacheResult;
			    }

			    /**
			     * Returns true if this request should cache the original unmodified data.
			     */
			    public boolean cacheSource() {
			        return cacheSource;
			    }

			    /**
			     * Returns true if this request should cache the final transformed result.
			     */
			    public boolean cacheResult() {
			        return cacheResult;
			    }
			}


    查看本地缓存如下：

    ![DiskCacheStrategy](https://raw.githubusercontent.com/goodbranch/AndroidNote/master/note/glide/glide-DiskCacheStrategy-1.png)


    如果图片需要分享或需要原图的建议缓存`ALL`，否则只缓存`RESULT`。

* ### Glide全局配置


1. **创建`GlideModel`**


		public class GlideConfigModule implements GlideModule {
		  @Override
		  public void applyOptions(Context context, GlideBuilder builder) {


		  }

		  @Override
		  public void registerComponents(Context context, Glide glide) {

		  }
		}


2. **在`AndroidManifest.xml`的`meta-data`配置`GlideModule`**

	    <meta-data android:name="com.branch.www.glidedemo.GlideConfigModule"
	               android:value="GlideModule"/>

3. **解决`GlideModel`冲突**

    有可能加入的library中也同样配置了`GlideModule`,如果配置了多个会出现冲突，无法编译运行，解决方式可在`AndroidManifest.xml`移除

		<meta-data android:name=”com.mypackage.MyGlideModule” tools:node=”remove” />

4. **默认Bitmap Format 是 RGB_565**

    为了降低内存消耗，Glide默认配置的Bitmap Format 为 RGB_565，修改GlideBuilder's `setDecodeFormat`设置

		    builder.setDecodeFormat(DecodeFormat.PREFER_ARGB_8888);

5. **配置Disk缓存**

    使用GlideBuilder's `setDiskCache()`方法设置缓存目录和大小。
    默认Glide使用`InternalCacheDiskCacheFactory`，默认最大缓存250M，这是一个应用内部目录的缓存，缓存的图片只能本应用可以有权获取。

		  builder.setDiskCache(
		  new InternalCacheDiskCacheFactory(context, yourSizeInBytes));

		  builder.setDiskCache(
		  new InternalCacheDiskCacheFactory(context, cacheDirectoryName, yourSizeInBytes));

    你也可以使用`ExternalCacheDiskCacheFactory`去设置SD卡缓存

		  builder.setDiskCache(
		  new ExternalCacheDiskCacheFactory(context, cacheDirectoryName, yourSizeInBytes));

    或者使用DiskLruCacheFactory配置自己管理的缓存目录和大小

			// If you can figure out the folder without I/O:
			// Calling Context and Environment class methods usually do I/O.
			builder.setDiskCache(
			  new DiskLruCacheFactory(getMyCacheLocationWithoutIO(), yourSizeInBytes));

			// In case you want to specify a cache folder ("glide"):
			builder.setDiskCache(
			  new DiskLruCacheFactory(getMyCacheLocationWithoutIO(), "glide", yourSizeInBytes));

			// In case you need to query the file system while determining the folder:
			builder.setDiskCache(new DiskLruCacheFactory(new CacheDirectoryGetter() {
			    @Override public File getCacheDirectory() {
			        return getMyCacheLocationBlockingIO();
			    }
			}), yourSizeInBytes);

    或者你想完全控制缓存可以通过实现`DiskCache.Factory`然后用`DiskLruCacheWrapper`去创建你期望的目录。

		    builder.setDiskCache(new DiskCache.Factory() {
		    @Override public DiskCache build() {
		        File cacheLocation = getMyCacheLocationBlockingIO();
		        cacheLocation.mkdirs();
		        return DiskLruCacheWrapper.get(cacheLocation, yourSizeInBytes);
		     }
		    });

	如果你不需要任何缓存可以使用`DiskCacheAdapter`或者自己实现`DiskCache`，`DiskCacheAdapter`内部没有任何实现。

6. **Bitmap Pool**

    为了避免Bitmap频繁解码，我们通常会在系统内存中缓存一部分经常使用的图片。GlideBuilder's `setBitmapPool()`可以设置你想要的缓存策略，缓存大小。例如我们常用的LRU策略。

    		builder.setBitmapPool(new LruBitmapPool(sizeInBytes));


    或实现`BitmapPool`自定义一个吧。

    Glide所有的基本配置都在`GlideModule`内，使用Glide提供的缓存方式如下：

		builder.setDiskCache(new DiskLruCacheFactory(dirPath, DiskCache.Factory.DEFAULT_DISK_CACHE_SIZE));

    如果需要自定义实现`DiskCache.Factory`。

    GlideModule 更多配置如下：

    ![GlideModule](https://raw.githubusercontent.com/goodbranch/AndroidNote/master/note/glide/glide-GlideBuilder-1.png)

* ### 自定义加载

1. **SimpleTarget**
   
    如果单纯的想获得Bitmap，显不显示或者其他再定，则可以使用`SimpleTarget`

	     Glide.with(this).load("http://inthecheesefactory.com/uploads/source/glidepicasso/cover.jpg")
	          .asBitmap()
	          .diskCacheStrategy(DiskCacheStrategy.ALL)
	          .into(new SimpleTarget<Bitmap>() {
	            @Override
	            public void onResourceReady(Bitmap resource, GlideAnimation<? super Bitmap> glideAnimation) {

	            }
	          });

	需要注意的是如果你想获取Bitmap那么一定不想因为`Activity`,`Fragment`的生命周期影响，因此`Glide.with(context)`使用Context,同时为了避免内存泄露建议`SimpleTarget`使用静态内部类而不是内部类（静态内部类不持有外部引用）。

2. **ViewTarget**

    如果想要在图片加载完成后设置一些动画则可以使用`ViewTarget`

		     Glide.with(this).load("http://inthecheesefactory.com/uploads/source/glidepicasso/cover.jpg")
		          .asBitmap()
		          .override(500,500)
		          .diskCacheStrategy(DiskCacheStrategy.ALL)
		          .into(new ViewTarget<ImageView, Bitmap>(mImageView) {
		            @Override
		            public void onResourceReady(Bitmap resource, GlideAnimation<? super Bitmap> glideAnimation) {

		              this.view.setImageBitmap(resource);
		            }
		          });
		    
    如果`.asGif()`使用`GlideDrawable `替换`Bitmap`，同时`ViewTarget`中包含`onStart()`, `onStop()`, `onDestroy()`生命周期回调。

    如果想要继续保留默认的特性可以相应的使用`GlideDrawableImageViewTarget`在`asGif()`之后，使用`BitmapImageViewTarget `在`asBitmap()`之后。

* ### 自定义url

    通过`width` x `height`拼接url下载指定大小的图片，或者根据屏幕大小选择高，中，低三种分别率的图片url。

    使用http或https加载图片可以继承`BaseGlideUrlLoader`

		    public interface MyDataModel {
		    public String buildUrl(int width, int height);
		    } 

		    public class MyUrlLoader extends BaseGlideUrlLoader<MyDataModel> {
		    @Override
		    protected String getUrl(MyDataModel model, int width, int height) {
		        // Construct the url for the correct size here.
		        return model.buildUrl(width, height);
		    }
		    }

    `MyDataModel`就相当于我们的媒体model,里面有当前媒体的类型(image/gif/video)，url等属性。

    然后你可以在加载是这样使用：

		     Glide.with(yourFragment)
		    .using(new MyUrlLoader())
		    .load(yourModel)
		    .into(yourView);

    如果不想每次都使用`.using(new MyUrlLoader())`，可以在前面我们的`GlideModule `中配置

		    public class MyGlideModule implements GlideModule {
		    ...
		    @Override
		    public void registerComponents(Context context, Glide glide) {
		        glide.register(MyDataModel.class, InputStream.class, 
		            new MyUrlLoader.Factory());
		    }
		    }
    
    配置后加载时跳过`.using()`


* ### <span id="getBitmap">在后台线程下载</span>

1. **downloadOnly**

    `downloadOnly`有异步版和同步版，如果已经在后台线程中执行必须同步版

		    FutureTarget<File> future = Glide.with(applicationContext)
		    .load(yourUrl)
		    .downloadOnly(500, 500);
		    File cacheFile = future.get();

    这种方式在主线程中会阻塞主线程。如果想要在主线程中执行可以使用前面讲到的`SimpleTarget`

	        Glide.with(this)
	          .load("http://inthecheesefactory.com/uploads/source/glidepicasso/cover.jpg")
	          .downloadOnly(new SimpleTarget<File>() {
	            @Override
	            public void onResourceReady(File resource, GlideAnimation<? super File> glideAnimation) {

	            }
	          });

2. **into**

    使用into可以用于下载，下面是在后台线程中的使用：

		    Bitmap myBitmap = Glide.with(applicationContext)
		    .load(yourUrl)
		    .asBitmap()
		    .centerCrop()
		    .into(500, 500)
		    .get()


    如果想在主线程中使用同样可以使用`SimpleTarget`

* ### 清除缓存

		Glide.get(getApplicationContext()).clearMemory();
		Glide.get(getApplicationContext()).clearDiskCache();


* ### 怎么使缓存失效

    有时我们并不是想要清理所有缓存，只是app版本变动则原来的缓存可能不需要了，或本地图片库中图片变更但是文件名地址没有变则可能就出现继续使用原来的缩略图或`.override(200,200)`使用的小图，其实最好的情况是如果数据变更则相应的改变url，如果不改变可以使用`StringSignature`解决这个问题.

1. **通过版本使缓存失效**

		Glide.with(yourFragment)
		    .load(yourFileDataModel)
		    .signature(new StringSignature(yourVersionMetadata))
		    .into(yourImageView);

2. **`MediaStore`中的数据**

		Glide.with(fragment)
		    .load(mediaStoreUri)
		    .signature(new MediaStoreSignature(mimeType, dateModified, orientation))
		    .into(view);

3. **自定义**

    实现`Key`接口，重写`equals()`, `hashCode()` 和 `updateDiskCacheKey()` 方法，可以参考`StringSignature`或`MediaStoreSignature`的实现。


* ### Transformations

1. **默认的Transformations**

    **Fit center**

    相当于Android's `ScaleType.FIT_CENTER.`

		    Glide.with(yourFragment)
		    .load(yourUrl)
		    .fitCenter()
		    .into(yourView);

    **Center crop**

    相当于Android's `ScaleType.CENTER_CROP`

		    Glide.with(yourFragment)
		    .load(yourUrl)
		    .centerCrop()
		    .into(yourView);


2. **自定义transformations**

    最简单的方式是集成`BitmapTransformation`

		    private static class MyTransformation extends BitmapTransformation {

		    public MyTransformation(Context context) {
		       super(context);
		    }

		    @Override
		    protected Bitmap transform(BitmapPool pool, Bitmap toTransform, 
		            int outWidth, int outHeight) {
		       Bitmap myTransformedBitmap = ... // apply some transformation here.
		       return myTransformedBitmap;
		    }

		    @Override
		    public String getId() {
		        // Return some id that uniquely identifies your transformation.
		        return "com.example.myapp.MyTransformation";
		    }
		    }
    
    然后使用

    `.transform(new MyTransformation(context))`

    [Transformations](https://github.com/wasabeef/glide-transformations) 有各种丰富的效果。

    `BitmapTransformation`可以用于改变bitmap形状，颜色，截取，放大，所有等操作。

* ### 设置占位图和加载错误图

		.placeholder(R.drawable.placeholder)
		.error(R.drawable.imagenotfound)

* ### ProGuard

		-keep public class * implements com.bumptech.glide.module.GlideModule
		-keep public enum com.bumptech.glide.load.resource.bitmap.ImageHeaderParser$** {
		  **[] $VALUES;
		  public *;
		}
		-keepresourcexmlelements manifest/application/meta-data@value=GlideModule

* ### 相关问题问答

1. **[通过url获取已经下载的图片](#getBitmap)**

    使用`downloadOnly`或`into`

2. **是否支持Webp**
   
    Android 4.0 以上能支持


3. **在`AndroidManifest.xml`的`meta-data`配置`GlideModule`**是如何获取的

    查看源码可以在`Glide.get(Context)`

		     public static Glide get(Context context) {
		    if (glide == null) {
		      synchronized (Glide.class) {
		        if (glide == null) {
		          Context applicationContext = context.getApplicationContext();
		          List<GlideModule> modules = new ManifestParser(applicationContext).parse();

		          GlideBuilder builder = new GlideBuilder(applicationContext);
		          for (GlideModule module : modules) {
		            module.applyOptions(applicationContext, builder);
		          }
		          glide = builder.createGlide();
		          for (GlideModule module : modules) {
		            module.registerComponents(applicationContext, glide.registry);
		          }
		        }
		      }
		    }

		    return glide;
		  }

    可以看到`GlideModule`的获取在`ManifestParser`同时也发现可以设置多个，不过同样的配置后面的会覆盖前面的。那么进入`ManifestParser`查看是怎么实现的。

			public final class ManifestParser {
			  private static final String GLIDE_MODULE_VALUE = "GlideModule";

			  private final Context context;

			  public ManifestParser(Context context) {
			    this.context = context;
			  }

			  public List<GlideModule> parse() {
			    List<GlideModule> modules = new ArrayList<>();
			    try {
			      ApplicationInfo appInfo = context.getPackageManager()
			          .getApplicationInfo(context.getPackageName(), PackageManager.GET_META_DATA);
			      if (appInfo.metaData != null) {
			        for (String key : appInfo.metaData.keySet()) {
			          if (GLIDE_MODULE_VALUE.equals(appInfo.metaData.get(key))) {
			            modules.add(parseModule(key));
			          }
			        }
			      }
			    } catch (PackageManager.NameNotFoundException e) {
			      throw new RuntimeException("Unable to find metadata to parse GlideModules", e);
			    }

			    return modules;
			  }

			  private static GlideModule parseModule(String className) {
			    Class<?> clazz;
			    try {
			      clazz = Class.forName(className);
			    } catch (ClassNotFoundException e) {
			      throw new IllegalArgumentException("Unable to find GlideModule implementation", e);
			    }

			    Object module;
			    try {
			      module = clazz.newInstance();
			    } catch (InstantiationException | IllegalAccessException e) {
			      throw new RuntimeException("Unable to instantiate GlideModule implementation for " + clazz,
			          e);
			    }

			    if (!(module instanceof GlideModule)) {
			      throw new RuntimeException("Expected instanceof GlideModule, but found: " + module);
			    }
			    return (GlideModule) module;
			  }
			}
