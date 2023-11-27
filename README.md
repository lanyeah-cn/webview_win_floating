# webview_win_floating

[visits-count-image]: https://img.shields.io/badge/dynamic/json?label=Visits%20Count&query=value&url=https://api.countapi.xyz/hit/jakky1_webview_win_floating/visits

Flutter webView for Windows.
It's also a plugin that implements the interface of [webview_flutter][1].

![](https://raw.githubusercontent.com/jakky1/webview_win_floating/master/screenshot.jpg)

## Platform Support

This package itself support only Windows.

But use it with [webview_flutter][1], you can write once then support Windows / Android / iOS at the same time.

Android / iOS webview is supported by [webview_flutter][1]

## Features & Limitations

This package place a native Windows WebView2 component on the window, no texture involved.

That's why it called "floating". In Windows, Flutter widgets cannot show on top of the webview.

However, since it is a native WebView2 component, without texture involved, the display speed is the same with native WebView2.

Feature:
- fast display speed  (no texture)
- support fullscreen
- support cross-platform (Windows / Android / iOS)

Limitations: (only in Windows)
- all the Flutter widgets cannot show on top of the webview
- cannot push a new route on top of the webview
- focus switch between webview and flutter widgets (via Tab key) is not support (only in Windows)
- The webview can be put in a scrollable widget, but you may need to assign a ScrollController to the scrollable widget (to enable reposition the webview when scrolling).
- The webview cannot be clipped by Flutter. So if the webview is put in a scrollable, and the webview is outside of the scrollable, the webview is still visible. (However, if the scrollable is filled with the window size, then this issue can be ignored)

Hmm... there are so many limitations.

So try this package only if the limitations mentioned above is not a concern for you, or your app really need cross-platform, or other packages cannot satisfy your requirement (ex. cannot build pass, text blur, low display fps, large ~200MB dll size, security issue when cannot updating the webview core, etc).

## Installation

Add this to your package's `pubspec.yaml` file:

```yaml
dependencies:
  webview_win_floating: ^1.0.0
  webview_flutter: ^4.0.1
```

Or

```yaml
dependencies:
  webview_win_floating:
    git:
      url: https://github.com/jakky1/webview_win_floating.git
      ref: master
  webview_flutter: ^4.0.1
```

# Problem shootting for building fail

If you build fail with this package, and the error message has the keyword "**MSB3073**":

- run "**flutter build .**" in command line in [**Administrator**] mode

# Usage

## register webview first

Before using webview, you should add the following code:
```
import 'package:webview_win_floating/webview.dart';

void main() {
  WindowsWebViewPlatform.registerWith(); // add this line ~~~
  runApp(const MyApp());
}
```

## Use webview now

NOTE: all the interface are supplied by [webview_flutter][1]

```
final controller = WebViewController();

@override
void initState() {
  super.initState();
  controller.setJavaScriptMode(JavaScriptMode.unrestricted);
  controller.loadRequest(Uri.parse("https://www.google.com/"));
}

@override
Widget build(BuildContext context) {
  return WebViewWidget(controller: controller);
}
```

#### enable javascript
don't forgot to add this line if you want to enable javascript:
```
controller.setJavaScriptMode(JavaScriptMode.unrestricted);
```

#### restricted user navigation
For example, to disable the facebook / twitter links in youtube website:
```
controller.setNavigationDelegate(NavigationDelegate(

  onNavigationRequest: (request) {
    return request.url.contains("youtube")
      ? NavigationDecision.navigate
      : NavigationDecision.prevent;
  },

  onPageStarted: (url) => print("onPageStarted: $url"),
  onPageFinished: (url) => print("onPageFinished: $url"),
  onWebResourceError: (error) =>
      print("onWebResourceError: ${error.description}"),
));
```

#### Communication with javascript

Hint: you can rename the name 'myChannelName' in the following code
```
controller.addJavaScriptChannel("myChannelName",
  onMessageReceived: (JavaScriptMessage jmsg) {
    String message = jmsg.message;
    print(message);  // print "This message is from javascript"
  }
);

controller.loadHtmlString(htmlContent);
controller.runJavascript("callByDart(100)");


var htmlContent = '''
<html>
<body>
<script>
function callByDart(int value) {
    console.log("callByDart: " + value);
}
myChannelName.postMessage("This message is from javascript");
</script>
</body>
</html>
''';
```

## controller operations

- controller.loadRequest(uri)
- controller.runJavascript( jsStr )
- controller.runJavaScriptReturningResult( jsStr )  // return javascript function's return value
- controller.reload()
- controller.canGoBack()
- controller.goBack()
- controller.goForward()
- controller.canGoForward()
- controller.currentUrl()
- controller.clearCache()

## dispose controller (cleanup webview instance)

```
controller = null;
// and make sure no any WebViewWidget keep that controller object.
```

After official API interface ``webview_flutter: 4.0.0``, controller is disposed after the WebViewController object is garbage collected.

So the controller object may not be disposed immediately when no any pointer keep the controller object.

## set user data folder

```
String cacheDir = "c:\\test";
var params = WindowsPlatformWebViewControllerCreationParams(
    userDataFolder: cacheDir);
var controller = WebViewController.fromPlatformCreationParams(params);
```

# standalone mode

If your app only runs on Windows, and you want to remove library dependencies as many as possible, you can modify `pubspec.yaml` file:

```yaml
dependencies:
  webview_win_floating: ^1.0.0
  # webview_flutter: ^4.0.1  # mark this line for Windows only app
```

and modify all the following class name in your code:
```
WebViewWidget -> WinWebViewWidget  // add "Win" prefix
WebViewController -> WinWebViewController  // add "Win" prefix
NavigationDelegate  -> WinNavigationDelegate  // add "Win" prefix
```

just only modify class names. All the properties / method are the same with [webview_flutter][1]

There are some Windows-only API:
* controller.openDevTools()
* onHistoryChanged` callback in WinNavigationDelegate
* controller.dispose()



# Example

```
import 'package:flutter/material.dart';
import 'package:webview_win_floating/webview.dart';
import 'package:webview_flutter/webview_flutter.dart';

void main() {
  WindowsWebViewPlatform.registerWith();
  runApp(const MyApp());
}

class MyApp extends StatefulWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  State<MyApp> createState() => _MyAppState();
}

class _MyAppState extends State<MyApp> {
  final controller = WebViewController();

  @override
  void initState() {
    super.initState();
    controller.setJavaScriptMode(JavaScriptMode.unrestricted);
    controller.setBackgroundColor(Colors.cyanAccent);
    controller.setNavigationDelegate(NavigationDelegate(
      onNavigationRequest: (request) {
        if (request.url.startsWith("https://www.google.com")) {
          return NavigationDecision.navigate;
        } else {
          log("prevent user navigate out of google website!");
          return NavigationDecision.prevent;
        }
      },
      onPageStarted: (url) => print("onPageStarted: $url"),
      onPageFinished: (url) => print("onPageFinished: $url"),
      onWebResourceError: (error) =>
          print("onWebResourceError: ${error.description}"),
    ));
    controller.loadRequest(Uri.parse("https://www.google.com/"));
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(
          title: const Text('Windows Webview example app'),
        ),
        body: WebViewWidget(controller: controller),
      ),
    );
  }
}
```
[1]: https://pub.dev/packages/webview_flutter "webview_flutter"