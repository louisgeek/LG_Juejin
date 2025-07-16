# Android WebView



## 去掉白色背景（让背景透明）
``` java
mWebView.setBackgroundResource(android.R.color.transparent); //无效
mWebView.setBackground(new ColorDrawable(Color.TRANSPARENT)); //无效
mWebView.setBackgroundColor(Color.TRANSPARENT); //有效
```

```java
public void setBackgroundColor(int color) {
    //mWebView.setBackgroundResource(android.R.color.transparent); //无效
    //mWebView.setBackground(new ColorDrawable(Color.TRANSPARENT)); //无效
    mWebView.setBackgroundColor(color); //重要
    //mWebView.setBackgroundColor(color); //二次调用未能验证，据说部分机型需要重复调用才生效
    if (color == Color.TRANSPARENT) {
        mWebView.setBackgroundResource(android.R.color.transparent); //附带一下
        //
        Drawable drawable = mWebView.getBackground();
        if (drawable != null) {
            drawable.setAlpha(0); //未能验证，据说影响部分机型
        }
        mWebView.setLayerType(View.LAYER_TYPE_SOFTWARE, null); //关闭硬件加速
    }
}
```