## `WebView`跳过证书验证

- `Android`

```java
// file name: FlutterWebViewClient.java
  private WebViewClient internalCreateWebViewClient() {
    return new WebViewClient() {
      @TargetApi(Build.VERSION_CODES.N)
      @Override
      public boolean shouldOverrideUrlLoading(WebView view, WebResourceRequest request) {
        return FlutterWebViewClient.this.shouldOverrideUrlLoading(view, request);
      }

      @Override
      public void onPageFinished(WebView view, String url) {
        FlutterWebViewClient.this.onPageFinished(view, url);
      }

      @Override
      public void onReceivedSslError(WebView view, SslErrorHandler handler, SslError er) {
        // Ignore SSL certificate errors
        handler.proceed();
      }

      @Override
      public void onUnhandledKeyEvent(WebView view, KeyEvent event) {
        // Deliberately empty. Occasionally the webview will mark events as having failed to be
        // handled even though they were handled. We don't want to propagate those as they're not
        // truly lost.
      }

    };
  }
```

  - `iOS`
  ```objective-c
  # file name FLTWKNavigationDelegate.m
  - (void)webView:(WKWebView*)webView didFinishNavigation:(WKNavigation*)navigation {
  [_methodChannel invokeMethod:@"onPageFinished" arguments:@{@"url" : webView.URL.absoluteString}];
}

- (void)webView:(WKWebView *)webView didReceiveAuthenticationChallenge:(NSURLAuthenticationChallenge *)challenge completionHandler:(void (^)(NSURLSessionAuthChallengeDisposition disposition, NSURLCredential * _Nullable credential))completionHandler{
    NSLog(@"检测到错误证书");

    NSURLCredential *card = [[NSURLCredential alloc]initWithTrust:challenge.protectionSpace.serverTrust];

    completionHandler(NSURLSessionAuthChallengeUseCredential,card);

}
  ```