# DJI Mobile iOS SDK Tutorial
 

# How to create a simple GroundStation App: Part 1/2
In this tutorial, you will learn how to setup the DJI PC Simulator, upgrade your Inspire 1, Phantom 3 Professional and Phantom 3 Advanced's firmware to the beta version, and how to test the GroundStation API with DJI PC Simulator. Also, you will get in touch with the basic process of using DJI GroundStation's Waypoint feature. So let's get started!

## 1. Using DJI PC Simulator

#### **1**. **Introduction**

The DJI PC Simulator is a flight simulator designed for SDK developers. The simulator creates a virtual 3D environment and a data analysis from flight data transmitted to the PC via the UDP protocol.

**Supported Operating Systems**: Windows 7, Windows 8 and Windows 8.1

**Supported DJI Platforms**: Matrice 100, Inspire 1, Phantom 3 Professional and Phantom 3 Advanced

#### **2**. **Install and setup DJI PC Simulator**

Firstly, you should download the DJI PC Simulator and WIN Driver from here: <http://dev.dji.com/en/products/sdk/onboard-sdk/downloads> :

- DJI PC Simulator Installer & User Manual V1.0
- WIN Driver Installer

You must install the driver before running the simulator. Since the simulator only supports Windows now, you should find a PC or install a Virtual Manchine(Like VMWare or Parallels Desktop) running Windows on your Mac. Now double tap the **DJI_WIN_Driver_Installer.exe** file to install it. If there is a dialog "Please power on MC and connect it to PC via USB!" pops up, just ignore it and click "YES" and follow the instructions to install the driver. 

Then, double click the **DJISimulator-Installer.exe** file and follow the instructions to install the program. 

#### **3**. **How to use DJI PC Simulator**

**1**. The Simulator Config window will appear once you launch the DJI PC Simulator. Set the Latitude and Longitude values based on your preference. The "**SN**" represents the connected aircraft's SN number.

 ![Config](../../images/simulator_config.png)

---
**Note**: 

- The aircraft will not take off if the latitude and longitude values are near a No Fly Zone.

- Select Show Log Window under the Log Settings tab to display the log window as below:

 ![showLog](../../images/showLog.png)

---

**2**. Connect the aircraft to your PC via a Micro USB cable, and then power on the aircraft and the
remote controller. Click Display Simulator. You can see the screenshot as below:

 ![display](../../images/display.png)

---
**Note**: 

- Do NOT launch the DJI Pilot app when the DJI PC Simulator is running.

- Do NOT mount the propellers on the aircraft when the DJI PC Simulator is running in case the motors start by accident, it's dangerous.

---

**3**. Start the simulation by clicking Start Simulation. You can use the remote controller to change the course of the aircraft or bring it back with the Return-to-Home function. Enable API Control to allow control from a mobile or onboard device. World X, Y, Z represents the North-South, East- West, and Up-Down axes (take the North, East and Up directions as positive). 

**4**. Left-click and drag to change the view angle. Scroll to zoom in and zoom out.

 ![zoomIn](../../images/zoomIn.png)
 
 ![zoomOut](../../images/zoomout.png)

**5**. Click Stop Simulation to stop the simulation. Close the simulator, and power off the aircraft and the remote controller after use. 

**Important**: If you want to stop midway through the GroundStation Waypoint Mission, you should click **Stop Simulation** firstly, otherwise the simulator may run the previous groundstation mission when you start it again, it may be confused.

For more info about DJI PC Simulator, please check the **DJI PC Simulator user manual.pdf** file, which you download along with the simulator.


## 2. Upgrade Firmware

It's important to make sure your aircraft's firmware supports DJI Mobile SDK's development before you going through the following steps. Please download the beta version firmware from here according to your aircraft's type: <http://dev.dji.com/en/products/sdk/onboard-sdk/downloads> :

- Phantom 3 Professional Firmware (P3X_FW_V01.01.1003.bin)
- Inspire 1 Firmware (WM610_FW_V01.02.01.02.bin)

It's not necessary to upgrade the Remote Controller's firmware, just put the download **bin** file in the SD Card and insert it to your aircraft's camera and restart it to upgrade. It may take 10 ~ 30 minutes to finish it.

You can check the firmware upgrade status by identifying the sound from the aircraft:

- Upgrading: B B B ...
- Upgrade Success: B BB...
- Upgrade Failed: B...
- Serious Error: D D D...

Also, you can check the firmware upgrade status by checking the **txt** file generated during the upgrade process. For Phantom 3 Professional, the txt file is named as **"P3X_FW_RESULT_AB.txt"**, For Inspire 1, it's named as **"WM610_FW_RESULT_AB.txt"**, here are the example contents:

  ![upgradeP3XSuccess](../../images/upgradeP3XSuccess.png)
  
  ![upgradeInspire1Success](../../images/upgradeInspire1Success.png)

## 3. Setup Map View

#### **1**. **Import Framework and Libraries**

As you finish the above steps, we can now start to work on the app. Since in our previous tutorial: [**How to create a simple FPV app**](http://gitlab.djicorp.com/SDKDemo/FPVDemo-Part1/blob/master/Tutorial/FPVDemo_Part1_en.md), you have learnt how to import and activate DJI Mobile SDK in your Xcode project, if you haven't read that before, please check it. Now Let's setup the Map View firstly. 

**1**. Create a new project in Xcode and named it as "**GSDemo**", copy the **DJISDK.framework** to your Xcode's project folder. And then select the project target and go to Build Phases -> Link Binary With Libraries. Click the "+" button at the bottom and add libstdc++.6.0.9.dylib and libz.dylib to your project. Like the screenshot below:

  ![framework](../../images/framework.png)

#### **2**. Create Map View
Now, let's delete the **ViewController.h** and **ViewController.m** files, which were created by Xcode when you create the project. Then create a viewController named as "**DJIRootViewController**", set it as the **Root View Controller** in **Main.storyboard**. Moreover, drag a MKMapView from Object Library to **DJIRootViewController**, setup its AutoLayout constraints, and set its delegate to **DJIRootViewController** as below:
   
![mkMapView](../../images/mkMapView.png)

After it, let's open **DJIRootViewController.h** file, create an IBOutlet for MKMapView, named it as "**mapView**" and link it to the MKMapView in **Main.storyboard**. Import the following header files and implement MKMapView's delegate method:

~~~objc
#import <DJISDK/DJISDK.h>
#import <MapKit/MapKit.h>

@interface DJIRootViewController : UIViewController<MKMapViewDelegate>

@property (weak, nonatomic) IBOutlet MKMapView *mapView;

@end
~~~

Now let's build and run the project, if everything is fine, you should see the following screenshot:

![mapView](../../images/mapView.png)

#### **3**. Add Annotations to MapView

So far the map view is too simple, let's add something interesting to it. Firstly, create a new **NSObject** file named as **DJIMapController**, it's used to deal with MKAnnotations(or say Waypoints) logics on the map. Open **DJIMapController.h** file and add the following codes to it:

~~~objc
#import <UIKit/UIKit.h>
#import <MapKit/MapKit.h>

@interface DJIMapController : NSObject

@property (strong, nonatomic) NSMutableArray *editPoints;

/**
 *  Add Waypoints in Map View
 */
- (void)addPoint:(CGPoint)point withMapView:(MKMapView *)mapView;

/**
 *  Clean All Waypoints in Map View
 */
- (void)cleanAllPointsWithMapView:(MKMapView *)mapView;

/**
 *  Current Edit Points
 *
 *  @return Return an NSArray contains multiple CCLocation objects
 */
- (NSArray *)wayPoints;


@end
~~~

Here we create an NSMutableArray **editPoints** to store waypoint objects, and add two methods to implement **Add** and **Remove** waypoint methods. For the last method, we use it to return the current waypoint objects on the map.

Let's come to **DJIMapController.m** file and replace the original codes with the followings:

~~~objc
#import "DJIMapController.h"

@implementation DJIMapController

- (instancetype)init
{
    if (self = [super init]) {
        self.editPoints = [[NSMutableArray alloc] init];
    }
    return self;
}

- (void)addPoint:(CGPoint)point withMapView:(MKMapView *)mapView
{
    CLLocationCoordinate2D coordinate = [mapView convertPoint:point toCoordinateFromView:mapView];
    CLLocation *location = [[CLLocation alloc] initWithLatitude:coordinate.latitude longitude:coordinate.longitude];
    [_editPoints addObject:location];
    MKPointAnnotation* annotation = [[MKPointAnnotation alloc] init];
    annotation.coordinate = location.coordinate;
    [mapView addAnnotation:annotation];
}

- (void)cleanAllPointsWithMapView:(MKMapView *)mapView
{
    [_editPoints removeAllObjects];
    NSArray* annos = [NSArray arrayWithArray:mapView.annotations];
    for (int i = 0; i < annos.count; i++) {
        id<MKAnnotation> ann = [annos objectAtIndex:i];
        [mapView removeAnnotation:ann];
    }   
}

- (NSArray *)wayPoints
{
    return self.editPoints;
}

@end
~~~
Firstly, we init the **editPoints** array in the **init** method, then we create **MKPointAnnotation** objects from **CGPoint** and add them to our **mapView**. Next, implement the **cleanAllPointsWithMapView** method to clean up **eidtPoints** array and the annotations on the mapView.

Go back to DJIRootViewController.h file, import the DJIMapController.h header file and create a DJIMapController's property named as "**mapController**". Since we want to add annotation pins by tapping on the map, we also need to create a **UITapGestureRecognizer** named as "**tapGesture**". Lastly, add an **UIButton** to DJIRootViewController's scene in **Main.storyboard**, set its IBOutlet named as "**editBtn**" and add its IBAction method named as "**editBtnAction**" as below:

~~~objc
@property (nonatomic, strong) DJIMapController *mapController;
@property (nonatomic, strong) UITapGestureRecognizer *tapGesture;
@property (weak, nonatomic) IBOutlet UIButton *editBtn;

- (IBAction)editBtnAction:(id)sender;
~~~

![editButton](../../images/editButton.png)


Once you're done, open **DJIMapController.m** file, init **mapController** and **tapGesture**, then add the **tapGesture** to mapView. Furthermore, we need a boolean variable named as "**isEditingPoints**" to store the edit waypoint state, which will also change **editBtn**'s title. Lastly, implement tapGesture's action method **addWayPoints** as below:

~~~objc
#import "DJIRootViewController.h"

@interface DJIRootViewController ()
@property (nonatomic, assign)BOOL isEditingPoints;
@end

@implementation DJIRootViewController
- (void)viewDidLoad {
    [super viewDidLoad];
    
    self.mapController = [[DJIMapController alloc] init];
    self.tapGesture = [[UITapGestureRecognizer alloc] initWithTarget:self action:@selector(addWaypoints:)];
    [self.mapView addGestureRecognizer:self.tapGesture];
    
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(registerAppSuccess:) name:@"RegisterAppSuccess" object:nil];
}

#pragma mark Custom Methods

- (void)registerAppSuccess:(NSNotification *)notification
{

}

- (void)addWaypoints:(UITapGestureRecognizer *)tapGesture
{
    CGPoint point = [tapGesture locationInView:self.mapView];
    
    if(tapGesture.state == UIGestureRecognizerStateEnded){

        if (self.isEditingPoints) {
            [self.mapController addPoint:point withMapView:self.mapView];
        }
    }
}

- (IBAction)editBtnAction:(id)sender {

    if (self.isEditingPoints) {
        [self.mapController cleanAllPointsWithMapView:self.mapView];
        [self.editBtn setTitle:@"Edit" forState:UIControlStateNormal];
    }else
    {
        [self.editBtn setTitle:@"Reset" forState:UIControlStateNormal];
    }
    
    self.isEditingPoints = !self.isEditingPoints;
    
}

#pragma mark MKMapViewDelegate Method
- (MKAnnotationView *)mapView:(MKMapView *)mapView viewForAnnotation:(id <MKAnnotation>)annotation
{
    if ([annotation isKindOfClass:[MKPointAnnotation class]]) {
        MKPinAnnotationView* pinView = [[MKPinAnnotationView alloc] initWithAnnotation:annotation reuseIdentifier:@"Pin_Annotation"];
        pinView.pinColor = MKPinAnnotationColorPurple;
        return pinView;
        
    }
    
    return nil;
}

~~~

For the above code, we also add an NSNotification observer to check the DJI Mobile SDK's Register App state. At the same time, we implement the **addWaypoints** gesture action by calling DJIMapController's 

     - (void)addPoint:(CGPoint)point withMapView:(MKMapView *)mapView
method to add waypoints to the map. Moreover, we implement **editBtn**'s IBAction method, it will update the button's title and clean up waypoints based on **isEditingPoints**'s value. Lastly, we implement MKMapViewDelegate's method to change the Annotation's pin color to purple.

When you are done all the steps above, build and run your project and try to add waypoints on the map. If everything is fine, you will see the following screenshot:

![waypoint1](../../images/waypoint1.jpg)

![waypoint2](../../images/waypoint2.jpg)

#### **4**. Focus MKMapView

You may be wondering why the map's location is different from your current location and it's difficult to find yours in the map. If we can focus the map to your current location quickly, that would be helpful. To implement it, we need to use **CLLocationManager**.

Open **DJIRootViewController.h** file, import CoreLocation's header file firstly. Create a **CLLocationManager** property named as "**locationManager**". Then create a **CLLocationCoordinate2D** property named as "**userLocation**" to store user's location data. Next, implement **CLLocationManager**'s **CLLocationManagerDelegate** protocol in the class as below:

~~~objc
#import <DJISDK/DJISDK.h>
#import <MapKit/MapKit.h>
#import <CoreLocation/CoreLocation.h>

@interface DJIRootViewController : UIViewController<MKMapViewDelegate, CLLocationManagerDelegate>

@property (nonatomic, weak) IBOutlet MKMapView *mapView;
@property (nonatomic, strong) CLLocationManager* locationManager;
@property (nonatomic, assign) CLLocationCoordinate2D userLocation;
@property (nonatomic, strong) UITapGestureRecognizer *tapGesture;
@property (nonatomic, weak) IBOutlet UIButton *editBtn;

- (IBAction)editBtnAction:(id)sender;
- (IBAction)focusMapAction:(id)sender;

@end
~~~

In the above code, we also add a UIButton named as "**Focus Map**" in DJIRootViewController's scene of **Main.storyboard** and add its IBAction method named as "**focusMapAction**", here is the screenshot of **Main.storyboard**'s scene:

![focusMap](../../images/focusMap.png)

Once you are done, go to **DJIRootViewController.m** file and add the following codes as below:

~~~objc
- (void)viewWillAppear:(BOOL)animated
{
    [super viewWillAppear:animated];
    [self startUpdateLocation];
}

- (void)viewWillDisappear:(BOOL)animated
{
    [super viewWillDisappear:animated];
    [self.locationManager stopUpdatingLocation];
}

- (void)viewDidLoad {
    [super viewDidLoad];
    
    self.userLocation = kCLLocationCoordinate2DInvalid;
    
    self.mapController = [[DJIMapController alloc] init];
    self.tapGesture = [[UITapGestureRecognizer alloc] initWithTarget:self action:@selector(addWaypoints:)];
    [self.mapView addGestureRecognizer:self.tapGesture];
    
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(registerAppSuccess:) name:@"RegisterAppSuccess" object:nil];
    
}

#pragma mark CLLocation Methods
-(void) startUpdateLocation
{
    if ([CLLocationManager locationServicesEnabled]) {
        if (self.locationManager == nil) {
            self.locationManager = [[CLLocationManager alloc] init];
            self.locationManager.delegate = self;
            self.locationManager.desiredAccuracy = kCLLocationAccuracyBest;
            self.locationManager.distanceFilter = 0.1;
            if ([self.locationManager respondsToSelector:@selector(requestAlwaysAuthorization)]) {
                [self.locationManager requestAlwaysAuthorization];
            }
            [self.locationManager startUpdatingLocation];
        }
    }else
    {
        UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"Location Service is not available" message:@"" delegate:self cancelButtonTitle:@"OK" otherButtonTitles:nil];
        [alert show];
    }
}

- (IBAction)focusMapAction:(id)sender {
{
    if (CLLocationCoordinate2DIsValid(self.userLocation)) {
        MKCoordinateRegion region = {0};
        region.center = self.userLocation;
        region.span.latitudeDelta = 0.001;
        region.span.longitudeDelta = 0.001;
        
        [self.mapView setRegion:region animated:YES];
    }
}

#pragma mark - CLLocationManagerDelegate
- (void)locationManager:(CLLocationManager *)manager didUpdateLocations:(NSArray *)locations
{
    CLLocation* location = [locations lastObject];
    self.userLocation = location.coordinate;
}

~~~

Firstly, we init userLocation's data to kCLLocationCoordinate2DInvalid in the viewDidLoad method. Then we add a new method named as **startUpdateLocation** to init **locationManger**, set its properties and start update location. If the Location Service is not available, we add an UIAlertView for warning. The **startUpdateLocation** is called in **viewWillAppear** method and stop update location in **viewWillDisappear** method. Moreover, we need to implement CLLocationManagerDelegate method to update **userLocation** property. Finally, we implement "**focusMapAction**" method to focus **mapView** to user's current location.

In iOS8, we must call **locationManager**'s **requestAlwaysAuthorization** first, which was done in **startUpdateLocation** method. Then add a NSLocationAlwaysUsageDescription or NSLocationWhenInUseUsageDescription key to your project’s Info.plist containing the message to be displayed to the user at the prompt to get **CLLocationManagerDelegate** method work. We set the messages empty here:

![infoPlist](../../images/infoPlist.png)

It's time to build and run the project to check the focus map feature. If it works well, you should see the following screenshot:

![coreLocation1](../../images/coreLocation1.jpg)

When you launch the app for the first time, it should pop up an alert to ask for your permission to access your location event, just select **Allow** and press the **Focus Map** button. If the map view animate to your current location like the following screenshot, congratulations, you have finish the "Focus Map" feature!

![coreLocation2](../../images/coreLocation2.jpg)


#### **5**. Show Aircraft on Map View

Now, we can focus the mapView to our current location, it's a good start! Let's do something more interesting, simulate the aircraft's GPS location using the DJI PC Simulator and show it on our map view. Sounds cool, right?

You may know how to setup the DJI PC Simulator, and its basic usage. If you want to place the aircraft in your current GPS location on Map View, you can set the latitude and longitude values in the **Simulator Config** to yours. We take the simulator's initial values for example here.

Let's come back to the code. Create a new file of **MKAnnotationView** subclass named as "DJIAircraftAnnotationView" and another new file of **NSObject** subclass named as "DJIAircraftAnnotation". Here are the codes:

- DJIAircraftAnnotationView.h

~~~objc
#import <MapKit/MapKit.h>

@interface DJIAircraftAnnotationView : MKAnnotationView

-(void) updateHeading:(float)heading;

@end
~~~

- DJIAircraftAnnotationView.m

~~~objc
#import "DJIAircraftAnnotationView.h"

@implementation DJIAircraftAnnotationView

- (instancetype)initWithAnnotation:(id <MKAnnotation>)annotation reuseIdentifier:(NSString *)reuseIdentifier
{
    self = [super initWithAnnotation:annotation reuseIdentifier:reuseIdentifier];
    if (self) {
        self.enabled = NO;
        self.draggable = NO;
        self.image = [UIImage imageNamed:@"aircraft.png"];
    }
    
    return self;
}

-(void) updateHeading:(float)heading
{
    self.transform = CGAffineTransformIdentity;
    self.transform = CGAffineTransformMakeRotation(heading);
}

@end

~~~

In the codes above, we create a MKAnnotationView for the aircraft, we add a method "updateHeading" to change the aircraft's rotation, and set its image to "**aircraft.png**"(You can get the image from this tutorial's demo project) in the init method. Also we disable the DJIAircraftAnnotationView's draggable property.

- DJIAircraftAnnotation.h

~~~objc
#import <MapKit/MapKit.h>
#import "DJIAircraftAnnotationView.h"

@interface DJIAircraftAnnotation : NSObject<MKAnnotation>

@property(nonatomic, readonly) CLLocationCoordinate2D coordinate;
@property(nonatomic, weak) DJIAircraftAnnotationView* annotationView;

-(id) initWithCoordiante:(CLLocationCoordinate2D)coordinate;

-(void)setCoordinate:(CLLocationCoordinate2D)newCoordinate;

-(void) updateHeading:(float)heading;

@end
~~~

- DJIAircraftAnnotation.m

~~~objc
#import "DJIAircraftAnnotation.h"

@implementation DJIAircraftAnnotation

-(id) initWithCoordiante:(CLLocationCoordinate2D)coordinate
{
    self = [super init];
    if (self) {
        _coordinate = coordinate;
    }   
    return self;
}

- (void)setCoordinate:(CLLocationCoordinate2D)newCoordinate
{
    _coordinate = newCoordinate;
}

-(void)updateHeading:(float)heading
{
    if (self.annotationView) {
        [self.annotationView updateHeading:heading];
    }
}
@end
~~~

For **DJIAircraftAnnotation** class, it implements the **MKAnnotation** protocol. It's used to store and update a CLLocationCoordinate2D property. Also we can update DJIAircraftAnnotationView's heading with the **updateHeading** method.

Once you're done. Open the **DJIMapController.h** file and import the **DJIAircraftAnnotation.h** file:

~~~objc
#import "DJIAircraftAnnotation.h"
~~~

Then create a DJIAircraftAnnotation's property and named it as "aircraftAnnotation". 

~~~objc
@property (nonatomic, strong) DJIAircraftAnnotation* aircraftAnnotation;
~~~

Furthermore, add two new methods to update the aircraft's location on the map and also its heading.

~~~objc
/**
 *  Update Aircraft's location in Map View
 */
-(void)updateAircraftLocation:(CLLocationCoordinate2D)location withMapView:(MKMapView *)mapView;

/**
 *  Update Aircraft's heading in Map View
 */
-(void)updateAircraftHeading:(float)heading;
~~~

Next, let's come to DJIMapController.m file and implement the above two methods:

~~~objc
-(void)updateAircraftLocation:(CLLocationCoordinate2D)location withMapView:(MKMapView *)mapView
{
    if (self.aircraftAnnotation == nil) {
        self.aircraftAnnotation = [[DJIAircraftAnnotation alloc] initWithCoordiante:location];
        [mapView addAnnotation:self.aircraftAnnotation];
    }
    
    [self.aircraftAnnotation setCoordinate:location];
}

-(void)updateAircraftHeading:(float)heading
{
    if (self.aircraftAnnotation) {
        [self.aircraftAnnotation updateHeading:heading];
    }
}
~~~

Also, since we don't want the **aircraftAnnotation** removed by the **cleanAllPointsWithMapView** method in the DJIMapController.m file, we need to modify it as below:

~~~objc
- (void)cleanAllPointsWithMapView:(MKMapView *)mapView
{
    [_editPoints removeAllObjects];
    NSArray* annos = [NSArray arrayWithArray:mapView.annotations];
    for (int i = 0; i < annos.count; i++) {
        id<MKAnnotation> ann = [annos objectAtIndex:i];
        if (![ann isEqual:self.aircraftAnnotation]) {
            [mapView removeAnnotation:ann];
        }
       
    }   
}
~~~
We add an if statement to check if the annotation of the map view's is equal to the **aircraftAnnotation** property, if not, just remove it. By doing so, we can prevent the Aircraft's annotation from removing.

To provide a better user experience, we need to add a status view on top of the mapView to show the aircraft's flight mode type, current GPS satellite count, vertical and horizontal flight speed and the flight altitude. Let's add the UI in **Main.storyboard**'s RootViewController Scene as below:

![statusView](../../images/statusView.png)

Once that's done, open **DJIRootViewController.h** file, create IBOutlets for the above UI elements. Then import DJISDK's header file and implement **DJIDroneDelegate** and **DJIMainControllerDelegate** protocol, here we use Inspire 1 for the demo, so we need to create a **DJIDrone**'s property and a **DJIInspireMainController**'s property. Also, we need to create a **CLLocationCoordinate2D** property named as "droneLocation" to record the aircraft's location as below:

~~~objc
#import <DJISDK/DJISDK.h>
@interface DJIRootViewController : UIViewController<MKMapViewDelegate, CLLocationManagerDelegate, DJIDroneDelegate, DJIMainControllerDelegate>

@property(nonatomic, strong) IBOutlet UILabel* modeLabel;
@property(nonatomic, strong) IBOutlet UILabel* gpsLabel;
@property(nonatomic, strong) IBOutlet UILabel* hsLabel;
@property(nonatomic, strong) IBOutlet UILabel* vsLabel;
@property(nonatomic, strong) IBOutlet UILabel* altitudeLabel;

@property(nonatomic, strong) DJIDrone* inspireDrone;
@property(nonatomic, strong) DJIInspireMainController* inspireMainController;
@property(nonatomic, assign) CLLocationCoordinate2D droneLocation;
~~~

Then let's move on to **DJIRootViewController.m** file and init the UI elements' values firstly. Add the **initUI** method and call it in the **viewDidLoad** method. Also, add an **initDrone** method to init **inspireDrone** proptery as below and call it in the **viewDidLoad** method too:

~~~objc

-(void) initUI
{
    self.modeLabel.text = @"N/A";
    self.gpsLabel.text = @"0";
    self.vsLabel.text = @"0.0 M/S";
    self.hsLabel.text = @"0.0 M/S";
    self.altitudeLabel.text = @"0 M";
}

- (void)initDrone
{
    self.inspireDrone = [[DJIDrone alloc] initWithType:DJIDrone_Inspire];
    self.inspireMainController = (DJIInspireMainController*)self.inspireDrone.mainController;
    self.inspireDrone.delegate = self;
    self.inspireMainController.mcDelegate = self;
}
~~~

The **DJIInspireMainController** subclass from **DJIMainController**， it's a mainController to control the aircraft, like get the DJIMCSystemState, taking off, landing, turning on motor and so on. You check its header file in the SDK for more info. Here we set **inspireDrone** and **inspireMainController** properties' delegates as self.

Remember the **registerAppSuccess** method we've added previously? Let's implement it like this:

~~~objc
#pragma mark NSNotification Selector Method
- (void)registerAppSuccess:(NSNotification *)notification
{
    [self.inspireDrone connectToDrone];
    [self.inspireMainController startUpdateMCSystemState];
}
~~~

When the app is registerd successfully, we can call DJIDrone's **connectToDrone** method to connect to the aircraft, and call **startUpdateMCSystemState** method to start update the aircraft's MC system state, we need these state data to update our aircraft's location and heading. Moreover, in the **viewWillDisappear** method, we need to disconnect with the drone:

~~~objc
- (void)viewWillDisappear:(BOOL)animated
{
    [super viewWillDisappear:animated];

    [self.locationManager stopUpdatingLocation];
    [self.inspireDrone disconnectToDrone];
}
~~~

In **viewDidLoad** method, assign the **droneLocation** property's value as kCLLocationCoordinate2DInvalid. And update the **focusMap** action method to set **droneLocation** as the center of map view's region as below:

~~~objc
- (IBAction)focusMapAction:(id)sender {

    if (CLLocationCoordinate2DIsValid(self.droneLocation)) {
        MKCoordinateRegion region = {0};
        region.center = self.droneLocation;
        region.span.latitudeDelta = 0.001;
        region.span.longitudeDelta = 0.001;
        [self.mapView setRegion:region animated:YES];
    }

}
~~~

Next, We need to modify the MKMapViewDelegate method like this:

~~~objc
- (MKAnnotationView *)mapView:(MKMapView *)mapView viewForAnnotation:(id <MKAnnotation>)annotation
{
    if ([annotation isKindOfClass:[MKPointAnnotation class]]) {
        MKPinAnnotationView* pinView = [[MKPinAnnotationView alloc] initWithAnnotation:annotation reuseIdentifier:@"Pin_Annotation"];
        pinView.pinColor = MKPinAnnotationColorPurple;
        return pinView;
        
    }else if ([annotation isKindOfClass:[DJIAircraftAnnotation class]])
    {
        DJIAircraftAnnotationView* annoView = [[DJIAircraftAnnotationView alloc] initWithAnnotation:annotation reuseIdentifier:@"Aircraft_Annotation"];
        ((DJIAircraftAnnotation*)annotation).annotationView = annoView;
        return annoView;
    }
    
    return nil;
}
~~~
It will check the annotation variable's class and set its annotationView as a **DJIAircraftAnnotationView** Class type object.

Furthermore, let's implement DJIMainControllerDelegate method:

~~~objc
#pragma mark - DJIMainControllerDelegate Method

-(void) mainController:(DJIMainController*)mc didUpdateSystemState:(DJIMCSystemState*)state
{
    self.droneLocation = state.droneLocation;
    DJIInspireMCSystemState* inspireState = (DJIInspireMCSystemState*)state;
    if (!inspireState.canIOCWork) {
        [self.inspireMainController setFlightModeSwitchable:YES withResult:nil];
    }
    if (inspireState.isIOCWorking) {
        [self.inspireMainController setIOCWorking:NO withResult:nil];
    }
    
     self.modeLabel.text = inspireState.flightModeString;
    self.gpsLabel.text = [NSString stringWithFormat:@"%d", inspireState.satelliteCount];
    self.vsLabel.text = [NSString stringWithFormat:@"%0.1f M/S",inspireState.velocityZ];
    self.hsLabel.text = [NSString stringWithFormat:@"%0.1f M/S",(sqrtf(inspireState.velocityX*inspireState.velocityX + inspireState.velocityY*inspireState.velocityY))];
    self.altitudeLabel.text = [NSString stringWithFormat:@"%0.1f M",inspireState.altitude];
        
    [self.mapController updateAircraftLocation:self.droneLocation withMapView:self.mapView];
    double radianYaw = (state.attitude.yaw * M_PI / 180.0);
    [self.mapController updateAircraftHeading:radianYaw];
    
}
~~~

Firstly, it will update the **droneLocation** with the aircraft's current location. Then disable inspireMainController's IOC function.

***
**Important**: Since there are conflicts between DJI Mobile SDK's GroundStation and IOC, you cannot use them at the same time. So please make sure to close the IOC feature before using GroundStation. Otherwise, you will run into trouble.
***

Next, update the status labels' text data from the DJIMCSystemState. Furthermore, update the aircraft's location and heading by calling the related methods of **DJIMapController**.

Finally, let's implement the DJIDroneDelegate Method as below:

~~~objc
#pragma mark - DJIDroneDelegate Method
-(void) droneOnConnectionStatusChanged:(DJIConnectionStatus)status
{
    if (status == ConnectionSuccessed) {
        [self.inspireMainController enterNavigationModeWithResult:^(DJIError *error) {
            if (error.errorCode != ERR_Successed) {
                NSString* message = [NSString stringWithFormat:@"Enter navigation mode failed:%@", error.errorDescription];
                UIAlertView* alertView = [[UIAlertView alloc] initWithTitle:@"Enter Navigation Mode" message:message delegate:self cancelButtonTitle:@"Cancel" otherButtonTitles:@"Retry", nil];
                alertView.tag = kEnterNaviModeFailedAlertTag;
                [alertView show];
            }
        }];
    }
    
}
~~~

If it succeeds to connect to the aircraft, call **DJIInspireMainController**'s "enterNavigationModeWithResult" method to check if the aircraft enter navigation mode successfully.If not, just show an UIAlertView for the user to retry. So we need to also implement UIAlertView's delegate method like this:

~~~
#pragma mark - UIAlertViewDelegate

- (void)alertView:(UIAlertView *)alertView clickedButtonAtIndex:(NSInteger)buttonIndex
{
    if (alertView.tag == 1002) {
        if (buttonIndex == 1) {
            [self.inspireMainController enterNavigationModeWithResult:^(DJIError *error) {
                if (error.errorCode != ERR_Successed) {
                    NSString* message = [NSString stringWithFormat:@"Enter navigation mode failed:%@", error.errorDescription];
                    UIAlertView* alertView = [[UIAlertView alloc] initWithTitle:@"Enter Navigation Mode" message:message delegate:self cancelButtonTitle:@"Cancel" otherButtonTitles:@"Retry", nil];
                    alertView.tag = kEnterNaviModeFailedAlertTag;
                    [alertView show];
                }
            }];
        }
    }
}
~~~

We've been gone through a long way in this tutorial. Let's test the app now. 
Build and run the project to install the app in your mobile device. After that, please connect the aircraft to your PC or Virtual Machine running Windows via a Micro USB cable, and then power on the aircraft and the remote controller. Click Display Simulator. You can type in your current location's latitude and longitude data in the Simulator Config if you want. 

![simulatorPreview](../../images/simulator_preview.png)

Then run the app and connect your mobile device to the RC using Apple's lighting cable. You may see the following screenshot:

![enterNaviModeFailed](../../images/enterNaviModeFailed.jpg)

**Important**: To fix this problem, please switch the Remote Controller's mode selection to the **F** position(Which used to be the A position in the previous version) and press **Retry** button. If the mode selection bar is in the F position when the autopilot is powered on, the user must switch back and forth and press the **Retry** button again.

You are required to do so when using the Ground Station, Hotpoint and Joystick functions in the DJI Mobile SDK.

![switchFlightMode](../../images/switchFlightMode.png)

Next, let's come to the DJI PC Simulator on your PC and press the **Start Simulation** button, now check the app and something magical happen, the tiny red aircraft is shown on the map as below:

![aircraftOnMap1](../../images/aircraftOnMap1.jpg)

If you cannot find the aircraft, you can try to press the "**Focus Map**" button, then the map view will zoom in to center the aircraft like this:

![aircraftOnMap2](../../images/aircraftOnMap2.jpg)

It looks pretty cool, right? Now, if you press the **Stop Simulation** button on the Simulator Config, the aircraft will disappear on the map. Since the simulator stop providing GPS data to the aircraft now.

## 4. Where To Go From Here?

   You can download the demo project for this tutorial from here: <https://github.com/dji-sdk/GSDemo-Part1.git>
   
   You’ve learnt how to setup and use the DJI PC Simulator to test your GroundStation app, how to upgrade your aircraft's firmware to the developer version, and how to use the DJI Mobile SDK to create an simple MapView, modify annotations of map view, showing the aircraft on the map view by simulate the GPS data from DJI PC Simulator, etc. That's a quite a lot.
   
   In the next tutorial, we will implement the basic functions of Waypoints in GroundStation. Your aircraft will fly automatically on the map view based on the waypoints you set. Please follow our next part of this tutorial, hope you enjoy it!