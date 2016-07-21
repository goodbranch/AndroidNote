### 开发过程中涉及到后台调试一直是一个比较麻烦的事，Fiddler却让你很方便的解决了这个问题，下面将讲解Fiddler在开发过程中最常用的使用
--
#### 1. 下载 [Fiddler](http://www.telerik.com/fiddler)

#### 2.设置监听端口号
--
>#### 设置步骤：Tools->FidderOptions->Connections
>
>![设置端口](https://raw.githubusercontent.com/goodbranch/AndroidNote/master/note/image/fiddler_guide_1.png) 
>
>![设置端口](https://raw.githubusercontent.com/goodbranch/AndroidNote/master/note/image/fiddler_guide_2.png)
>
>默认8888，一般不需要修改，除非端口冲突
>
### 3. 设置过滤，避免抓取不必要的请求
--
>#### 可以配置ip,也可以是域名，支持模糊设置，多个设置使用分号隔开
>
>![过滤配置](https://raw.githubusercontent.com/goodbranch/AndroidNote/master/note/image/fiddler_guide_3.png)
>
>配置后如果想要监听其他可以自行修改可以设置不过滤
>
#### 4. 分析返回的数据
>
>返回已经编码过的数据直接在json中可以查看，但是如果不解码是不能修改的，按照指示点击解码区域即可，如果你的操作不需要修改数据，则不用解码也行，json中可以查看已经格式化的数据，也非常方便。
>通常调试在这一步就可以了。
>
>![返回数据](https://raw.githubusercontent.com/goodbranch/AndroidNote/master/note/image/fiddler_guide_4.png)
>
>
#### 5. 修改返回数据
>
>#### 修改数据的前提是第4点已经完成，并且在选项栏`TextView`有数据展示
>
>首先把需要修改返回数据的请求接口拉到`AutoResponder`中，如下图：
>
>![AutoResponder](https://raw.githubusercontent.com/goodbranch/AndroidNote/master/note/image/fiddler_guide_5.png)
>
>
> 然后点击鼠标右键选择`Edit Response`或者选中时按快捷键`F2`进入到修改数据模式，如下图：
>
>
>![修改数据](https://raw.githubusercontent.com/goodbranch/AndroidNote/master/note/image/fiddler_guide_6.png)
>
>
>#### 注意修改后一定要点击保存，并且中文使用的`Unicode` 编码，修改中文也必须转成同样编码
>
>
>最后选中需要在下一次同样请求的接口返回修改后的数据接口前面打勾，如下图：
>
>![修改Response生效](https://raw.githubusercontent.com/goodbranch/AndroidNote/master/note/image/fiddler_guide_7.png)
>
>
>
>
#### 以上就可以满足大部分开发要求。
