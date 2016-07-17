##一个可以无限循环的轮播LoopSwitchView

## 效果
![image](Screenshot_2016-07-17-11-18-21.png")
![image](loopswitch.gif")
    
## 实现目标
>* 可以手动左右滚动
>* 可以自动轮播
>* 自定义性强

## 实现思路
>首先想到的是使用ViewPager，ViewPager本身在处理页面左右滚动上已经很好了，在ViewPager的基础上加上Handler让他可以自动滚动。
>### 遇到的问题
>* 不能循环滚动，到ViewPager的最后一个再往后不能滚动，在第一个再往前也是不行。
>
>### 解决思路
>在0位置上显示第count-1 的item，在count上显示第0个item，也就是需要额外再ViewPager的Adapter getCount（）方法中增加 2，实现过程就变成滑动到0后就立即是viewPager.setCurrentItem(count-1),同样滑动到count 就使viewPager.setCurrentItem(0)，也就通过这样无缝对接轮播。
>
>### 在nexus 6p上出现手动滑动卡顿问题
>上面思路在大部分手机上都有很好的体验，但是在部分手机上会出现卡顿，出现卡顿的原因就是在viewPager.setCurrentItem()，那么我们可不可以避免呢，显然使用ViewPager是不能避免这种情况的，那么现在我们再想想我们原来只是增加了额外2个item，现在我增加100个呢，用户滑动到临界点的概率就大大降低了，我们就使用这种方式。
>

## 部分实现代码：
> ####1. 首先看下`AutoLoopSwitchBaseView.java` 中布局了`ViewPager` ，在这个View中我们定义一个`Handler` 发送循环消息。
> 在`LoopHandler`中定义了三种消息类型
```
 //请求更新显示的View。
    private static final int MSG_UPDATE = 1;
    //请求暂停轮播。
    public static final int MSG_STOP = 2;
    //请求恢复轮播。
    public static final int MSG_REGAIN = 3;
```
在`    public void handleMessage(Message msg)`中处理这三种消息

	@Override
   	 public void handleMessage(Message msg) {
      	super.handleMessage(msg);
      if (mView == null || mView.mHandler == null || mView.mPagerAdapter == null || mView.mIsDragging) {
        return;
      }

      Log.e("ryze", "stop: " + mIsStop);

      switch (msg.what) {
        case MSG_UPDATE:
          if (mIsStop || hasMessages(MSG_UPDATE)) {
            return;
          }
          if (mView.mPagerAdapter.getCount() > 1) {
            mView.mCurrentItem++;
            mView.mCurrentItem %= mView.mPagerAdapter.getCount();
            mView.mViewPager.setCurrentItem(mView.mCurrentItem, true);
            sendEmptyMessageDelayed(MSG_UPDATE, mView.getDurtion());
          }
          break;
        case MSG_STOP:
          if (hasMessages(MSG_UPDATE)) {
            removeMessages(MSG_UPDATE);
          }
          mIsStop = true;
          Log.e("ryze", "stop: MSG_STOP " + mIsStop);
          break;
        case MSG_REGAIN:
          if (hasMessages(MSG_UPDATE)) {
            removeMessages(MSG_UPDATE);
          }
          sendEmptyMessageDelayed(MSG_UPDATE, mView.getDurtion());
          mIsStop = false;
          Log.e("ryze", "stop: MSG_REGAIN " + mIsStop);
          break;
      } 
      
      
 >#### 2.来看看`Adapter`的实现，在前面讲到过大概的实现方式会在前后各增加一个，同时再增加当前 100 的倍数关系的数量 ，那么`getCount（）` 的数量也就会是实际数量*100+2
 
 ```
   public static final int VIEWPAGER_RADIX = 100;

    @Override
  public final int getCount() {
    if (getDataCount() > 1) {
      //如果轮播个数大于1个，那么需要轮播，增加基数，同时在首尾加上一个，
      return getDataCount() * VIEWPAGER_RADIX + 2;
    } else {
      return getDataCount();
    }
  }
 ```
 
 >#### 3. 数据和自动滚动的消息已经有了，同时前后增加的数据也增加了，那么是怎么实现循环播放的连接呢？
监听`ViewPager`的`OnPageChangeListener` 在` public void onPageScrollStateChanged(int i)` 


	 @Override
	 public void onPageScrollStateChanged(int i) {

    if (i == ViewPager.SCROLL_STATE_DRAGGING) {
      mIsDragging = true;
    } else if (i == ViewPager.SCROLL_STATE_IDLE) {
      if (mViewPager.getCurrentItem() == 0) {
        isLoopSwitch = true;
        mViewPager.setCurrentItem(mPagerAdapter.getCount() - 2, false);
      } else if (mViewPager.getCurrentItem() == mPagerAdapter.getCount() - 1) {
        isLoopSwitch = true;
        mViewPager.setCurrentItem(1, false);
      }
      mCurrentItem = mViewPager.getCurrentItem();

      if (mIsDragging && mHandler != null) {
        //如果从dragging状态到不是mIsDragging
        mHandler.sendEmptyMessageDelayed(LoopHandler.MSG_UPDATE, getDurtion());
      }

      mIsDragging = false;

      Log.e("ryze", "onPageScrollStateChanged  " + i);
     }
    }
    
 
 
###[demo](https://github.com/goodbranch/LoopSwitch)