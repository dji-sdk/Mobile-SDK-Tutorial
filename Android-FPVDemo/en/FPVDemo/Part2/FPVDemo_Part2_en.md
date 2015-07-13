# DJI Mobile Android SDK Tutorial (2)

## How to create a simpel FPV app: Part 2/2

In part 1, we have implemented the basic FPV view and can see the live video stream from the the aicraft's camera. In this tutorial, we are using Inspire 1 for demo, you will learn how to implement features such as taking photos and video recoding in this app.

### 1. Implement the capture function

A **private void captureAction()** function is implemented to taking photos. Whenever the  "Capture" button is pressed, the function will be called and a photo will be taken. The codes are as follows,
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
There are two stages in the **captureAction()** method, the first is to set the camera mode, the second is to set the camera capture mode and take a photo. We presents the code for the second step in **onResult()** to make sure the second stage runs after the first stage is executed successfully.

Build and run the project and try the capture function, if the screen flashes after your press the capture button, it shows that capture feature is functioning. 


### 2. Implement the recording function

Similar to the implementation of the capture function, a **private void recordAction()** function is implemented for recording videos. When a user presses the button "Record", the function will be called and the app starts to record a video. The codes are as follows,
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
### 3. Implement the stopping recording function

a **private void stopRecord()** function is implemented to stop the recording of the videos. When a user presses the button "Stop recording", the function will be called and the app stops recording a video. The codes are as follows,
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

Now, we can build and run the project and verify the functions. You may try out the Record function. If a screenshoot similar to 
![recordVideoScreenShot](https://github.com/dji-sdk/Mobile-SDK-Tutorial/raw/master/Android-FPVDemo/en/images/recordVideo.png)

Congratualtions! Your Aerial FPV Android app is done, you can use this app to control your Inspire 1's camera now.

### 4. Go to Next Tutorial

You can download the final project for this tutorial here: <https://github.com/DJI-Mobile-SDK/Android-FPVDemo-Part2>

You’ve come a long way in this tutorial: you’ve learned how to use DJI Mobile SDK to show the FPV View from the drone's camera and control the camera of DJI's drones, they are the most used and basic features in a simple mobile app for the aircraft: Capture and Record. But, in order to make a cooler drone app, you still have a long way to go, such as previewing the photo and video in the SD Card, showing the OSD data of the aircraft and so on.  