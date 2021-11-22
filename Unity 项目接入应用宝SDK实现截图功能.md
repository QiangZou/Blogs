
# Unity项目接入应用宝SDK实现截图功能


----------


## 问题由来

 - 点击应用宝悬浮窗

![](https://github.com/QiangZou/Blogs/blob/master/Images/TIM%E5%9B%BE%E7%89%8720190108200043.png?raw=true)

- 如图所示 左下角有一个截图按钮


## 需要解决那些问题


- 截图信息需要由游戏引擎提供

- SDK获取截图信息为同步 但是Unity引擎没有提供同步接口

- 如何防止测试同学和智障不停的点击截图按钮









## 点击截图按钮程序流程

- SDK调用caputureImage方法获取截图信息

- 在caputureImage方法中通知Unity截图

- 在caputureImage方法中等待截图信息

- Unity截图完成后发送给安卓层

- 返回数据给SDK

- 实现一个缓存5秒截图信息功能



## unity代码

- 安卓层通知Unity截图接口

```
public void CaputureImage()
{
    StartCoroutine(Caputure());
}
```
- Unity获取截图信息返回给安卓层

```
IEnumerator Caputure()
{
	//等待当前帧渲染完成
    yield return new WaitForEndOfFrame();

    // 先创建一个的空纹理，大小可根据实现需要来设置
    Texture2D screenShot = new Texture2D(Screen.width, Screen.height, TextureFormat.RGB24, false);

    // 读取屏幕像素信息并存储为纹理数据，
    screenShot.ReadPixels(new Rect(0, 0, Screen.width, Screen.height), 0, 0);
    screenShot.Apply();

    // 然后将这些纹理数据，成一个png图片文件
    byte[] bytes = screenShot.EncodeToPNG();

	//把数据返回给安卓层
    using (AndroidJavaClass jc = new AndroidJavaClass("com.unity3d.player.UnityPlayer"))
    {
        AndroidJavaObject jo = jc.GetStatic<AndroidJavaObject>("currentActivity");
        jo.Call("SendScreenshotData", bytes);
    }
}
```

## Android代码

- 定义Bitmap变量
- 定义Timer定时

```
Bitmap bitmap = null;
Timer timer = null;
```

- 接受Unity发送过来的截图信息并转换为Bitmap类型

```
public void SendScreenshotData(byte[] bytes)
{
    bitmap = BitmapFactory.decodeByteArray(bytes, 0, bytes.length);
}
```

- SDK截图回调

```
// 游戏助手内截屏分享功能
YSDKApi.setScreenCapturer(new IScreenImageCapturer() {
	@Override
	public Bitmap caputureImage() {
		
		//如果没有缓存则通知Unity截图
		if (bitmap == null) {

			UnityPlayer.UnitySendMessage("Directional Light", "CaputureImage", "");
		}

	    //强行延迟等待截图数据
		while (bitmap == null) {

			try {
				Thread.sleep(500);
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}

		//开始一个5秒定时器
		if (timer == null) {

			timer = new Timer();

			timer.schedule(new TimerTask() {
				@Override
				public void run() {
					
					//删除缓存和定时器
					bitmap = null;

					timer = null;
				}
			}, 5000);
		}

		return bitmap;

	}
});
```

> 如果你有更好的思路和解决方法，也请多多指教