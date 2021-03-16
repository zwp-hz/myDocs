# H5微信分享配置
## 前言

一般情况下编写H5页面是要考虑微信内置浏览器的兼容性及可行性。而需求方有时会提出页面在微信内部的分享配置，这个时候就要使用微信的js-sdk提供的接口去实现这个功能。

## 配置前&配置后的分享效果
![Aaron Swartz](https://raw.githubusercontent.com/zwp1994/blog-markdown-photos/master/photos/weixin_config1.png)
![Aaron Swartz](https://raw.githubusercontent.com/zwp1994/blog-markdown-photos/master/photos/weixin_config2.png)

## 配置步骤

* 首先你需要一个认证过的企业号，在使用微信JS-SDK对应的JS接口前，需确保已获得使用对应JS接口的权限，可以在[微信官方JS-SDK文档](http://qydev.weixin.qq.com/wiki/index.php?title=%E5%BE%AE%E4%BF%A1JS-SDK%E6%8E%A5%E5%8F%A3#.E4.BD.BF.E7.94.A8.E8.AF.B4.E6.98.8E)中看到分享接口的权限仅限于认证号。

* 有了企业号后，在需要调用JS接口的页面引入如下JS文件，（支持https）：[http://res.wx.qq.com/open/js/jweixin-1.2.0.js](http://res.wx.qq.com/open/js/jweixin-1.2.0.js).

* ### 再通过config接口注入权限验证配置

```
	wx.config({
	    debug: true, // 开启调试模式,调用的所有api的返回值会在客户端alert出来，若要查看传入的参数，可以在pc端打开，参数信息会通过log打出，仅在pc端时才会打印。
	    appId: '', // 必填，企业号的唯一标识，此处填写企业号corpid
	    timestamp: , // 必填，生成签名的时间戳
	    nonceStr: '', // 必填，生成签名的随机串
	    signature: '',// 必填，签名，见附录1
	    jsApiList: [] // 必填，需要使用的JS接口列表，所有JS接口列表见附录2
	});
```
这里的appid我们可以在[微信公众平台](https://mp.weixin.qq.com/)的管理界面中‘开发者中心’ > '配置项'中看到。而签名、生成签名的时间戳、生成签名的随机串一般是由后台生成返回的。因为微信token获取每个月是有次数限定的，为了安全起见还是交由后台生成吧。如果你们的后台人员不了解这一块的话你可以让他看一下这个地址[JS-SDK使用权限签名算法](http://qydev.weixin.qq.com/wiki/index.php?title=%E5%BE%AE%E4%BF%A1JS-SDK%E6%8E%A5%E5%8F%A3#.E9.99.84.E5.BD.951-JS-SDK.E4.BD.BF.E7.94.A8.E6.9D.83.E9.99.90.E7.AD.BE.E5.90.8D.E7.AE.97.E6.B3.95)。

签名算法中用的url必须是调用JS接口页面的完整URL，而这个url必须是企业号中的安全域名。

### 配置js安全域名
打开[微信公众平台](https://mp.weixin.qq.com/)找到**公众号设置**点击**功能设置**再找到**JS接口安全域名**，点击旁边的**设置**。

![Aaron Swartz](https://raw.githubusercontent.com/zwp1994/blog-markdown-photos/master/photos/weixin_config3.png)

下载文件上传到你你添加的域名目录下，确定可以访问到文件后点击**确定**进行保存。*注：每月只能保存3次，请谨慎操作。*

### 贴上自己写的代码

```
	var wxParam = {};
	var wx_share = function(param, url) {
        $.ajax({
            url:  “获取签名信息的接口”,
            type: "GET",
            data: url,
            success: function(res) {
                if (res.code == 0) {
                    var data = res.data;
                    wxParam = param;

                    // 分享配置
                    wx.config({
                        debug: false, // 开启调试模式,调用的所有api的返回值会在客户端alert出来，若要查看传入的参数，可以在pc端打开，参数信息会通过log打出，仅在pc端时才会打印。
                        appId: data.app_id, // 必填，企业号的唯一标识
                        timestamp: data.timestamp, // 必填，生成签名的时间戳
                        nonceStr: data.nonceStr, // 必填，生成签名的随机串
                        signature: data.signature,// 必填，签名，见附录1
                        jsApiList: ['onMenuShareTimeline','onMenuShareAppMessage','onMenuShareQQ','onMenuShareWeibo','onMenuShareQZone'] // 必填，需要使用的JS接口列表，所有JS接口列表见附录2
                    });
                }
            },  
            error : function(res) {}
        })
    }
    
    wx_share({
    	title: "分享标题",
      	desc: "分享描述" ,
      	link: "分享链接，该链接域名必须与当前企业的可信域名一致",
      	imgUrl: "分享图标",
      	type: 分享类型,music、video或link，不填默认为link,
      	dataUrl: 如果type是music或video，则要提供数据链接，默认为空,
      	successFn: function(){
          	// 用户确认分享后执行的回调函数
      	},
      	cancelFn: function(){
          	// 用户取消分享后执行的回调函数
      	}
  	},window.location.href);
   
   	wx.ready(function () {
		//分享到朋友圈
		wx.onMenuShareTimeline({
		    title: wxParam.title,
		    link: wxParam.link,
		    imgUrl: wxParam.imgUrl,
		    success: function () {
		        if (wxParam.successFn) {wxParam.successFn();}
		    }, cancel: function () {
		        if (wxParam.cancelFn) {wxParam.cancelFn();}
		    }
		});
		
		//分享给朋友
		wx.onMenuShareAppMessage({
		    title: wxParam.title,
		    desc: wxParam.desc,
		    link: wxParam.link,
		    imgUrl: wxParam.imgUrl,
		    type: wxParam.type,
		    dataUrl: wxParam.dataUrl,
		    success: function () {
		        if (wxParam.successFn) {wxParam.successFn();}
		    }, cancel: function () {
		        if (wxParam.cancelFn) {wxParam.cancelFn();}
		    }
		});
		
		//分享到QQ
		wx.onMenuShareQQ({
		    title: wxParam.title,
		    desc: wxParam.desc,
		    link: wxParam.link,
		    imgUrl: wxParam.imgUrl,
		    success: function () {
		        if (wxParam.successFn) {wxParam.successFn();}
		    }, cancel: function () {
		        if (wxParam.cancelFn) {wxParam.cancelFn();}
		    }
		});
		
		//分享到腾讯微博
		wx.onMenuShareWeibo({
		    title: wxParam.title,
		    desc: wxParam.desc,
		    link: wxParam.link,
		    imgUrl: wxParam.imgUrl,
		    success: function () {
		        if (wxParam.successFn) {wxParam.successFn();}
		    }, cancel: function () {
		        if (wxParam.cancelFn) {wxParam.cancelFn();}
		    }
		});
		
		//分享到QQ空间
		wx.onMenuShareQZone({
		    title: wxParam.title,
		    desc: wxParam.desc,
		    link: wxParam.link,
		    imgUrl: wxParam.imgUrl,
		    success: function () {
		        if (wxParam.successFn) {wxParam.successFn();}
		    }, cancel: function () {
		        if (wxParam.cancelFn) {wxParam.cancelFn();}
		    }
		});
	}
```