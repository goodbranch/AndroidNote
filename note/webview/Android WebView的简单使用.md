### 使用WebView开发web App

通常我们有一些页面需要使用web，那么在`Activity`中包含一个`WebView`，这样你就可以显示在线内容了。

### 增加一个WebView到应用中

 在xml 布局文件中加入

	  <WebView
	      android:id="@+id/webview"
	      android:layout_width="match_parent"
	      android:layout_height="match_parent"/>

使用loadurl加载页面到WebView，如下：

	WebView myWebView = (WebView) findViewById(R.id.webview);
	myWebView.loadUrl("https://www.baidu.com/");
    //mWebView.loadData(); 直接加载网页内容

在能工作之前，一定要让你的应用有网络权限，在manifest file中配置如下：

	<manifest ... >
	    <uses-permission android:name="android.permission.INTERNET" />
	    ...
	</manifest>

### 配置WebView，参考如下：

		WebSettings webSettings = mWebView.getSettings();
	    webSettings.setAppCacheEnabled(true);  //是否可以缓存数据到本地，默认为false，设置后要跟setAppCachePath（）结合使用
	    webSettings.setAppCachePath(FileManage.getInstance(mWebView.getContext()).getWebViewCachePath());
	    webSettings.setAllowFileAccess(true); //是否可以访问文件
	    webSettings.setGeolocationEnabled(true); //是否可以定位，默认true 同时需要{@link android.Manifest.permission#ACCESS_COARSE_LOCATION},{@link android.Manifest.permission#ACCESS_FINE_LOCATION};权限
	    webSettings.setJavaScriptEnabled(true);  //是否支持js执行
	    webSettings.setSupportZoom(true);


### 网页页面进度获取

在加载网页时我们常常需要设置网页加载进度，则可以通过 `WebView`设置 `mWebView.setWebChromeClient()`实现`onProgressChanged`方法。

	 public class MyWebChromeClient extends WebChromeClient {

	    @Override
	    public void onProgressChanged(WebView view, int newProgress) {
	      super.onProgressChanged(view, newProgress);
	      if (mLoadingView != null) {
	        mLoadingView.setCurrentProgress(newProgress);
	      }
	    }
	}


处理文件通过网页上传文件

	 @Override
	    public boolean onShowFileChooser(WebView webView, ValueCal是文件上传的回调，在代码中我们打开图片或者文件选择，选择文件后在`onActivityResult`中回调获得文件，当然这里也可以使用别的方式。

	    	lback<Uri[]> filePathCallback, FileChooserParams fileChooserParams) {

	      mValueCallback = filePathCallback;

	      Intent intent = new Intent(Intent.ACTION_GET_CONTENT);
	      if (mChooserImage) {
	        intent.setType("image/*");
	      } else {
	        //文件选择
	        intent.setType("file/*");
	      }
	      startActivityForResult(Intent.createChooser(intent, mChooserTitle), FILE_REQUESTCODE);

	      return true;
	    }

`filePathCallback`是文件上传的回调，在代码中我们打开图片或者文件选择，选择文件后在`onActivityResult`中回调获得文件，当然这里也可以使用别的方式。在`onActivityResult`中我们如下处理:


	  @Override
	  protected void onActivityResult(int requestCode, int resultCode, Intent data) {
	    super.onActivityResult(requestCode, resultCode, data);

	    if (resultCode == RESULT_OK) {

	      if (requestCode == FILE_REQUESTCODE) {
	        if (data != null && data.getData() != null && mValueCallback != null) {
	          Uri[] uri = new Uri[1];
	          uri[0] = data.getData();
	          mValueCallback.onReceiveValue(uri);
	        }
	      }
	    }

	  }



### 处理页面链接点击事件

有时我们通过网页链接点击触发跳转，网页认证，网页加载和完成等，可以给`WebView` 设置 `setWebViewClient()`，如下：

	myWebView.setWebViewClient(new WebViewClient());


* 当你点击链接是就能被客户端捕获，如下：

		private class MyWebViewClient extends WebViewClient {
		    @Override
		    public boolean shouldOverrideUrlLoading(WebView view, String url) {
		        if (Uri.parse(url).getHost().equals("https://www.baidu.com/")) {
		            // This is my web site, so do not override; let my WebView load the page
		            return false;
		        }
		        // Otherwise, the link is not for a page on my site, so launch another Activity that handles URLs
		        Intent intent = new Intent(Intent.ACTION_VIEW, Uri.parse(url));
		        startActivity(intent);
		        return true;
		    }
		}

 * 如果你同时需要做登入认证，则如下：

		   @Override
		    public void onReceivedHttpAuthRequest(WebView view, HttpAuthHandler handler, String host, String realm) {
		      if (mNeedAuth) {
		        handler.proceed(..., null);
		      } else {
		        super.onReceivedHttpAuthRequest(view, handler, host, realm);
		      }
		    }


* WebViewClient 内还有网页开始加载和加载完成的回调，如下：

		    @Override
		    public void onPageFinished(WebView view, String url) {
		      super.onPageFinished(view, url);
		    }

		    @Override
		    public void onPageStarted(WebView view, String url, Bitmap favicon) {
		      super.onPageStarted(view, url, favicon);
		    }


### 页面导航

`WebView` 会自动记录加载过的页面，你可以通过`goBack()`和`goForward()`前进或后台页面,例如在Activity中捕获返回按键：

	@Override
	public boolean onKeyDown(int keyCode, KeyEvent event) {
	    // Check if the key event was the Back button and if there's history
	    if ((keyCode == KeyEvent.KEYCODE_BACK) && myWebView.canGoBack()) {
	        myWebView.goBack();
	        return true;
	    }
	    // If it wasn't the Back key or there's no web page history, bubble up to the default
	    // system behavior (probably exit the activity)
	    return super.onKeyDown(keyCode, event);
	}


### 其他需要注意的

* 把WebView加入到Activity的生命周期，如果不加入当正在播放视频时，网页不可见时不会暂停播放，所以加上生命周期，如下


		  @Override
		  protected void onPause() {
		    super.onPause();
		    if(mWebView!=null){
		      mWebView.onPause();
		    }
		  }


		  @Override
		  protected void onResume() {
		    super.onResume();
		    if(mWebView!=null){
		      mWebView.onResume();
		    }
		  }


* 销毁,WebView比较占用内存，关闭后应尽可能销毁所有，或把WebView的Activity放在其他进程中，这样关闭后也能立即销毁。


	    removeAllViews();
	    destroyDrawingCache();
	    setWebChromeClient(null);
	    setWebViewClient(null);
	    stopLoading();
	    clearFormData();
	    clearHistory();
	    clearSslPreferences();
	    clearDisappearingChildren();
	    clearMatches();

	    if (Build.VERSION.SDK_INT < 19) {// Build.VERSION_CODES.KITKAT
	      freeMemory();
	    }






