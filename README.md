#1 获取SDK

SDK版本号v0.16（alpha）

#2 开发

下面我们开始配置Android工程，以支持识途定位。

##2.1 添加Jar包以及.so文件

+ 获取识途定位SDK（包括.jar及.so文件）；
+ 在Android对应工程中，新建libs文件夹；
+ 将.jar拷贝至libs下，将.so文件拷贝至libs/armeabi下；
+ 在Android工程中，将.jar文件添加至BuildPath中。

##2.2 AndroidMenifest.xml
在AndroidMenifest.xml的<application>标签中添加如下代码

```
<meta-data
	android:name="ShituKey"
	android:value="你的Key" 
/>
```
因为识途定位SDK使用网络定位，所以请记得为您的应用添加网络权限。

若要使用识途的WiFi定位，还需加上如下权限

```
android.permission.WAKE_LOCK
```


##2.3 添加初始化代码
在任何Activity或Application中添加定位初始化代码（需先于定位调用）

```
//传入Context对象
ShituLocationLoader.getInstance(this);
```

##2.4 获取建筑列表
定位需要先获取您的建筑列表，每个建筑都对应一个ShituStore对象，定位时需传入该对象。

获取建筑列表，先实例化```ShituStoreDatas对象```，然后调用```fetchDatas()```方法。```FetchDatas()```方法使用异步获取列表，获取对象会回调至```OnGetSHTStoreDataListener```中。

```
//为Activity或其他对象，实施OnGetSHTStoreDataListener接口。

// this为context对象
ShituStoreDatas storeDatas = new ShituStoreDatas(this);
storeDatas.addOnGetSHTStoreDataListener(this);
storeDatas.fetchDatas();

...
@Override
public void onGetDatas(List<ShituStore> stores) {
	//TODO 存储建筑
}
```

##2.5 定位

在Activity的onCreate()方法中，构造定位参数类ShituLocationParameters，需传入参数

+ 要进行定位的建筑对象，为ShituStore对象，由4节中所述。
+ 初始楼层
+ 地图像素宽度的集合（SparseIntArray对象，键为楼层，值为像素宽度）
+ 地图像素高度的集合（SparseIntArray对象，键为楼层，值为像素高度）
+ 定位模式，可选的为```ShituLocationManager.LOC_MAG_ONLY```, ```ShituLocationManager.LOC_MAG_WIFI```以及```ShituLocationManager.LOC_WIFI_ONLY```。

其中，像素高度及宽度依照具体室内地图图片，**该宽度与高度，应等于您在识途云平台上传入的地图图片的高宽度，其包含楼层也需与识途云平台上传的一致**，否则，因比例尺错误会引起定位的准确性降低。

此外，不同定位模式对应的技术不同，各定位模式介绍如下：

+ LOC_MAG_ONLY: 使用纯地磁定位，暂无法确定初始楼层（我们会在今后的版本中提供该功能），需传入准确的初始楼层；
+ LOC_MAG_WIFI: 使用地磁与WiFi混合定位，可自动判断初始楼层；
+ LOC_WIFI_ONLY: 使用纯WiFi定位，可自动判断初始楼层；

例如

```
ShituLocationParameters parameters = new ShituLocationParameters(store, (short)6, mapWidthPx, mapHeightPx, ShituLocationManager.LOC_MAG_ONLY);
	
```

然后使用ShituLocationManager.newManager()构建ShituLocationManager对象，并在Activity的生命周期方法中调用相关的方法。

设置回调setOnNatureListener()，并在回调中实现相应的逻辑;

示例：

```
public class LocationTestActivity extends Activity implements ShituLocationListener {
	private ShituLocationManager mLocationManager;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		//分别代入参数，store，由ShituStoreDatas获取的建筑对象，初始楼层（目前SDK版本暂不支持楼层判断）
		//所显示的地图的像素宽度以及高度，以及定位模式		
		ShituLocationParameters parameters = new ShituLocationParameters(store, (short)6, mapWidthPx, mapHeightPx, ShituLocationManager.LOC_MAG_ONLY);

		mLocationManager = ShituLocationManager.newManager(this, parameters);
		mLocationManager.setOnNatureListener(this);
	}
	
	@Override
	protected void onResume() {
		super.onResume();
		mLocationManager.onResume();
		mLocationManager.start();
	}
	
	@Override
	protected void onPause() {
		super.onPause();
		mLocationManager.onPause();
	}

	@Override
	protected void onDestroy() {
		super.onDestroy();
		mLocationManager.onDestroy();
	}

	@Override
	public void onGetLocation(Position position) {
		//获得坐标

	}

	@Override
	public void onGetStatus(int status) {
		//获得定位状态，包括
		//STATUS_START_LOCATE: 定位开始，请用户沿当前道路持续行走即可获取位置
	 	//STATUS_FIRST_RELIABLE_LOCATE: 获取第一个相对可靠位置
	 	//STATUS_MEDIUM_ACCURACY: 传感器精度一般，可进行定位，建议进矫正
	 	//STATUS_UNRELIABLE_ACCURACY: 传感器精度不可靠，必须进行矫正方可正确定位
	 	//STATUS_FAILED_TO_GETLOCATION_MANYTIMES: 长时间无法获取位置，建议重新行走定位
		//STATUS_RECONNECT_SUCCESS, 重新连接成功

	}

	@Override
	public void onGetAngle(int angle) {
		//获得角度
	}

	@Override
	public void onException(String e, int errorType) {
		//异常
	}
```

##2.6 返回结果

识途定位SDK（Android）通过ShituLocationListener的接口，将结果回调至主线程。

其中，需要特殊注意的是，只有当在```onGetStatus(int status)```中拿到```STATUS_FIRST_RELIABLE_LOCATE```后，在onGetPosition中获取的位置才具有较高的可靠性。此前的位置坐标均可丢弃。

回调```onGetPosition(Position position)```中获取的Position对象，包含了定位的准确信息，其中包括坐标、楼层以及系统建议的精确度的半径参数（像素单位）。

其中，坐标为像素坐标，即相对于室内地图左上角（0，0）的坐标，该坐标系x轴向右增长，y轴向下增长。此坐标与使用**[识途Creator](http://ubirouting.com/creator.php)**采集过程中的平面图坐标系一致。



