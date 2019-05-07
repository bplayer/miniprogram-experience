## 微信小程序开发问题总结
* page之间数据交互
	* 页面 A 跳转到页面 B
	* 页面 A 提供 bridge 对象，包含 getData，setData 方法
	* 页面 B 里，拿到页面 A 的 bridge 对象，进行数据交互

* 全屏弹框问题
	* 无法覆盖 map、textarea 等原生组件
	* 使用 cover-view，在 ios10 上不兼容，无法显示
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
			})`  返回多个页面时，会显示中间页面，效果不好
	* 在当前页面使用 `wx.reLaunch` url 会报错，如果前一个页面已经 onshow，url 要相对于前一个页面；否则，要相对本页面

		```
		//在当前页面
		onUnload: function() {
			 wx.reLaunch({
			        url: '../index/index',
			      })
   		}
		```
	* 当前方案，在当前页面 onUnload 里面设置 flag，返回后，在前一个页面调用 `wx.reLaunch`

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
* 避免重复代码
	* wxml 中可能有同样的 view，根据不同条件使用不同的 class 控制样式
		* 应该把条件判断代码写在 class 里面
		* 不应该使用 `wx:if` `wx:elif` 重复写相同的 view 元素

* map 问题
	* 13333
	* 2
	* 3
* setData 太慢问题