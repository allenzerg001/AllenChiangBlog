---
layout: post
title: 用javascript唤起iOS和Android客户端
---

大部分pc到客户端引流都采用二维码扫描的方式，扫码这里就不说了，关键说一下如何从扫码到唤起客户端，暂时仅包括android和ios两大平台。

### 处理逻辑

1. 扫码到一个http页面
2. 页面上写javascript通过useragent判断当前平台android／iphone／ipad；
3. create一个隐藏的frame，设置他的src为应用注册的schema来唤起客户端，比如淘宝主客就是taobao://；
4. 如果用户没有安装客户端，再通过settimeout引导到客户端下载页；
5. 这里要特别注意android和ios一个最大的区别是android唤起的时间要比ios慢许多；比如ios只需要200ms就可以判断唤起，但是android起码要600ms，所以引导到客户端下载页的timeout要有所区别；

### 代码示例


		var noClientCallback = function(){
			window.location = “http://2.taobao.com/m.html”;
		};

		(function(){
			var outTime = (navigator.userAgent.match(/android/i))?600:200;
			var goUrl = “taobao://wnwebview?url=http://www.taobao.com/”;
			var e_frame = document.createElement(‘frame’);
			e_frame.setAttribute(‘id’, ‘J_smartFrame’);
			e_frame.style.cssText = ‘display:none’;
			document.body.appendChild(e_frame);
			e_frame.setAttribute(‘src’,goUrl);
			window.setTimeout(function(){
				if(e_frame){
					document.body.removeChild(e_frame);
				}
				noClientCallback();
			},outTime);
		})();

	
