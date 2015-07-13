# DJI Mobile Android SDK Tutorial (1)

## How to create a simple FPV app: Part 1/2

### 1.Preparation

(1) Download the Android SDK:
You can download the latest Android SDK from here: 
<http://dev.dji.com/cn/products/sdk/mobile-sdk/downloads>

(2)Update your drone (Phantom 3 Professional, Phantom 3 Advanced or Inspire 1)'s firmware using that on <http://dev.dji.com/cn/products/sdk/mobile-sdk/downloads>

Please follow the process "Updating the Aircraft Firmware" according to the document here <http://download.dji-innovations.com/downloads/phantom_3/en/How_to_Update_Firmware_en.pdf> to update the firmware.

### 2.Unzip the SDK package and import Lib into your project
(1) Unzip the SDK package. Import the folder Lib into your eclipse, add it as a library for your own project (Right click on your project->Select "Properties"->Select "Android").
![setLib](http://gitlab.djicorp.com/jian.zhao/SDK-SampleCodes-Android/raw/master/FPVDemo-Tutorial/images/1_importLib.png)

(2) Check that the library is imported.

![checkLib](http://gitlab.djicorp.com/jian.zhao/SDK-SampleCodes-Android/raw/master/FPVDemo-Tutorial/images/1_CheckLib.png)

### 3.Write the codes for FPV View

(1) Activate the SDK: 

Add the following meta-data element in your AndroidManifest as the configuration of APP KEY for activation.

![appKeyMetaData](http://gitlab.djicorp.com/jian.zhao/SDK-SampleCodes-Android/raw/master/FPVDemo-Tutorial/images/1_appKeyMetaData.png)

Please use the APP KEY you have applied in <http://dev.dji.com> as the value of the meta-data element. When you apply the APP KEY, you should use an Identification Code the same as your project's package name.

![appKey](http://gitlab.djicorp.com/jian.zhao/SDK-SampleCodes-Android/raw/master/FPVDemo-Tutorial/images/1_appKey.png)
 
Add the following codes before calling SDK APIs,
~~~java
	new Thread(){
		public void run(){
			try{
				DJIDrone.checkPermission(getApplicationContext(), new DJIGeneralListener(){
					@Override
					public void onGetPermissionResult(int result){
						if(result == 0) {
							// show success
							Toast.makeText(FPVActivity.this, "activation success", Toast.LENGTH_SHORT).show();
						}else {
							// show errors
							Toast.makeText(FPVActivity.this, "error: "+result, Toast.LENGTH_SHORT).show();
						}
					}
				});
			}
		}
	}
~~~
The SDK API can only be used when its activation is successful. The error code for the APP KEY activation is as follows. If you run into any problem in activation, please first check the following points:(1) 
Check whether the mobile device has Internet connections; (2) Check wheter using your project package name as the identification code in applying the app key; (3) Check whether the installed capacity of the app key
has run over. If you have further questions, please contact our mobile SDK support by sending emails to <sdk@dji.com>

result  	  | Description 
------------- | -------------
0   | Check permission successful
-1  | Cannot connect to Internet
-2  | Invalid app key
-3  | Get permission data timeout
-4  | Device uuid not match
-5  | Project package name does not match the app 	   key's identification code
-6  | App key is forbidden
-7  | Activated device number is up to the maximum 		available one
-8  | App key's platform is not correct
-9  | App key does not exist
-10 | App key has no permission
-11 | Server parser failed
-12 | Error in server obtaining uuid
-13 | Server app package name abnormal
-14 | Server parsing activation data failed
-15 | AES 256 encryption unsupported
-16 | AES 256 encryption failed
-17 | Get device uuid failed
-18 | Empty app key
-1000 | Server error 


(2) Add the Android Open Accessory (AOA) support:

We need AOA to support DJI's new Remote Controller firmware. In AndroidManifest.xml, use DJIAoaActivity as the main activity, which will be the entry point as the application is open. 
~~~xml
	<activity
		android:name=".DJIAoaActivity"
		android:configChanges="orientation|screenSize|keyboardHidden|keyboard"
		android:screenOrientation="sensorLandscape" >
		
		<intent-filter>
			<action android:name="android.intent.action.MAIN" />
			<category android:name="android.intent.category.LAUNCHER" />
		</intent-filter>
		
		<intent-filter>
			<action android:name="android.hardware.usb.action.USB_ACCESSORY_ATTACHED" />
		</intent-filter>
		
		<meta-data
			android:name = "android.hardware.usb.action.USB_ACCESSORY_ATTACHED"
			android:resource = "@xml/accessory_filter" />
	</activity>
~~~

DJIAoaActivity include the following code for supporting AOA. 
~~~java
	private static boolean isStarted = false;
	...
	@Override
	protected void onCreate(Bundle savedInstanceState){
		super.onCreate(savedInstanceState);
		setContentView(new View(this));
		
		if (isStarted) {
			//Do nothing
		}else {
			isStarted = true;
			ServiceManager.getInstance();
			UsbAccessoryService.registerAoaReceiver(this); 
			Intent intent = new Intent(DJIAoaActivity.this, FPVActivity.class);
			startActivity(intent);
		}
		
		Intent aoaIntent = getIntent();
		if(aoaIntent != null) {
			String action = aoaIntent.getAction();
			if (action==UsbManager.ACTION_USB_ACCESSORY_ATTACHED || action == Intent.ACTION_MAIN){
				Intent attachedIntent = new Intent();
				attachedIntent.setAction(DJIUsbAccessoryReceiver.ACTION_USB_ACCESSORY_ATTACHED);
				sendBroadcast(attachedIntent);
			}
		}
		finish();
	}
	...
~~~ 

OK. The next step is to add codes for pausing or resuming the Service for AOA data connection when an Activity's **onPause()** or **onResume()** lifecycle callback is called. You can use DemoBaseActivity as the base activity, which includes the following codes,
~~~java
	...
	@Override
	protected void onResume(){
		super.onResume();
		ServiceManager.getInstance().pauseService(false); // Resume the service
	} 
	
	@Override
	protected void onPause() {
		super.onPause();
		ServiceManager.getInstance().pauseService(true); // Pause the service
	}
~~~	
	
(3) Implement the FPV View
	
(a) To use the SDK APIs, we first need to initiate the SDK APIs according to the type of drones. We have different APIs for different types of drones.
~~~java
	@Override
	protected void onCreate(Bundle savedInstanceState){
		
		...
		DroneCode = 1; 
		onInitSDK(DroneCode);
		...
	}
	
	private void onInitSDK(int type){
		switch(type){
			case 0: {
				DJIDrone.initWithType(this.getApplicationContext(), DJIDroneType.DJIDrone_Vision);
				// The SDK initiation for Phantom 2 Vision or Vision Plus 
				break;
			}
			case 1: {
				DJIDrone.initWithType(this.getApplicationContext(), DJIDroneType.DJIDrone_Inspire1); 
				// The SDK initiation for Inspire 1 or Phantom 3 Professional.
				break;
			}
			case 2: {
				DJIDrone.initWithType(this.getApplicationContext(), DJIDroneType.DJIDrone_Phantom3_Advanced);
				// The SDK initiation for Phantom 3 Advanced
				break;
			}
			case 3: {
				DJIDrone.initWithType(this.getApplicationContext(), DJIDroneType.DJIDrone_M100);
				// The SDK initiation for Matrice 100.
				break;
			}
			default:{
				break;
			}
	}
	
	...
~~~	
(b) After initiating the SDK APIs, we need to use the following line of code to connect to the drone,
~~~java
	DJIDrone.connectToDrone(); // Connect to the drone
~~~	
(c) Now we can implement the code for FPV view by using the API **public void setReceivedVideoDataCallBack(DJIReceivedVideoDataCallBack mReceivedVideoDataCallBack)** to obtain the live view video data(raw H264 format). The video data can be processed by writing codes for specific applications. Here, we use the decoder provided by DJI to decode the video data and show the live view using **SurfaceView**.

Add the surfaceview in the layout activity_fpv.xml in the sample code） which is used as the **FPVActivity**'s view,
~~~xml
	<dji.sdk.widget.DjiGLSurfaceView
		android:id="@+id/DjiSurfaceView_02"
		android:layout_width="fill_parent"
		android:layout_height="fill_parent" />
~~~		
		
Add the code to send video data to **DjiGLSurfaceView** for decoding and displaying. 
~~~java
	...
	
	private DjiGLSurfaceView mDjiGLSurfaceView;
	
	...
	
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_fpv);
		
		...
		
		mDjiGLSurfaceView = (DjiGLSurfaceView)findViewById(R.id.DjiSurfaceView_02);
		mDjiGLSurfaceView.start();
		mReceivedVideoDataCallBack = new DJIReceivedVideoDataCallBack(){
			@Override
			public void onResult(byte[] videoBuffer, int size){
				mDjiGLSurfaceView.setDataToDecoder(videoBuffer, size);
			}
		};
		DJIDrone.getDjiCamera().setReceivedVideoDataCallBack(mReceivedVideoDataCallBack);
		
		...
		
		
	}
~~~	
Note that you should start **mDjiGLSurfaceView** first, then call **DJIDrone.getDjiCamera().setReceivedVideoDataCallBack(mReceivedVideoDataCallBack)** to send the video data to mDjiGLSurfaceView for decoding and displaying.

When the activity is closed, you should first call **DJIDrone.getDjiCamera().setReceivedVideoDataCallBack(null)** to reset the callback using null, and then destroy **mDjiGLSurfaceView**. Codes are as follows,
~~~java
	...
	
	@Override
	protected void onDestroy() {
		if (DJIDrone.getDjiCamera() != null) {
			DJIDrone.getDjiCamera().setReceivedVideoDataCallBack(null);
		}
		mDjiGLSurfaceView.destroy();
		
		...
	}
~~~
You should pay attention to the sequence of the start of mDjiGLSurfaceView and the setting of the callback, and the sequence of the destroy of mDjiGLSurfaceView and setting the callback using null.

(d) Build and Run the project, check if everything is OK. if you see the following screenshot, then you can start connecting to your aircraft and enjoy the Video Stream from its camera!

![afterCompileScreenShot](http://gitlab.djicorp.com/jian.zhao/SDK-SampleCodes-Android/raw/master/FPVDemo-Tutorial/images/afterComplileScreenShot.png)

### 4.Connect to your DJI Drones

After you build and run the project successfully, you can now connect your mobile device to your DJI Drones to check the FPV View, here are the guides:

* Connect to a DJI Inspire 1 or Phantom 3 Professional/Advanced:

1. Firstly, start your remote controller, then power on your aircraft

2. Connect your mobile device to the remote controller using a USB cable. Select your own app when a window "Choose an app for the USB device" pops up.

3. Click "OK" when a window asking "Allow the app to access the USB accessory" pops up.

4. Click "OK" when the activation alert pops up.

5. Now you can enjoy the FPV of your drone.

* Connect to a DJI Phantom 2 Vision+ or Phantom 2 Vision:
	
1. Power on your remote control firstly, and then start your aircraft

2. Make sure the mobile device has Internet connection, open the app to activate the app. Click "OK" when the activation is successful. 

3. Power on the Wi-Fi range extender

4. Turn on the Wi-Fi on your mobile device and connect to the network named Phantom-xxxxxx (where xxxxxx is your range extender’s SSID number)

5. Now you can enjoy the FPV of your drone.

### 5.Enjoy the FPV View
If you can see the live video stream in the app, congratulations! You can start the Part 2 tutorial now:
![runAppScreenShot]()

### 6.Go to Next Tutorial

You can download the demo project for this tutorial from here: <http://gitlab.djicorp.com/jian.zhao/SDK-SampleCodes-Android.git>

You’ve learned how to setup the DJI Mobile SDK's developmengt environment and use it to show the FPV view from the aircraft's camera. We will add the capture and record functions in the app in next part of this tutorial. Hope you enjoy it!