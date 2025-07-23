# Android Media3 PlayerView 监听 SurfaceTextureListener

```java
//androidx.media3.ui.PlayerView 设置 app:surface_type="texture_view"
View videoSurfaceView = playerView.getVideoSurfaceView();
if (videoSurfaceView == null) {
    return;
}
if (videoSurfaceView instanceof TextureView) {
  TextureView textureView = (TextureView) videoSurfaceView;
  //textureView.getSurfaceTextureListener()
  textureView.setSurfaceTextureListener(new SurfaceTextureProxy(textureView.getSurfaceTextureListener(), new SurfaceTextureProxy.Listener() {
      @Override
      public void onSurfaceTextureAvailable(SurfaceTexture surface, int width, int height) {
          Log.e(TAG, "onSurfaceTextureAvailable: zfq01");
      }
  }));
}
```