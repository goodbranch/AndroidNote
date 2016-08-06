### 

### `AsyncTaskLoader` 是在Android 3.0加入了，需要兼容更低版本可以使用V4包，`AsyncTaskLoader`只能在`Fragment` 和 `Activity`上使用，并且生命周期跟随`Activity`的`onStart()`，`onStop()`和`onDestory()`，内部线程使用`AsyncTask`和 v4包中使用`ModernAsyncTask`,如果使用的`AsyncTask`默认`Executor` 不会是的`SERIAL_EXECUTOR`（一次只能执行一个任务）。

#### 1.AsyncTaskLoader的优缺点
>
>优点：使用`AsyncTask`执行数据请求，如果当前在Activity关闭后还没有执行完成，那么Activity的就会被持有，因而导致内存泄漏,通常我们的做法是在`Activity`或者`Fragment`中的`onDestory()`方法中做一些数据清理工作，以及引用持有的清理工作，而`AsyncTaskLoader` 有生命周期的管理则会帮我们处理好这些。
>
>
>缺点：我们只能在`Activity`或者`Fragment`中使用，并且不能使用`AsyncTask`的progress
>
#### 2.Loader生命周期管理
>
>`AsyncTaskLoader`最终通过`LoaderManager` 进行生命周期管理，数据分发以及回调，在`Activity`中有一个`LoaderManager` `mLoaderManager`实例，我们在`Activity`或者`Fragment` 中调用`getLoaderManager()`创建出这个实例。

>`mLoaderManager`在Activity中生命周期的管理分别是：

> 1. `onStart()`

	  protected void onStart() {
	        if (DEBUG_LIFECYCLE) Slog.v(TAG, "onStart " + this);
	        mCalled = true;

	        if (!mLoadersStarted) {
	            mLoadersStarted = true;
	            if (mLoaderManager != null) {
	                mLoaderManager.doStart();
	            } else if (!mCheckedForLoaderManager) {
	                mLoaderManager = getLoaderManager("(root)", mLoadersStarted, false);
	            }
	            mCheckedForLoaderManager = true;
	        }

	        getApplication().dispatchActivityStarted(this);
	    }

  2.`performStop()`

	   final void performStop() {
	        mDoReportFullyDrawn = false;
	        if (mLoadersStarted) {
	            mLoadersStarted = false;
	            if (mLoaderManager != null) {
	                if (!mChangingConfigurations) {
	                    mLoaderManager.doStop();
	                } else {
	                    mLoaderManager.doRetain();
	                }
	            }
	        }

3.`performDestroy()`

	 final void performDestroy() {
	        mDestroyed = true;
	        mWindow.destroy();
	        mFragments.dispatchDestroy();
	        onDestroy();
	        if (mLoaderManager != null) {
	            mLoaderManager.doDestroy();
	        }
	        if (mVoiceInteractor != null) {
	            mVoiceInteractor.detachActivity();
	        }
	    }

#### 3.使用`AsyncTaskLoader`
>
>##### 1. 在Activity中实现`LoaderManager.LoaderCallbacks<D>`
>
>	`public class MainActivity extends AppCompatActivity implements LoaderManager.LoaderCallbacks<List<ApplicationInfo>> { `

>
>##### 2. 实现三个方法：
>
>
>


		 @Override
		  public Loader<List<ApplicationInfo>> onCreateLoader(int id, Bundle args) {

		    //args 是getSupportLoaderManager().initLoader传过来的数据
		    Log.e("branch", "onCreateLoader");

		    return new AppListLoader(this);
		  }

		  @Override
		  public void onLoadFinished(Loader<List<ApplicationInfo>> loader, List<ApplicationInfo> data) {


		    Log.e("branch", "onLoadFinished-》 " + data);

		    mListApps = data;

		    mApplistAdapter.notifyDataSetChanged();

		    if (progressDialog != null) {
		      progressDialog.dismiss();
		    }
		  }

		  @Override
		  public void onLoaderReset(Loader<List<ApplicationInfo>> loader) {

		    Log.e("branch", "onLoaderReset");
		    if (progressDialog != null) {
		      progressDialog.dismiss();
		    }

		    mListApps = null;
		    mApplistAdapter.notifyDataSetChanged();

		  }
>
> 在`onCreateLoader` 方法中创建我们的Loader，在Activity的onCreate()或者其他地方调用方法中调用Loader init 方法，如




		  @Override
		  protected void onCreate(Bundle savedInstanceState) {
		    super.onCreate(savedInstanceState);
		    setContentView(R.layout.activity_main);

		    startLoad();
		  }

		 
		  private void startLoad() {

		    progressDialog = new ProgressDialog(this);
		    progressDialog.show();

		    getSupportLoaderManager().initLoader(1, null, this);
		    

		  }


> 当你想放弃上一次的数据是，可以使用`getLoaderManager().restartLoader(0, null, this);`

		public boolean onTextChanged(String newText) {
		    // Called when the action bar search text has changed.  Update
		    // the search filter, and restart the loader to do a new query
		    // with this filter.
		    getLoaderManager().restartLoader(0, null, this);
		    return true;
		}


>这样写完我们的Loader就开始加载数据了

#### 4.AppListLoader的实现
>
> 继承 `AsyncTaskLoader` ,实现以下几个方法
>


		public class AppListLoader extends AsyncTaskLoader<List<ApplicationInfo>> {

		  List<ApplicationInfo> mApps;

		  PackageManager mPm;

		  public AppListLoader(Context context) {
		    super(context);

		    // Retrieve the package manager for later use; note we don't
		    // use 'context' directly but instead the save global application
		    // context returned by getContext().
		    mPm = getContext().getPackageManager();
		  }

		  /**
		   * This is where the bulk of our work is done.  This function is called in a background thread and
		   * should generate a new set of data to be published by the loader.
		   */
		  @Override
		  public List<ApplicationInfo> loadInBackground() {
		    // Retrieve all known applications.
		    List<ApplicationInfo> apps = mPm.getInstalledApplications(
		        PackageManager.GET_UNINSTALLED_PACKAGES | PackageManager.GET_DISABLED_COMPONENTS | PackageManager.GET_META_DATA);
		    if (apps == null) {
		      apps = new ArrayList<ApplicationInfo>();
		    }

		    // Done!
		    return apps;
		  }

		  /**
		   * Called when there is new data to deliver to the client.  The super class will take care of
		   * delivering it; the implementation here just adds a little more logic.
		   */
		  @Override
		  public void deliverResult(List<ApplicationInfo> apps) {
		    if (isReset()) {
		      // An async query came in while the loader is stopped.  We
		      // don't need the result.
		      if (apps != null) {
		        onReleaseResources(apps);
		      }
		    }
		    List<ApplicationInfo> oldApps = mApps;
		    mApps = apps;

		    if (isStarted()) {
		      // If the Loader is currently started, we can immediately
		      // deliver its results.
		      super.deliverResult(apps);
		    }

		    // At this point we can release the resources associated with
		    // 'oldApps' if needed; now that the new result is delivered we
		    // know that it is no longer in use.
		    if (oldApps != null) {
		      onReleaseResources(oldApps);
		    }
		  }

		  /**
		   * Handles a request to start the Loader.
		   */
		  @Override
		  protected void onStartLoading() {
		    if (mApps != null) {
		      // If we currently have a result available, deliver it
		      // immediately.
		      deliverResult(mApps);
		    }

		    if (takeContentChanged() || mApps == null) {
		      // If the data has changed since the last time it was loaded
		      // or is not currently available, start a load.
		      forceLoad();
		    }
		  }

		  /**
		   * Handles a request to stop the Loader.
		   */
		  @Override
		  protected void onStopLoading() {
		    // Attempt to cancel the current load task if possible.
		    cancelLoad();
		  }

		  /**
		   * Handles a request to cancel a load.
		   */
		  @Override
		  public void onCanceled(List<ApplicationInfo> apps) {
		    super.onCanceled(apps);

		    // At this point we can release the resources associated with 'apps'
		    // if needed.
		    onReleaseResources(apps);
		  }

		  /**
		   * Handles a request to completely reset the Loader.
		   */
		  @Override
		  protected void onReset() {
		    super.onReset();

		    // Ensure the loader is stopped
		    onStopLoading();

		    // At this point we can release the resources associated with 'apps'
		    // if needed.
		    if (mApps != null) {
		      onReleaseResources(mApps);
		      mApps = null;
		    }

		  }

		  /**
		   * Helper function to take care of releasing resources associated with an actively loaded data
		   * set.
		   */
		  protected void onReleaseResources(List<ApplicationInfo> apps) {
		    // For a simple List<> there is nothing to do.  For something
		    // like a Cursor, we would close it here.
		  }

		}
    

#### 5.效果图
![效果图](https://raw.githubusercontent.com/goodbranch/AndroidNote/master/note/image/asynctaskloaderdemo.png)

#### 6.demo
>
>[源码](https://github.com/goodbranch/AsyncTaskLoaderDemo.git)
>
>[Apk](https://raw.githubusercontent.com/goodbranch/AndroidNote/master/note/Apk/AsyncTaskLoaderDemo.apk)

 