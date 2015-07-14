# DJI Mobile Android SDK Tutorial

## How to create a Camera Application: Part 1/2

### 1.Preparation

(1) Download the SDK:
Download the Mobile SDK for Android from the following URL: 
<http://dev.dji.com/cn/products/sdk/mobile-sdk/downloads>

(2)Update the firmware of the aircraft (Phantom 3 Professional, Phantom 3 Advanced or Inspire 1) through the URL: <http://dev.dji.com/cn/products/sdk/mobile-sdk/downloads>

Refer to "Updating the Aircraft Firmware" from the URL: <http://download.dji-innovations.com/downloads/phantom_3/en/How_to_Update_Firmware_en.pdf> to update the firmware.

### 2.Unzip the SDK package and import Lib 
(1) Unzip the SDK package. Import the folder **Lib** into eclipse, add it as a library for your own project (Right click on your project->Select "**Properties**"->Select "**Android**").
![setLib](https://github.com/dji-sdk/Mobile-SDK-Tutorial/raw/master/Android-FPVDemo/en/images/1_importLib.png)

(2) Locate the imported library.
![checkLib](https://github.com/dji-sdk/Mobile-SDK-Tutorial/raw/master/Android-FPVDemo/en/images/1_CheckLib.png)

### 3.Implementing for FPV View

(1) Activate the SDK: 

Add the highlighted meta-data elements into your **AndroidManifest** for activation.

![appKeyMetaData](https://github.com/dji-sdk/Mobile-SDK-Tutorial/raw/master/Android-FPVDemo/en/images/1_appKeyMetaData.png)

Input the APP KEY that you have applied from <http://dev.dji.com>. Note that the Identification Code is identical to your project's package name.

![appKey](https://github.com/dji-sdk/Mobile-SDK-Tutorial/raw/master/Android-FPVDemo/en/images/1_appKey.png)
 
Add the following codes before calling the SDK APIs,
~~~java
	new Thread(){
		public void run(){
			try{
				DJIDrone.checkPermission(getApplicationContext(), new DJIGeneralListener(){
					@Override
					public void onGetPermissionResult(int result){
						if(result == 0) {
							// show success
							Log.e(TAG, "onGetPermissionResult ="+result);
							Log.e(TAG, 
"onGetPermissionResultDescription="+DJIError.getCheckPermissionErrorDescription(result));
						}else {
							// show errors
							Log.e(TAG, "onGetPermissionResult ="+result);
							Log.e(TAG, 
"onGetPermissionResultDescription="+DJIError.getCheckPermissionErrorDescription(result));
						}
					}
				});
			}
		}
	}
~~~
Complete the activate procedure before using the SDK API for the first time. The error code for the APP KEY activation is as follows.Follow the instruction below to troubleshoot the activation procedure:(1) 
Esnure you have access to the internet; (2) Ensure the project package name is identical to the Identification Code when applying the APP KEY; (3) Ensure that APP KEY has not reach its installed capacity limit.
If you have further questions, contact our mobile SDK support by sending emails to <sdk@dji.com>

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

In order to support the new remote controller from DJI, AOA is required. Modify **AndroidManifest.xml** to set **DJIAoaActivity** as the main activity, which is served the entry point when the application is initiated. And add **uses-feature android:name="android.hardware.usb.accessory" android:required="false"**, **uses-feature android:name="android.hardware.usb.host" android:required="false"** in **AndroidManifest.xml**. Add **uses-library android:name="com.android.future.usb.accessory"** under element application.
~~~xml
	...
	
	<uses-feature android:name="android.hardware.usb.accessory" android:required="false" />
	<uses-feature android:name="android.hardware.usb.host" android:required="false" />
	<application
		android:label="@string/app_name"
		android:theme="@style/AppTheme">
		
		<uses-library android:name="com.android.future.usb.accessory" />
	...
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

Added the following code in DJIAoaActivity to enable the support of AOA.
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

Add the following codes to pause or resume the AOA data connection service when the **onPause()** or **onResume()** lifecycle callback is called. Set the **DemoBaseActivity** as the base activity, which contains the following codes:
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
	
(3) Implementing the FPV View
	
(a) Initiate the SDK API according to the type of the aircraft.  
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
(b) Connect to the aircraft using the following codes:
~~~java
	DJIDrone.connectToDrone(); // Connect to the drone
~~~	
(c) Implement the FPV view by using the SDK API **public void setReceivedVideoDataCallBack(DJIReceivedVideoDataCallBack mReceivedVideoDataCallBack)** to obtain the live preview video data(raw H264 format). The video data can then be processed by using codes for specific purpose. Here, we use the decoder provided by DJI to decode the video data and display them to the live view using **SurfaceView**.

Add the **surfaceview** in the layout activity_fpv.xml in the sample code, which is used as the **FPVActivity**'s view,
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
Note  **mDjiGLSurfaceView** should be started first, follow by calling the  **DJIDrone.getDjiCamera().setReceivedVideoDataCallBack(mReceivedVideoDataCallBack)** to send the video data to 
**mDjiGLSurfaceView** for decoding and displaying.

After activity is closed, you should first call **DJIDrone.getDjiCamera().setReceivedVideoDataCallBack(null)** to reset the callback and then destroy **mDjiGLSurfaceView** to complete the life cycle.
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
Be aware of the sequence of the start of **mDjiGLSurfaceView** and the setting of the callback, and the sequence of the destroy of **mDjiGLSurfaceView** and setting the callback using null.

(d) Build and run the project, check if everything is running as desired. If the follow screen can be seen, it means that you have succesfully created an simple app by using the SDK APIs.
![afterCompileScreenShot](https://github.com/dji-sdk/Mobile-SDK-Tutorial/raw/master/Android-FPVDemo/en/images/afterComplileScreenShot.png)

### 4.Connect to your DJI Drones

After you build and run the project successfully, you can now connect your mobile device to the aircraft to check the FPV View. Follow the instruction to check your FPV View app:

#### Connect to a DJI Inspire 1 or Phantom 3 Professional/Advanced:

1. Turn on your remote controller, then turn on your aircraft

2. Connect your mobile device to the remote controller using a USB cable. Tap your own app and a message windows of "Choose an app for the USB device" prompts.

3. Tap "OK" when the messagte window prompts "Allow the app to access the USB accessory".

4. Tap "OK" when the activation alert displays.

5. Now you start using FPV View app. 

#### Connect to a DJI Phantom 2 Vision+ or Phantom 2 Vision:
	
1. Turn on your remote controller, then turn on your aircraft.

2. Ensure that the mobile device have access to the Internet. Tap the app to activate and select "OK" when the activation is done.

3. Turn on the Wi-Fi range extender

4. Turn on the Wi-Fi on your mobile device and connect to the Wi-Fi network of Phantom-xxxxxx (where xxxxxx is your range extender’s SSID number)

5. Now you start using FPV View app.

### 5.Check the result of FPV View
If you can see the live video stream in the app, congratulations! You can move on to the Part 2 of the tutorial now:
![runAppScreenShot](https://github.com/dji-sdk/Mobile-SDK-Tutorial/raw/master/Android-FPVDemo/en/images/runAppScreenShot.png)

### 6.Where to Go From Here?

You can download the demo project for this tutorial from here: <https://github.com/DJI-Mobile-SDK/Android-FPVDemo-Part1.git>

You’ve learned how to setup the DJI Mobile SDK's developmengt environment and use it to show the FPV view from the aircraft's camera. We will add the capture and record functions in the app in next part of this tutorial. Hope you enjoy it!
