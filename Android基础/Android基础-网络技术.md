# Android基础-网络技术

~~~java
package com.example.webviewtest;

import androidx.appcompat.app.AppCompatActivity;

import android.os.Bundle;
import android.webkit.WebView;
import android.webkit.WebViewClient;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        WebView webView = (WebView)findViewById(R.id.web_viwe);
        webView.getSettings().setJavaScriptEnabled(true); //设置支持JavaScript脚本
        webView.setWebViewClient(new WebViewClient());//当一个网页要打开另一个网页的时候，我们需要目标网页仍然在该webView里面显示，而不是打开浏览器。
        webView.loadUrl("http://www.baidu.com");
    }
}
~~~

在老版本里面，这里应该是可以直接运行的，但是新版本就不可以，因为网络连接权限问题，网页不能够启动起来，会显示网络连接失败。

~~~xml
    <uses-permission android:name="android.permission.INTERNET"/>
~~~

权限如上。

