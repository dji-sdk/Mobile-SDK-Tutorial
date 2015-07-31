# DJI Mobile Android SDK Tutorial

## How to create a Camera Application: Part 2/2

In Part 1, we implemented the basic FPV view which allowed us to display a live video stream from our aircraft's camera in our app. In this tutorial we will add photo taking and video recording features to the app, using the Inspire 1 as an example. Let's get started!

You can download the final project for this tutorial here: <https://github.com/DJI-Mobile-SDK/Android-FPVDemo-Part2.git>. We *strongly* recommend that you download the final project code and have it open as reference as you work through this tutorial.

### 0. Creating a Handler
We will be using a handler to display error and confirmation messages. Set up this handler by copying the code below into your FPVActivity class:
~~~java
private Handler handler = new Handler(new Handler.Callback() {
        
        @Override
        public boolean handleMessage(Message msg) {
            switch (msg.what) {
                case SHOWDIALOG:
                    showMessage(getString(R.string.demo_activation_message_title),(String)msg.obj); 
                    break;
                case SHOWTOAST:
                    Toast.makeText(FPVActivity.this, (String)msg.obj, Toast.LENGTH_SHORT).show();
                    break;
                default:
                    break;
            }
            return false;
        }
    });
    
    private Handler handlerTimer = new Handler();
    Runnable runnable = new Runnable(){
        @Override
        public void run() {
        // handler自带方法实现定时器
        try {

            handlerTimer.postDelayed(this, TIME);
            viewTimer.setText(Integer.toString(i++));

        } catch (Exception e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        }
    };
~~~

### 1. Implementing the capture function

The **private void captureAction()** function is used to take photos. In our source code, we implement a "Capture" button which calls this function whenever pressed.
~~~java
	 // function for taking photo
    private void captureAction(){
        
        CameraMode cameraMode = CameraMode.Camera_Capture_Mode;
        // Set the cameraMode as Camera_Capture_Mode. All the available modes can be seen in
        // DJICameraSettingsTypeDef.java
        DJIDrone.getDjiCamera().setCameraMode(cameraMode, new DJIExecuteResultCallback(){

            @Override
            public void onResult(DJIError mErr)
            {
                
                String result = "errorCode =" + mErr.errorCode + "\n"+"errorDescription =" + DJIError.getErrorDescriptionByErrcode(mErr.errorCode);
                if (mErr.errorCode == DJIError.RESULT_OK) {
                    CameraCaptureMode photoMode = CameraCaptureMode.Camera_Single_Capture; 
                    // Set the camera capture mode as Camera_Single_Capture. All the available modes 
                    // can be seen in DJICameraSettingsTypeDef.java
                    
                    DJIDrone.getDjiCamera().startTakePhoto(photoMode, new DJIExecuteResultCallback(){

                        @Override
                        public void onResult(DJIError mErr)
                        {
                            
                            String result = "errorCode =" + mErr.errorCode + "\n"+"errorDescription =" + DJIError.getErrorDescriptionByErrcode(mErr.errorCode);
                            handler.sendMessage(handler.obtainMessage(SHOWTOAST, result));  // display the returned message in the callback               
                        }
                        
                    }); // Execute the startTakePhoto API if successfully setting the camera mode as
                    	// Camera_Capture_Mode 
                } else {
                    handler.sendMessage(handler.obtainMessage(SHOWTOAST, result)); 
                    // Show the error when setting fails
                }
                
            }
            
        });
                   
    }
~~~
That was a lot of code we just threw at you, so let's break it down.

The first thing we need to do is define a CameraMode enum, which we will use to set the mode of the camera onboard our DJI Drone.
~~~java
CameraMode cameraMode = CameraMode.Camera_Capture_Mode;
~~~
The reason we defined this enum 'cameraMode' was so that we could pass it as a parameter for the **setCameraMode()** function that we are about to call. **setCameraMode()** sets the mode of the DJI drone's camera (Capture Mode, Playback Mode, Record Mode etc.). **setCameraMode()** takes in two parameters:

**setCameraMode(DJICameraSettingsTypeDef.CameraMode mode, DJIExecuteResultCallback mCall)**

The first parameter, a CameraMode enum, tells the function what mode to set the camera to. In this case, we tell it to set the camera to Capture Mode.
The second parameter is a callback function, which is run after **setCameraMode()** attempts to set the camera mode. The callback function is reproduced, in brief, below.
~~~java
new DJIExecuteResultCallback(){

            @Override
            public void onResult(DJIError mErr)
            {

                String result = "errorCode =" + mErr.errorCode + "\n"+"errorDescription =" + DJIError.getErrorDescriptionByErrcode(mErr.errorCode);
                if (mErr.errorCode == DJIError.RESULT_OK) {
                    // Take a photo!
                } else {
                    handler.sendMessage(handler.obtainMessage(SHOWTOAST, result)); 
                    // Show the error when setting fails
                }

            }

        });
~~~
The callback function takes in a confirmation signal from the drone, in the form of a DJIError object 'mErr'. If the error code given by 'mErr' matches the value DJIError.RESULT_OK, then code to take the photo is carried out. Else, a handler will show an appropriate error message depending on the error code.

Within the callback function we have code to tell the drone to take a photo.
~~~java
CameraCaptureMode photoMode = CameraCaptureMode.Camera_Single_Capture; 
// Set the camera capture mode as Camera_Single_Capture. All the available modes 
// can be seen in DJICameraSettingsTypeDef.java

DJIDrone.getDjiCamera().startTakePhoto(photoMode, new DJIExecuteResultCallback(){

	@Override
    public void onResult(DJIError mErr)
    {
    	String result = "errorCode =" + mErr.errorCode + "\n"+"errorDescription =" + DJIError.getErrorDescriptionByErrcode(mErr.errorCode);
        handler.sendMessage(handler.obtainMessage(SHOWTOAST, result));  // display the returned message in the callback               
    }
}); // Execute the startTakePhoto API if successfully setting the camera mode as
                        // Camera_Capture_Mode
~~~
If this code looks familiar, it's because it follows a structure almost identical to the larger function it is a part of! First we create a CameraCaptureMode enum called 'photoMode'. When the drone takes a photo, this enum instructs the drone whether it should take a single photo, a burst of photos, or a continuous stream of photos. For this example we'll be taking a single photo at a time.

The **startTakePhoto()** method tells the drone's camera to take a photo. Just like the **setCameraMode()** function, it takes in an enum and a callback function. We've just gone over what the enum it takes in is.
The callback function uses a handler to display a message giving an error code and an error description after the drone's camera attempts to take a photo. If a photo has successfully been taken, this message will confirm it.

And that's it! Add a "Capture" button into your app which calls this method, and give it a go!


### 2. Implement the recording function

The **recordAction()** method is almost identical to the **captureAction()** method we just implemented, with just a few key differences! Take a quick look at the code below:
~~~java
	 // function for starting recording
    private void recordAction(){
        // Set the cameraMode as Camera_Record_Mode.
        CameraMode cameraMode = CameraMode.Camera_Record_Mode;
        DJIDrone.getDjiCamera().setCameraMode(cameraMode, new DJIExecuteResultCallback(){
		 
            @Override
            public void onResult(DJIError mErr)
            {
                
                String result = "errorCode =" + mErr.errorCode + "\n"+"errorDescription =" + DJIError.getErrorDescriptionByErrcode(mErr.errorCode);
                if (mErr.errorCode == DJIError.RESULT_OK) {
                    
                    //Call the startRecord API
                    DJIDrone.getDjiCamera().startRecord(new DJIExecuteResultCallback(){

                        @Override
                        public void onResult(DJIError mErr)
                        {
                            
                            String result = "errorCode =" + mErr.errorCode + "\n"+"errorDescription =" + DJIError.getErrorDescriptionByErrcode(mErr.errorCode);
                            handler.sendMessage(handler.obtainMessage(SHOWTOAST, result));  // display the returned message in the callback               
                            
                        }
                        
                    }); // Execute the startTakePhoto API
                } else {
                    handler.sendMessage(handler.obtainMessage(SHOWTOAST, result));
                }
                
            }
            
        });
        
    }
~~~
Notice that the cameraMode enum has been set as **Camera_Record_Mode** because this time we want the camera to record.
~~~java
// Set the cameraMode as Camera_Record_Mode.
CameraMode cameraMode = CameraMode.Camera_Record_Mode;
~~~
Additionally, within our callback function, we call **startRecord()** instead of **startTakePhoto()**. **startRecord()** only takes in one parameter, a callback function. It does not take in an enum as **startTakePhoto()** does, as there is only one recording mode.
### 3. Implement the stopping recording function

Once the camera starts recording, we need some way to tell it to stop! That's where **stopRecord()** comes in. The code below should look quite familiar to you by now:
~~~java
	 // function for stopping recording
    private void stopRecord(){
    // Call the API
        DJIDrone.getDjiCamera().stopRecord(new DJIExecuteResultCallback(){

            @Override
            public void onResult(DJIError mErr)
            {
                
                String result = "errorCode =" + mErr.errorCode + "\n"+"errorDescription =" + DJIError.getErrorDescriptionByErrcode(mErr.errorCode);
                handler.sendMessage(handler.obtainMessage(SHOWTOAST, result));

            }
            
        });
    }

~~~

You can now add a 'Record' and 'Stop Recording' button to your app, and have them call **recordAction()** and **stopRecord()** respectively. Build and run the project, and it should look something like the screenshot below:
![recordVideoScreenShot](https://github.com/dji-sdk/Mobile-SDK-Tutorial/raw/master/Android-FPVDemo/en/images/recordVideo.png)

Then congratulations! Your Aerial First Person View Android app is complete, capable of viewing your DJI Drone's video feed, as well as remotely taking picture and videos!

### 4.Viewing your images

Unfortunately, this tutorial does not include guidance on viewing photos and videos onboard your DJI Drone's SD card. However, if you would like to see the pictures and videos you took through your brand new app, you can download DJI's Pilot App, found here:
<https://play.google.com/store/apps/details?id=dji.pilot&hl=en>
Alternatively, you can search for the app in the Google Play Store under the name 'DJI Pilot'. 


### 5.Where to Go From Here?

You’ve come a long way in this tutorial: you’ve learned how to use the DJI Mobile SDK to show the FPV view of the aircraft's camera and control the camera of a DJI platform. These features, **Capture** and **Record** are the most basic and common features in a typical drone mobile app. However, if you want to create a drone app that is more fancy, you still have a long way to go. More advanced features would include previewing the photo and video in the SD Card, showing the OSD data of the aircraft and so on. Hope you enjoyed this tutorial, stay tuned for our next one!

