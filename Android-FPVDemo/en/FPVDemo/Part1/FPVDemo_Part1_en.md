# DJI Mobile Android SDK Tutorial

## How to create a Camera Application: Part 1/2

You can download the demo project for this tutorial from here:
<https://github.com/DJI-Mobile-SDK/Android-FPVDemo-Part1.git>

We **strongly** recommend that you have the demo project open as reference as you work through this tutorial.

### Overview
In part 1 of this tutorial, we will cover how to set up your programming environment, apply for and activate your DJI app, and view your drone's camera feed through the app.

### 1.Preparation


(1) Download the Mobile SDK for Android from the following URL: 
<http://dev.dji.com/cn/products/sdk/mobile-sdk/downloads>

(2) Update the firmware of the aircraft (Phantom 3 Professional, Phantom 3 Advanced or Inspire 1) through the URL: <http://dev.dji.com/cn/products/sdk/mobile-sdk/downloads>

Refer to "Updating the Aircraft Firmware": <http://download.dji-innovations.com/downloads/phantom_3/en/How_to_Update_Firmware_en.pdf> to update the firmware.

(3) Set up an Android development environment (if you do not yet have one). Throughout this tutorial we will be using Eclipse 4.2.2, which you can download here: <https://eclipse.org/downloads/packages/eclipse-classic-422/junosr2>. Once Eclipse is installed, you will then have to install the Eclipse Android Development Tool Plug-In, found here: http://developer.android.com/intl/zh-TW/sdk/installing/installing-adt.html

*Note: Google's support for Android Development Tools in Eclipse is ending. If you would like to complete this demo using Eclipse as we have, or if you have already completed this demo in Eclipse, you can find instructions to migrate your project into Android Studio here: <https://developer.android.com/intl/zh-TW/sdk/installing/migrate.html>. If you would like to follow this tutorial using Android Studio, the mobile SDK folder contains an Android Studio library as well as an Eclipse one. Instructions on how to import the SDK library are given for both Eclipse and Android Studio below.*

### 2.Setting up your Programming Environment

###Eclipse

(1) Create a new 'Android Application Project'. Name the Application, Project and Package as you please. Under the 'Create Activity' page of the project set-up, create a blank activity, and name it 'FPVActivity'. The layout activity should automatically fill out to 'activity_fpv'.

(2) Unzip the SDK package downloaded from the DJI website. Import the folder **Lib** (Eclipse\DJI-SDK-Android-V2.1.0) into Eclipse (File -> Import -> Android -> Existing Android Code into Workspace). Next, add the imported file to your library (right click on your project -> Select "**Properties**" -> Select "**Android**" -> Add).
![setLib](https://github.com/dji-sdk/Mobile-SDK-Tutorial/raw/master/Android-FPVDemo/en/images/1_importLib.png)

(3) The imported library should now be located as shown below:
![checkLib](https://github.com/dji-sdk/Mobile-SDK-Tutorial/raw/master/Android-FPVDemo/en/images/1_CheckLib.png)

###Android Studio
(1) Start a new Android Studio Project. Give the application any name you like. Hit 'next' until you reach the 'Customize the Activity' page, where you should name your activity 'FPVActivity'. The layout name should automatically fill out with 'activity_fpv'. Press 'Finish'.

(2) Unzip the SDK package downloaded from the DJI website. Go to File -> New -> Import Module. In the 'Source Directory' field, find the DJI-SDK-LIB folder location (Android Studio\DJI-SDK-Android-V2.1.0\Lib\DJI-SDK-LIB). Press Finish.

*Note: The folder 'Android Studio' is found in the SDK package downloaded from the DJI website. The library used in the demo project code is from the 'Eclipse' folder in the same SDK package, so if you are working in Android Studio make sure that you are using the correct library from the 'Android Studio' folder. For your convenience the SDK package download link is reproduced here <http://dev.dji.com/cn/products/sdk/mobile-sdk/downloads>*
![importModule](https://raw.githubusercontent.com/alexanderwangus/Mobile-SDK-Tutorial/master/Android-FPVDemo/en/images/importModuleScreenshot.png)
Next, right click on the 'app' module in the file directory to the left, and click 'Open Module Settings". Navigate to the 'Dependencies' tab. Press the green plus sign, click 'Module Dependency', and select ':DJI-SDK-LIB'. Press 'OK' to confirm. After Gradle finishes rebuilding, you're environment will be ready!

![addDependency](https://raw.githubusercontent.com/alexanderwangus/Mobile-SDK-Tutorial/master/Android-FPVDemo/en/images/addDependencyScreenshot.png)


### 3.Activating your App

(1) Register for an account at <http://dev.dji.com>. Once registered, click on your name in the upper right corner. Click on 'Mobile SDK', then 'Create APP' and fill out the creation form. Type in your project's package name in the 'Identification Code' field.

(2) Activate the SDK: 

Copy both the 'uses-permission' lines of code and the highlighted meta-data element into your **AndroidManifest.xml** file for activation, as shown below.  

![appKeyMetaData](https://github.com/dji-sdk/Mobile-SDK-Tutorial/raw/master/Android-FPVDemo/en/images/1_appKeyMetaData.png)

Fill in the 'android:value' field with the APP KEY that you have applied for from <http://dev.dji.com>.

![appKey](https://github.com/dji-sdk/Mobile-SDK-Tutorial/raw/master/Android-FPVDemo/en/images/1_appKey.png)

In your FPVActivity.java file, add the following variable in the FPVActivity class.
~~~java
private static final String TAG = "FPVActivity";
~~~
In your onCreate method, add the following code.
~~~java
	new Thread(){
		public void run(){
			try{
				DJIDrone.checkPermission(getApplicationContext(), new DJIGerneralListener(){
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
			} catch (Exception e) {
        			// TODO Auto-generated catch block
        			e.printStackTrace();
        		}
		}
	}.start();
~~~
Run your project code on an Android device or Android emulator to complete the activation procedure. Instructions for running your code can be found here: http://developer.android.com/intl/zh-TW/tools/building/building-eclipse.html

Check the 'LogCat' panel at the bottom of your coding environment window for a return message.

![logcat](https://github.com/dji-sdk/Mobile-SDK-Tutorial/raw/master/Android-FPVDemo/en/images/logcatScreenshot.png)

Check the error code against the table below:


Error Code  	  | Description 
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

If you have received an error code that is not '0', follow the instructions below:

1. Ensure that you have access to the internet
2. Ensure that, when creating an app on the http://dev.dji.com website, you have filled out the 'Identification Code' field with your project package name 
3. Ensure that APP KEY has not reach its installed capacity limit. If this does not solve the issue, refer to the table below for further troubleshooting.
If you have further questions, contact our mobile SDK support by sending emails to <sdk@dji.com>



### 3.Adding Android Open Accessory (AOA) support

In order to support the new remote controller from DJI, AOA is required. Modify **AndroidManifest.xml** to set **.DJIAoaActivity** as the main activity, which is served the entry point when the application is initiated. 

Under the 'manifest' element in your **AndroidManifest.xml** file, add the following lines of code: 
- **uses-feature android:name="android.hardware.usb.accessory" android:required="false"**
- **uses-feature android:name="android.hardware.usb.host" android:required="false"**

Under the 'application' element, add the following line of code:
- **uses-library android:name="com.android.future.usb.accessory"**
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

Create a new Android Activity Page, using 'DJIAoaActivity' as the activity name. (Right click on your package -> New -> Other -> Android -> Android Activity). In your newly created 'DJIAoaActivity.java' file, locate the 'onCreate' method, and copy the following code in to enable AOA support.
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

Create a new Android Activity Page called '**DemoBaseActivity**', and add the following code. This code allows you to pause or resume the AOA data connection service when the **onPause()** or **onResume()** lifecycle callbacks are called.

This will be our project's base activity. Change your 'FPVActivity' class header so that it now extends 'DemoBaseActivity', rather than 'Activity'.
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
	
### 4.Implementing the First Person View (FPV)
	
(1) Initiate the SDK API according to the type of the aircraft.

In 'FPVActivity.java', in the FPVActivity class, add the variable below.
~~~java
private int DroneCode;
~~~
Import the following package.
~~~java
import dji.sdk.api.DJIDroneTypeDef.DJIDroneType;
~~~
Add two lines of code in the 'onCreate' method as shown below. Additionally, within the 'FPVActivity' class, copy the 'onInitSDK' method shown below.
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
	}
	
	...
~~~
(2) In the 'onCreate' method, use the following line of code to connect to the aircraft. Make sure call this method only after the code that activates your APP key.
~~~java
	DJIDrone.connectToDrone(); // Connect to the drone
~~~	
(3) Locate the 'activity_fpv.xml' file (res/layout/activity_fpv.xml). This file controls the view of FPVActivity. Add the following **surfaceview** element code in the 'activity_fpv.xml' file.
~~~xml
	<dji.sdk.widget.DjiGLSurfaceView
		android:id="@+id/DjiSurfaceView_02"
		android:layout_width="fill_parent"
		android:layout_height="fill_parent" />
~~~	
In your 'FPVActivity.java' file, in the FPVActivity class, add the variable below.
~~~java
private DJIReceivedVideoDataCallBack mReceivedVideoDataCallBack = null;
~~~
Create a **private DjiGLSurfaceView mDjiGLSurfaceView** element as shown below. Add the following code in the 'onCreate' method, making sure to insert it after where you call the connectToDrone() method. This code implements the the FPV view by using the SDK API **public void setReceivedVideoDataCallBack(DJIReceivedVideoDataCallBack mReceivedVideoDataCallBack)** to obtain the live preview video data(raw H264 format) which can then be processed using code. Here, we use the decoder provided by DJI to decode the video data and display it to the live view using **SurfaceView**. Adding the code below sends video data to **DjiGLSurfaceView** for decoding and displaying. 
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

After the activity is closed, you should first call **DJIDrone.getDjiCamera().setReceivedVideoDataCallBack(null)** to reset the callback and then destroy **mDjiGLSurfaceView** to complete the activity life cycle.
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
Be aware that you must first start up **mDjiGLSurfaceView** and before setting the callback, and that you must set the callback to null before destroying **mDjiGLSurfaceView**.

### 5.Connect to your DJI Drones

After you have built and run the project successfully, you can now connect your mobile device to an aircraft to check the FPV View. Follow the appropriate instructions for your specific aircraft modelp:

#### Connecting to a DJI Inspire 1 or Phantom 3 Professional/Advanced:

1. Turn on your remote controller, then turn on your aircraft

2. Connect your mobile device to the remote controller using a USB cable. Tap your own app and a message window "Choose an app for the USB device" will prompt.

3. Tap "OK" when the messagte window prompts "Allow the app to access the USB accessory".

4. Tap "OK" when the activation alert displays.

5. You are ready to use the FPV View app. 

#### Connecting to a DJI Phantom 2 Vision+ or Phantom 2 Vision:
	
1. Turn on your remote controller, then turn on your aircraft.

2. Ensure that the mobile device has access to the Internet. Tap the app to activate and select "OK" when the activation is done.

3. Turn on the Wi-Fi range extender

4. Turn on the Wi-Fi on your mobile device and connect to the Wi-Fi network of Phantom-xxxxxx (where xxxxxx is your range extender’s SSID number)

5. You are ready to use the FPV View app.

### 5.Checking the result of FPV View
If you can see the live video stream in the app, congratulations! You can now move on to the Part 2 of the tutorial:
![runAppScreenShot](https://github.com/dji-sdk/Mobile-SDK-Tutorial/raw/master/Android-FPVDemo/en/images/runAppScreenShot.png)

### 6.Where to Go From Here?

You’ve learned how to setup the DJI Mobile SDK's developmengt environment and use it to show the FPV view from the aircraft's camera. We will add the capture and record functions in the app in next part of this tutorial. Hope you enjoy it!
