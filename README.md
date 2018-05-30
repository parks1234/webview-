# webview-


    @BindView(R.id.webview)
    WebView wv;
 
    @BindView(R.id.myProgressBar)
    ProgressBar myProgressBar;



    @Override
    public void initData() {
    //        wv.getSettings().setTextSize(WebSettings.TextSize.NORMAL);
//        webSettings.setDefaultFixedFontSize(13);//字体
//        wv.getSettings().setPluginsEnabled(true);//可以使用插件
        //    webSettings.setBuiltInZoomControls(true); //设置内置的缩放控件。若为false，则该WebView不可缩放
        //    webSettings.setDisplayZoomControls(false); //隐藏原生的缩放控件
        //   webSettings.setJavaScriptCanOpenWindowsAutomatically(true); //支持通过JS打开新窗口 
        // webSettings.setLoadsImagesAutomatically(true); //支持自动加载图片
        // webSettings.setDefaultTextEncodingName("utf-8");//设置编码格式
        //webSettings.setDatabaseEnabled(true);   //开启 database storage API 功能
        wv.getSettings().setDomStorageEnabled(true);// 开启 DOM storage API 功能
        //  wv.getSettings().setAppCacheMaxSize(1024*1024*8);//最大缓存
        String appCachePath = getApplicationContext().getCacheDir().getAbsolutePath();
        wv.getSettings().setAppCachePath(appCachePath);//cache路径
        wv.getSettings().setAppCacheEnabled(true);//开启 Application Caches 功能
        wv.getSettings().setCacheMode(WebSettings.LOAD_DEFAULT);//缓存
        wv.getSettings().setSupportZoom(true);//支持缩放，默认为true
        wv.getSettings().setSaveFormData(false);
        wv.getSettings().setJavaScriptEnabled(true);//支持Javascript
        wv.getSettings().setPluginState(WebSettings.PluginState.ON);
        wv.getSettings().setBlockNetworkImage(false);
        wv.getSettings().setJavaScriptCanOpenWindowsAutomatically(true);
        wv.getSettings().setAllowFileAccess(true);//设置可以访问文件 
        wv.getSettings().setDefaultTextEncodingName("UTF-8");
        wv.getSettings().setLoadWithOverviewMode(true);//将图片调整到适合webview的大小 
        
        wv.getSettings().setUseWideViewPort(true);//将图片调整到适合webview的大小 
        
        wv.setVisibility(View.VISIBLE);
        wv.setDownloadListener(new MyWebViewDownLoadListener());
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP)
            wv.getSettings().setMixedContentMode(WebSettings.MIXED_CONTENT_ALWAYS_ALLOW);
        wv.getSettings().setUserAgentString(wv.getSettings().getUserAgentString() + " /(android)" + Build.MODEL + "/" + Build.MANUFACTURER + "/" + Build.VERSION.SDK_INT);

        wv.getSettings().setTextZoom(100);

        WebChromeClient wvcc = new WebChromeClient() {
            @Override
            public void onProgressChanged(WebView view, int newProgress) {
                if (newProgress == 100) {
                    myProgressBar.setVisibility(View.GONE);
                } else {
                    if (View.GONE == myProgressBar.getVisibility()) {
                        myProgressBar.setVisibility(View.VISIBLE);
                    }
                    myProgressBar.setProgress(newProgress);
                }
                super.onProgressChanged(view, newProgress);
            }

            @Override
            public void onReceivedTitle(WebView view, String title) {
                super.onReceivedTitle(view, title);
//                Log.d("ANDROID_LAB", "TITLE=" + title);
                baseTopText1.setText(title);
            }

        };
        // 设置setWebChromeClient对象  
        wv.setWebChromeClient(wvcc);
        String string = getIntent().getExtras().getString("url");
        if (string != null && !TextUtil.isEmpty(string)) {
            if (!string.startsWith("http")) {
                string = "http://" + string;
            }
        }
        shareurl = string;
        wv.loadUrl(string);

        wv.setWebViewClient(new WebViewClient() {
            @Override
            public boolean shouldOverrideUrlLoading(WebView view, WebResourceRequest request) {
                String url = request.toString();
                if (url == null) return false;
                FileOutUtils.w("webview", url);
                try {
                    if (url.startsWith("http:") || url.startsWith("https:")) {
                        view.loadUrl(url);
                        return true;
                    } else {
                        Intent intent = new Intent(Intent.ACTION_VIEW, Uri.parse(url));
                        startActivity(intent);
                        return true;
                    }
                } catch (Exception e) { //防止crash (如果手机上没有安装处理某个scheme开头的url的APP, 会导致crash)
                    return false;
                }
            }

            @Override
            public boolean shouldOverrideUrlLoading(WebView view, String url) { // 重写此方法表明点击网页里面的链接还是在当前的webview里跳转，不跳到浏览器那边
                return super.shouldOverrideUrlLoading(view, url);

            }

            @Override
            public void onReceivedSslError(WebView view, SslErrorHandler handler, SslError error) {
                handler.proceed();
            }
         


        });
    }
    // 如果不做任何处理，浏览网页，点击系统“Back”键，整个Browser会调用finish()而结束自身，  
    // 如果希望浏览的网 页回退而不是推出浏览器，需要在当前Activity中处理并消费掉该Back事件。   
    @Override
    public boolean onKeyDown(int keyCode, KeyEvent event) {

        if ((keyCode == KeyEvent.KEYCODE_BACK)) {
            if (wv.canGoBack()) {
                wv.goBack();
                return true;
            } else
                finish();

        }

        return super.onKeyDown(keyCode, event);

    }

    private class MyWebViewDownLoadListener implements DownloadListener {

        @Override

        public void onDownloadStart(String url, String userAgent, String contentDisposition, String mimetype,

                                    long contentLength) {

//            Log.i("tag", "url="+url);
//
//            Log.i("tag", "userAgent="+userAgent);
//
//            Log.i("tag", "contentDisposition="+contentDisposition);
//
//            Log.i("tag", "mimetype="+mimetype);
//
//            Log.i("tag", "contentLength="+contentLength);

            Uri uri = Uri.parse(url);

            Intent intent = new Intent(Intent.ACTION_VIEW, uri);

            startActivity(intent);

        }

    }



    @Override
    protected void onPause() {
        super.onPause();
        //暂停WebView在后台的所有活动
        wv.onPause();
//暂停WebView在后台的JS活动
        wv.pauseTimers();
    }

    @Override
    protected void onResume() {
        super.onResume();
        wv.onResume();
        wv.resumeTimers();
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();

        wv.destroy();

    }
}
