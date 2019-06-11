WebViewJavascriptBridge
=======================

[![Circle CI](https://img.shields.io/circleci/project/github/marcuswestin/WebViewJavascriptBridge.svg)](https://circleci.com/gh/marcuswestin/WebViewJavascriptBridge)

作为一个在IOS/OSX 下的WKWebViews, UIWebViews & WebViews 里Obj-C 和 JS 之间发送效益的桥；

一、迁移指南
---------------

从v5.0.x升级到6.0.x时，您必须更新 `setupWebViewJavascriptBridge` javascript 片段。 请参阅 https://github.com/marcuswestin/WebViewJavascriptBridge#usage 第4部分).

二、谁需要使用 WebViewJavascriptBridge?
---------------------------------
WebViewJavascriptBridge被一系列公司和项目使用。这是一个小而不完整的样本列表：

- [Facebook Messenger](https://www.facebook.com/mobile/messenger)
- [Facebook Paper](https://facebook.com/paper)
- [Yardsale](http://www.getyardsale.com/)
- [EverTrue](http://www.evertrue.com/)
- [Game Insight](http://www.game-insight.com/)
- [Sush.io](http://www.sush.io)
- [Imbed](http://imbed.github.io/)
- [CareZone](https://carezone.com)
- [Hemlig](http://www.hemlig.co)
- [Altralogica](http://www.altralogica.it)
- [鼎盛中华](https://itunes.apple.com/us/app/ding-sheng-zhong-hua/id537273940?mt=8)
- [FRIL](https://fril.jp)
- [留白·WHITE](http://liubaiapp.com)
- [BrowZine](http://thirdiron.com/browzine/)
- ... & many more!

三、安装 (iOS & OSX)
------------------------

### 使用 CocoaPods 安装
Add this to your [podfile](https://guides.cocoapods.org/using/getting-started.html) and run `pod install` to install:

```ruby
pod 'WebViewJavascriptBridge', '~> 6.0'
```

### 手动安装

Drag the `WebViewJavascriptBridge` folder into your project.

In the dialog that appears, uncheck "Copy items into destination group's folder" and select "Create groups for any folders".

四、示例
--------

See the `Example Apps/` folder. Open either the iOS or OSX project and hit run to see it in action.

To use a WebViewJavascriptBridge in your own project:

五、用法
-----

1) Import the header file and declare an ivar property:

```objc
#import "WebViewJavascriptBridge.h"
```

...

```objc
@property WebViewJavascriptBridge* bridge;
```

2) Instantiate WebViewJavascriptBridge with a WKWebView, UIWebView (iOS) or WebView (OSX):

```objc
self.bridge = [WebViewJavascriptBridge bridgeForWebView:webView];
```

3) Register a handler in ObjC, and call a JS handler:

```objc
[self.bridge registerHandler:@"ObjC Echo" handler:^(id data, WVJBResponseCallback responseCallback) {
	NSLog(@"ObjC Echo called with: %@", data);
	responseCallback(data);
}];
[self.bridge callHandler:@"JS Echo" data:nil responseCallback:^(id responseData) {
	NSLog(@"ObjC received response: %@", responseData);
}];
```

4) Copy and paste `setupWebViewJavascriptBridge` into your JS:
	
```javascript
function setupWebViewJavascriptBridge(callback) {
	if (window.WebViewJavascriptBridge) { return callback(WebViewJavascriptBridge); }
	if (window.WVJBCallbacks) { return window.WVJBCallbacks.push(callback); }
	window.WVJBCallbacks = [callback];
	var WVJBIframe = document.createElement('iframe');
	WVJBIframe.style.display = 'none';
	WVJBIframe.src = 'https://__bridge_loaded__';
	document.documentElement.appendChild(WVJBIframe);
	setTimeout(function() { document.documentElement.removeChild(WVJBIframe) }, 0)
}
```

5) Finally, call `setupWebViewJavascriptBridge` and then use the bridge to register handlers and call ObjC handlers:

```javascript
setupWebViewJavascriptBridge(function(bridge) {
	
	/* Initialize your app here */

	bridge.registerHandler('JS Echo', function(data, responseCallback) {
		console.log("JS Echo called with:", data)
		responseCallback(data)
	})
	bridge.callHandler('ObjC Echo', {'key':'value'}, function responseCallback(responseData) {
		console.log("JS received response:", responseData)
	})
})
```

六、自动参数计算 (ARC)
----------------------------------
This library relies on ARC, so if you use ARC in you project, all works fine.
But if your project have no ARC support, be sure to do next steps:

1) In your Xcode project open project settings -> 'Build Phases'

2) Expand 'Compile Sources' header and find all *.m files which are belongs to this library. Make attention on the 'Compiler Flags' in front of each source file in this list

3) For each file add '-fobjc-arc' flag

Now all WVJB files will be compiled with ARC support.

七、Contributors & Forks
--------------------
Contributors: https://github.com/marcuswestin/WebViewJavascriptBridge/graphs/contributors

Forks: https://github.com/marcuswestin/WebViewJavascriptBridge/network/members

八、API 参考
-------------

## 8.1  ObjC API

##### `[WebViewJavascriptBridge bridgeForWebView:(WKWebVIew/UIWebView/WebView*)webview`

Create a javascript bridge for the given web view.

Example:

```objc	
[WebViewJavascriptBridge bridgeForWebView:webView];
```

##### `[bridge registerHandler:(NSString*)handlerName handler:(WVJBHandler)handler]`

Register a handler called `handlerName`. The javascript can then call this handler with `WebViewJavascriptBridge.callHandler("handlerName")`.

Example:

```objc
[self.bridge registerHandler:@"getScreenHeight" handler:^(id data, WVJBResponseCallback responseCallback) {
	responseCallback([NSNumber numberWithInt:[UIScreen mainScreen].bounds.size.height]);
}];
[self.bridge registerHandler:@"log" handler:^(id data, WVJBResponseCallback responseCallback) {
	NSLog(@"Log: %@", data);
}];

```

##### `[bridge callHandler:(NSString*)handlerName data:(id)data]`
##### `[bridge callHandler:(NSString*)handlerName data:(id)data responseCallback:(WVJBResponseCallback)callback]`

Call the javascript handler called `handlerName`. If a `responseCallback` block is given the javascript handler can respond.

Example:

```objc
[self.bridge callHandler:@"showAlert" data:@"Hi from ObjC to JS!"];
[self.bridge callHandler:@"getCurrentPageUrl" data:nil responseCallback:^(id responseData) {
	NSLog(@"Current UIWebView page URL is: %@", responseData);
}];
```

#### `[bridge setWebViewDelegate:(id)webViewDelegate]`

Optionally, set a `WKNavigationDelegate/UIWebViewDelegate` if you need to respond to the [web view's lifecycle events](https://developer.apple.com/reference/uikit/uiwebviewdelegate).

##### `[bridge disableJavscriptAlertBoxSafetyTimeout]`

UNSAFE. Speed up bridge message passing by disabling the setTimeout safety check. It is only safe to disable this safety check if you do not call any of the javascript popup box functions (alert, confirm, and prompt). If you call any of these functions from the bridged javascript code, the app will hang.

Example:

	[self.bridge disableJavscriptAlertBoxSafetyTimeout];



## 8.2  Javascript API

### 8.2.1  `bridge.registerHandler("handlerName", function(responseData) { ... })`

注册一个名为 `handlerName` 的处理回调。 然后，Objc 可以通过使用 `[bridge callHandler:"handlerName" data:@"Foo"]` 和  `[bridge callHandler:"handlerName" data:@"Foo" responseCallback:^(id responseData) { ... }]`调用这个回调。

实例:
```javascript
bridge.registerHandler("showAlert", function(data) { alert(data) })
bridge.registerHandler("getCurrentPageUrl", function(data, responseCallback) {
	responseCallback(document.location.toString())
})
```


### 8.2.2  `bridge.callHandler("handlerName", data)`
### 8.2.3  `bridge.callHandler("handlerName", data, function responseCallback(responseData) { ... })`

调用一个名为 `handlerName` 的ObjC 处理函数。 如果给出六 `responseCallback` 函数， 则 ObjC 处理程序可以相应

实例:
```javascript
bridge.callHandler("Log", "Foo")
bridge.callHandler("getScreenHeight", null, function(response) {
	alert('Screen height:' + response)
})
```


### 8.2.4  `bridge.disableJavscriptAlertBoxSafetyTimeout()`

在 ObjC 中，调用 `bridge.disableJavscriptAlertBoxSafetyTimeout()` 与调用 `[bridge disableJavscriptAlertBoxSafetyTimeout];`  具有相同效果。

实例:
```javascript
bridge.disableJavscriptAlertBoxSafetyTimeout()
```
