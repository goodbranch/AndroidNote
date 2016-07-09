##一个可以无限循环的轮播LoopSwitchView

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

## 具体实现请看代码