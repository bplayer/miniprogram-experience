## 微信小程序开发问题总结
* request 合法域名设置
	* 第三方域名需要添加
		* https://restapi.amap.com
* page之间数据交互
	* 页面 A 跳转到页面 B
	* 页面 A 提供 bridge 对象，包含 getData，setData 方法
	* 页面 B 里，拿到页面 A 的 bridge 对象，进行数据交互
*  page 里面 data 数据太多太乱，应该整理分类

	* 页面相关的数据可以放在 viewModel 对象里面
	* 网络相关的数据可以放在 apiModel 里面

		```
		...
		viewModel:{},
		apiModel: {
			responseModel: {},
			requestModel: {}
		},
		...
		
		```
		
* 全屏弹框问题
	* 无法覆盖 map、textarea 等原生组件
	* 使用 cover-view，在 ios10 上不兼容，无法显示
	* cover-view 圆角设置只能设置所有四个角，无法单独设置其中几个圆角
	* 最终方案，弹框改为跳转到新页面

* 导航进入 n 个页面之后，如何返回首页
	* 无法监听处理左上角返回按钮事件
	* 官方文档 [返回的页面数，如果 delta 大于现有页面数，则返回到首页。](https://developers.weixin.qq.com/miniprogram/dev/api/wx.navigateBack.html) 实际在真机上导致退出当前小程序
	
		```
		//在当前页面
		onUnload: function() {
			wx.navigateBack({
			  delta: 20
			})
		}
		```
	* 使用 `wx.navigateBack({
			  delta: 20
			})`  返回多个页面时
		* 返回之后，页面点击无反应，可能是页面层级太多，影响点击事件
		* 会显示中间页面，效果不好
	* 在当前页面使用 `wx.reLaunch` url 会报错，如果前一个页面已经 onshow，url 要相对于前一个页面；否则，要相对本页面

		```
		//在当前页面
		onUnload: function() {
			 wx.reLaunch({
			        url: '../index/index',
			      })
   		}
		```
	* 在当前页面 onUnload 里面设置 flag，返回后，在前一个页面调用 `wx.reLaunch`。问题，导航页面层级太多，后面页面点击无响应

		```
		  //在前一个页面
		  onShow: function() {
		    if (xxx) {
		      wx.reLaunch({
		        url: '../index/index',
		      })
	   		 }
	 	   }
	
		```

	* 当前方案，改变进入当前页面方式。通过首页进入当前页面，这样返回时就直接到首页了。

		```
		  //前一个页面通过首页跳转到后面页面
		  wx.reLaunch({
	        url: '../index/index?redirectTo=my-zoe',
	      })
	
		```
	
* 避免重复代码
	* wxml 中可能有同样的 view，根据不同条件使用不同的 class 控制样式
		* 应该把条件判断代码写在 class 里面
		* 不应该使用 `wx:if` `wx:elif` 重复写相同的 view 元素

* input 判断是否为空出错问题

	```
	//当 value 是 number 类型时，转为 String 类型
	input.text = String(value)
	
	//字符串判断逻辑
	if (input.text && input.text.length) {
	  return true
	}
	
	```


* map 问题
	* map的默认显示区域可以用 include-points 设置，但建议取值为最大最小的坐标点，避免 setdata 是数据过大导致加载缓慢
	* 设置地图路径画线数据（ polyline ）：由于小程序 setdata 设置一次最大为1M，路径长往往会超出，而且页面加载会极度缓慢，建议数据分开设置
	
		```
		for (var i = 0; i < data.points.length; i++) {
	          var key = "polyline[" + i + "]"
	          var obj = {}
	          obj[key] = {
	            points: data.points[i],
	            arrowLine: true,
	            color: "#4693fd",
	            width: 5,
	          }
	          that.setData(obj)
	        }
		```
	
	* 设置地图 marker 问题
		* 当设置的 marker 需要显示 callout 属性信息时，多个 callout 的id必须不同，否则在 Android 设备上所有 marker 只会显示最后一条 marker 的 callout 信息
* setData 太慢问题
	* 【微信小程序】性能优化<https://juejin.im/post/5b496d5d5188251a90187635>

* showToast 不显示问题
	* 当同时调用 showToast 和 showLoading 的时候，showToast 提示框会不显示
 
* textarea 层级过高，导致内容穿透
	* 原因：textarea 是原生控件。原生组件的层级是最高的，所以页面中的其他组件无论设置 z-index 为多少，都无法盖在原生组件上
	* 解决方案：textarea 输入完成时，隐藏该 textarea，将输入内容显示在一个 text 文本框；当点击 text 时，隐藏 text，显示 textarea,并获取焦点