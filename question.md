###Q: 无地磁传感器的手机是否能够使用识途定位SDK（Android）

市面上部分低端Android手机不带磁传感器，无法使用包含地磁的定位方式：

+ ShituLocationManager.LOC_MAG_ONLY
+ ShituLocationManager.LOC_MAG_WIFI

仅支持LOC_WIFI_ONLY模式。

###Q: 纯地磁定位为何需要行走一段距离

纯地磁定位，需要用户行走一段距离后方可确定初始位置，此后在定位过程中，系统不断校正位置，达到定位的效果。初始行走状态是为了让系统充分学习用户数据，为以后的定位做基准。

其他关于定位技术的问题，可见[识途定位技术Q&A](http://ubirouting.com/qa.php).

