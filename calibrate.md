在使用包含地磁定位的定位模式时，若传感器精度下降，则会影响定位的精度。

**识途定位SDK（Android）**提供了校正方法，帮助开发者在自己的应用中校正传感器。

校正过程中，应用友好的语言提示用户，握住手机，并连续沿八字晃动手机进行校正，并在该界面调用识途定位SDK（Android）中的```CalibratorJudger```进行校正。

示例：

```
public class MagneticCalibrateActivity extends Activity implements
		OnCalibrationListener {
		
		private CalibratorJudger mCalibratorJudger;
		
		@Override
		protected void onCreate(Bundle savedInstanceState) {
			super.onCreate(savedInstanceState);
			setContentView(R.layout.activity_layout);
			
			//初始化CalibratorJudger，并设置回调方法
			mCalibratorJudger = new CalibratorJudger(this);
			mCalibratorJudger.setOnCalibrationListener(this);
			
			//开始校正
			mCalibratorJudger.startCalibration();
		}
		
		@Override
		protected void onResume() {
			super.onResume();
			mCalibratorJudger.onResume();
		};

		@Override
		protected void onPause() {
			super.onPause();
			mCalibratorJudger.onPause();
		}

		@Override
		protected void onDestroy() {
			super.onDestroy();
			mCalibratorJudger.onDestroy();
		}

		@Override
		public void onCalibrating(float percent) {
		
			//校正进度会以百分比的形式回调，此时可在界面上用百分比提示用户校正的进度
			//例如：textviewCalibrator.setText(percent * 100 + "%");
		}

		@Override
		public void onFinish() {
			//校正结束时回调，可在此加入启动定位Activity的逻辑。
		}

		
}
		
```