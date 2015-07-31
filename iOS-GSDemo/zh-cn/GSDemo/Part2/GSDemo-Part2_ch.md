# DJI Mobile iOS SDK 教程

# 如何创建一个地图和地面站航点功能App: 第二部分
在此教程里，泥浆学会如何实行地面站中航点的基本过程. 在三个功能(航点，热点，跟随我), 航点是地面站中最复杂也是最常用的功能. 如果你还没有阅读此教程的第一部分，请在进行下一步之前阅读 [here](../Part1/GSDemo-Part1_en.md). 让我们开始吧!

   你可以在此处下载此教程的demo[here](https://github.com/DJI-Mobile-SDK/iOS-GSDemo-Part2.git)
   
## 1. UI重构
在此教程第一部分中，代码的结构比较简单并且不够健壮. 为了此教程以后的开发，代码需要被重组并且我们需要加入更多的UI元素. 
#### **1**. 加入并操作新的UIButtons
首先，我们将创建一个新的文件名为 **DJIGSButtonController**的**UIViewController**的子类. 确保你在创建文件时在 **Also create XIB file** 旁打钩. 然后打开 **DJIGSButtonController.xib** 文件并将其的大小设为**Simulated Metrics**部分**Size**菜单里的 **Freeform** .在显示部分，将宽改为 **110** 并将高改为 **260**. 请看下面的改变:

![freeform](https://raw.githubusercontent.com/dji-sdk/Mobile-SDK-Tutorial/master/iOS-GSDemo/en/images/freeform.png)
![changeSize](https://raw.githubusercontent.com/dji-sdk/Mobile-SDK-Tutorial/master/iOS-GSDemo/en/images/changeSize.png)

接下来，拖动六个按钮到显示屏上并将它们的名字改为 **Edit**, **Back**, **Clear**, **Focus Map**, **Start** 和 **Stop**. **Edit** 在 **Back**上面, 而 **Focus Map** 在 **Clear**上面. 确保 **Back**, **Clear**, **Start** 和 **Stop** 按钮是隐藏状态.

![gsButtons](https://raw.githubusercontent.com/dji-sdk/Mobile-SDK-Tutorial/master/iOS-GSDemo/en/images/gsButtons.png)

 然后在为六个按钮**DJIGSButtonViewController.h**文件里加入IBOutlets 和 IBActions. 同时，我们我们将加入一个 Enum名为 **DJIGSViewMode** 与应用的两种不同的模式. 下一步，我们加入几个当按钮的IBAction方法运行时将会被delegate viewcontroller实现的 delegate 方法. 最后，加入一个 **- (void)switchToMode:(DJIGSViewMode)mode inGSButtonVC:(DJIGSButtonViewController *)GSBtnVC;** 方法来更新按钮的状态当 **DJIGSViewMode** 改变的时候changed. 请看下面的代码:
 
 ~~~objc
 #import <UIKit/UIKit.h>
 
 typedef NS_ENUM(NSUInteger, DJIGSViewMode) {
    DJIGSViewMode_ViewMode,
    DJIGSViewMode_EditMode,
};

@class DJIGSButtonViewController;

@protocol DJIGSButtonViewControllerDelegate <NSObject>

- (void)stopBtnActionInGSButtonVC:(DJIGSButtonViewController *)GSBtnVC;
- (void)clearBtnActionInGSButtonVC:(DJIGSButtonViewController *)GSBtnVC;
- (void)focusMapBtnActionInGSButtonVC:(DJIGSButtonViewController *)GSBtnVC;
- (void)startBtnActionInGSButtonVC:(DJIGSButtonViewController *)GSBtnVC;

- (void)switchToMode:(DJIGSViewMode)mode inGSButtonVC:(DJIGSButtonViewController *)GSBtnVC;
@end

@interface DJIGSButtonViewController : UIViewController

@property (weak, nonatomic) IBOutlet UIButton *backBtn;
@property (weak, nonatomic) IBOutlet UIButton *stopBtn;
@property (weak, nonatomic) IBOutlet UIButton *clearBtn;
@property (weak, nonatomic) IBOutlet UIButton *focusMapBtn;
@property (weak, nonatomic) IBOutlet UIButton *editBtn;
@property (weak, nonatomic) IBOutlet UIButton *startBtn;

@property (assign, nonatomic) DJIGSViewMode mode;
@property (weak, nonatomic) id <DJIGSButtonViewControllerDelegate> delegate;

- (IBAction)backBtnAction:(id)sender;
- (IBAction)stopBtnAction:(id)sender;
- (IBAction)clearBtnAction:(id)sender;
- (IBAction)focusMapBtnAction:(id)sender;
- (IBAction)editBtnAction:(id)sender;
- (IBAction)startBtnAction:(id)sender;

@end
 ~~~
 
 当你完成了以后，打开 **DJIGSButtonViewController.m** 文件用下面的代码来取代已有的代码:
 
 ~~~objc
 #import "DJIGSButtonViewController.h"

@implementation DJIGSButtonViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    [self setMode:DJIGSViewMode_ViewMode];
}

- (void)didReceiveMemoryWarning {
    [super didReceiveMemoryWarning];
    // Dispose of any resources that can be recreated.
}

#pragma mark - Property Method
- (void)setMode:(DJIGSViewMode)mode
{
    _mode = mode;
    [_editBtn setHidden:(mode == DJIGSViewMode_EditMode)];
    [_focusMapBtn setHidden:(mode == DJIGSViewMode_EditMode)];
    [_backBtn setHidden:(mode == DJIGSViewMode_ViewMode)];
    [_clearBtn setHidden:(mode == DJIGSViewMode_ViewMode)];
    [_startBtn setHidden:(mode == DJIGSViewMode_ViewMode)];
    [_stopBtn setHidden:(mode == DJIGSViewMode_ViewMode)];
}

#pragma mark - IBAction Methods
- (IBAction)backBtnAction:(id)sender {
    [self setMode:DJIGSViewMode_ViewMode];
    if ([_delegate respondsToSelector:@selector(switchToMode:inGSButtonVC:)]) {
        [_delegate switchToMode:self.mode inGSButtonVC:self];
    }
}

- (IBAction)stopBtnAction:(id)sender {
 
    if ([_delegate respondsToSelector:@selector(stopBtnActionInGSButtonVC:)]) {
        [_delegate stopBtnActionInGSButtonVC:self];
    }
}

- (IBAction)clearBtnAction:(id)sender {
    
    if ([_delegate respondsToSelector:@selector(clearBtnActionInGSButtonVC:)]) {
        [_delegate clearBtnActionInGSButtonVC:self];
    }
}

- (IBAction)focusMapBtnAction:(id)sender {
    
    if ([_delegate respondsToSelector:@selector(focusMapBtnActionInGSButtonVC:)]) {
        [_delegate focusMapBtnActionInGSButtonVC:self];
    }
}

- (IBAction)editBtnAction:(id)sender {
    
    [self setMode:DJIGSViewMode_EditMode];
    if ([_delegate respondsToSelector:@selector(switchToMode:inGSButtonVC:)]) {
        [_delegate switchToMode:self.mode inGSButtonVC:self];
    }
}

- (IBAction)startBtnAction:(id)sender {
    
    if ([_delegate respondsToSelector:@selector(startBtnActionInGSButtonVC:)]) {
        [_delegate startBtnActionInGSButtonVC:self];
    }
}

@end
 ~~~
 
 这些改变是的代码建构更加简洁与健壮，这将会方便以后的维护.
 
 现在，让我们来到 **DJIRootViewController.h** 文件并删除 **editButton** IBOutlet,  **resetPointsAction** 方法, 以及  **focusMapAction** 方法. 删除这些以后，创造一个UIView IBOutlet 名为 "topBarView" 并将其连接到 **Main.storyboard**的 RootViewController的黄面, 如下所示:
 
 ![topBarView](https://raw.githubusercontent.com/dji-sdk/Mobile-SDK-Tutorial/master/iOS-GSDemo/en/images/topBarView.png)
 
 然后打开 **DJIRootViewController.m** 文件, 导入 **DJIGSButtonViewController.h** 头文件. 创造一个 **DJIGSButtonViewController**类型的属性并命名为 **gsButtonVC** 然后在类里实现 **DJIGSButtonViewController**的 **DJIGSButtonViewControllerDelegate** 协议:
 
~~~objc
#import "DJIRootViewController.h"
#import "DJIGSButtonViewController.h"

#define kEnterNaviModeFailedAlertTag 1001

@interface DJIRootViewController ()<DJIGSButtonViewControllerDelegate>
@property (nonatomic, assign)BOOL isEditingPoints;
@property (nonatomic, strong)DJIGSButtonViewController *gsButtonVC;
@end
~~~

下一步，初始化**initUI**方法里的 **gsButtonVC** 属性并将原本的 **focusMapAction** 方法的内容移到一个名为 **focusMap**的方法里, 如下所示:

~~~objc
self.gsButtonVC = [[DJIGSButtonViewController alloc] initWithNibName:@"DJIGSButtonViewController" bundle:[NSBundle mainBundle]];
[self.gsButtonVC.view setFrame:CGRectMake(0, self.topBarView.frame.origin.y + self.topBarView.frame.size.height, self.gsButtonVC.view.frame.size.width, self.gsButtonVC.view.frame.size.height)];
self.gsButtonVC.delegate = self;
[self.view addSubview:self.gsButtonVC.view];
~~~

~~~objc
- (void)focusMap
{
    if (CLLocationCoordinate2DIsValid(self.droneLocation)) {
        MKCoordinateRegion region = {0};
        region.center = self.droneLocation;
        region.span.latitudeDelta = 0.001;
        region.span.longitudeDelta = 0.001;
        
        [self.mapView setRegion:region animated:YES];
    }
}
~~~

最后，实现 **DJIGSButtonViewController**的 delegate 方法, 如下所示:

~~~objc
#pragma mark - DJIGSButtonViewController Delegate Methods
- (void)stopBtnActionInGSButtonVC:(DJIGSButtonViewController *)GSBtnVC
{
}

- (void)clearBtnActionInGSButtonVC:(DJIGSButtonViewController *)GSBtnVC
{
    [self.mapController cleanAllPointsWithMapView:self.mapView];
}

- (void)focusMapBtnActionInGSButtonVC:(DJIGSButtonViewController *)GSBtnVC
{
    [self focusMap];
}

- (void)startBtnActionInGSButtonVC:(DJIGSButtonViewController *)GSBtnVC
{
}

- (void)switchToMode:(DJIGSViewMode)mode inGSButtonVC:(DJIGSButtonViewController *)GSBtnVC
{
    if (mode == DJIGSViewMode_EditMode) {
         self.isEditingPoints = YES;
        [self focusMap];
    }else
    {
         self.isEditingPoints = YES;
    }
}
~~~

在 **- (void)switchToMode:(DJIGSViewMode)mode inGSButtonVC:(DJIGSButtonViewController *)GSBtnVC** delegate方法里, 我们运行 **focusMap** 方法. 这样的话，我们呢挂在按下edit见识将地图焦点集中在飞机的位置，免去用户每次都需要放大到edit并使其更容易使用. 同时，当应用在edit 模式下,  **isEditingPoints**属性被设置为 **YES**. 

现在，让我们建造并运行此project并尝试按下**Edit** 以及 **Back** 按钮. 以下是当你按下按钮是的截图:

![GSButtons1](https://raw.githubusercontent.com/dji-sdk/Mobile-SDK-Tutorial/master/iOS-GSDemo/en/images/GSButtons1.jpg)
![GSButtons2](https://raw.githubusercontent.com/dji-sdk/Mobile-SDK-Tutorial/master/iOS-GSDemo/en/images/GSButtons2.jpg)

## 2. 调试 DJIGroundStationWaypoint 和 The DJIGroundStationTask

#### **1**. DJIGroundStationWaypoint

让我们来到 **DJIGroundStationWaypoint.h** 文件看一看. 比如说，你可以使用: 

~~~objc
-(id) initWithCoordinate:(CLLocationCoordinate2D)coordinate;
~~~
来创造一个航点实例与一个详细的坐标. 当你创造了航点时，你可以向其添加一个 **DJIWaypointAction** 如果你运行下面的代码:

~~~objc
-(BOOL) addWaypointAction:(DJIWaypointAction*)action;
~~~

更多的是，有了航点，你可以设置坐标，高度，方向，水平速度还有很多很多东西. 更多细节，请查看*DJIGroundStationWaypoint.h** 头文件.

#### **2**. DJIGroundStationTask

当你想要开始一个GroundStation Waypoint 工作时，**DJIGroundStationTask** 将被使用. Y你可以运它的类方法 **+(id) newTask;** 来直接创建一个新的任务. 当你创造了任务，你可以使用下面的方法来加入 **DJIGroundStationWaypoint** 类型的航点: 

~~~objc
-(void) addWaypoint:(DJIGroundStationWaypoint*)waypoint;
~~~

On the contrary, you can also delete waypoints from a task by using the method: 

~~~objc
-(void) removeWaypoint:(DJIGroundStationWaypoint*)waypoint;
~~~
 method.
 
 更多的是。你可以设置**DJIGroundStationTask**的 **isLoop** 属性来决定是否循环执行任务. 同时，你可以设置**DJIGSTaskFinishedAction** enum 类型的 **finishedAction** 属性来调整飞机的动作当任务完成以后. 最后，你可以设置**DJIGSHeadingMode** enum类型的 **headingMode** 属性来调整飞机执行任务时的方向. 下面是一部分头文件:
 
~~~objc
/**
 *  Action of task when finished
 */
typedef NS_ENUM(NSUInteger, DJIGSTaskFinishedAction){
    /**
     *  No action. aircraft will stay at the last waypoint
     */
    GSTaskFinishedNoAction,
    /**
     *  Aircraft will go home
     */
    GSTaskFinishedGoHome,
    /**
     *  Aircraft will auto landing
     */
    GSTaskFinishedAutoLanding,
    /**
     *  Aircraft will go to the first waypoint
     */
    GSTaskFinishedGoFirstWaypoint
};

/**
 *  Heading mode
 */
typedef NS_ENUM(NSUInteger, DJIGSHeadingMode){
    /**
     *  Aircraft's heading toward to the next waypoint
     */
    GSHeadingTowardNexWaypoint,
    /**
     *  Aircraft's heading using the initial direction
     */
    GSHeadingUsingInitialDirection,
    /**
     *  Aircraft's heading control by the remote controller
     */
    GSHeadingControlByRemoteController,
    /**
     *  Aircraft's heading using the waypoint's heading value
     */
    GSHeadingUsingWaypointHeading,
};

/**
 *  Whether execute task looply. Default is NO
 */
@property(nonatomic, assign) BOOL isLoop;

/**
 *  Action for the aircraft while the task finished
 */
@property(nonatomic, assign) DJIGSTaskFinishedAction finishedAction;

/**
 *  How the aircraft heading while executing task
 */
@property(nonatomic, assign) DJIGSHeadingMode headingMode;

/**
 *  Create new task
 *
 */
+(id) newTask;

/**
 *  Add waypoint
 *
 *  @param waypoint
 */
-(void) addWaypoint:(DJIGroundStationWaypoint*)waypoint;

/**
 *  Remove one waypoint
 *
 *  @param waypoint Waypoint will be removed
 */
-(void) removeWaypoint:(DJIGroundStationWaypoint*)waypoint;

~~~
 
更多细节，请在DJI Mobile SDK查看 **DJIGroundStationTask.h** 头文件.

#### **3**. 创建 DJIWaypointConfigViewController

在此demo中，我们假设所有加入地图中的航点的参数都是一样的. 

现在，让我们创造一个新的ViewController来让用户设置航点的参数. 去Xcode的project navigator，邮件 **GSDemo** 目录, 选择 **New File...**, 将它的子类设为 **UIViewController**, 命名其为 **DJIWaypointConfigViewController**, 然后确保 "Also create XIB file" 为已选状态. 下一步，打开 **DJIWaypointConfigViewController.xib** 文件并实现UI, 如下所示:

![wayPointConfig](https://raw.githubusercontent.com/dji-sdk/Mobile-SDK-Tutorial/master/iOS-GSDemo/en/images/wayPointConfig.png)

在Waypoint Configuration ViewController里, 我们使用UITextField来时用户设置**DJIGroundStationWaypoint** 实例的**altitude** 属性. 然后，一个UISwitcher来调整**DJIGroundStationTask** 实例的**isLoop** 属性, 它将会让你打开或关闭，当任务重复时. 接下来，有三个UISegmentedControls 来调整**DJIGroundStationWaypoint** object的 **horizontalVelocity** 属性, **DJIGroundStationTask** 实例的 **finishedAction** 属性,以及**DJIGroundStationTask** 实例的 **headingMode** 属性. 

在最下面，我们加入了两个UIButtons为了**Cancel** 和 **Finish** 功能. 更多细节，比如框架位置，框架大小，以及每个UI 元素的背景色，请查看在下载了的project的源代码里的 **DJIWaypointConfigViewController.xib** 文件.

吸纳在，让我们为**DJIWaypointConfigViewController.h**文件里的每一个UI院所创造一个 IBOutlets 和 IBActions, 如下所示:

~~~objc
#import <UIKit/UIKit.h>

@class DJIWaypointConfigViewController;

@protocol DJIWaypointConfigViewControllerDelegate <NSObject>

- (void)cancelBtnActionInDJIWaypointConfigViewController:(DJIWaypointConfigViewController *)waypointConfigVC;
- (void)finishBtnActionInDJIWaypointConfigViewController:(DJIWaypointConfigViewController *)waypointConfigVC;

@end

@interface DJIWaypointConfigViewController : UIViewController

@property (weak, nonatomic) IBOutlet UITextField *altitudeTextField;
@property (weak, nonatomic) IBOutlet UISwitch *repeatSwitcher;
@property (weak, nonatomic) IBOutlet UISegmentedControl *speedSegmentedControl;
@property (weak, nonatomic) IBOutlet UISegmentedControl *actionSegmentedControl;
@property (weak, nonatomic) IBOutlet UISegmentedControl *headingSegmentedControl;

@property (weak, nonatomic) id <DJIWaypointConfigViewControllerDelegate>delegate;

- (IBAction)cancelBtnAction:(id)sender;
- (IBAction)finishBtnAction:(id)sender;

@end
~~~

这里，我们我们同时创造两个 **DJIWaypointConfigViewControllerDelegate** delegate 方法为了当 **Cancel** 和 **Finish** 两个按钮被按下时.

下载，让我们用一下的代码来取代 ** DJIWaypointConfigViewController.m** 文件里的代码:

~~~objc
#import "DJIWaypointConfigViewController.h"

@interface DJIWaypointConfigViewController ()

@end

@implementation DJIWaypointConfigViewController

- (void)viewDidLoad {
    [super viewDidLoad];

    [self initUI];   
}

- (void)didReceiveMemoryWarning {
    [super didReceiveMemoryWarning];
    // Dispose of any resources that can be recreated.
}

- (void)initUI
{
    self.altitudeTextField.text = @"50";
    [self.repeatSwitcher setOn:NO animated:YES]; //Turn off it to cancel execute GroundStation Task looply
    [self.speedSegmentedControl setSelectedSegmentIndex:1]; //Set the horizontal speed to Mid
    [self.actionSegmentedControl setSelectedSegmentIndex:1]; //Set the finishAction to GSTaskFinishedGoHome
    [self.headingSegmentedControl setSelectedSegmentIndex:0]; //Set the headingMode to GSHeadingTowardNexWaypoint
    
}

- (IBAction)cancelBtnAction:(id)sender {
 
    if ([_delegate respondsToSelector:@selector(cancelBtnActionInDJIWaypointConfigViewController:)]) {
        [_delegate cancelBtnActionInDJIWaypointConfigViewController:self];
    }
}

- (IBAction)finishBtnAction:(id)sender {
    
    if ([_delegate respondsToSelector:@selector(finishBtnActionInDJIWaypointConfigViewController:)]) {
        [_delegate finishBtnActionInDJIWaypointConfigViewController:self];
    }
}

@end
~~~

在以上代码中, 我们创造一个用默认数据来初始化UI控制的**initUI** 方法, 他在**viewDidload**方法里运行. 比如说，我们将 **altitudeTextField** 的默认文字调为 **50**, 使用户不需要再第一次打开程序时输入高度数值. 不需要在开始前改变设置。他们可以马上按下 **Finish** 键.

## 3. 设置地面站任务

#### **1**.加入一个 **DJIWaypointConfigViewController** to和 **DJIRootViewController**
现在，让我们来到 **DJIRootViewController.m** 文件, 在上方加入 **DJIWaypointConfigViewController.h** 头文件, 以及创造一个**DJIWaypointConfigViewController**类型的属性名为**waypointConfigVC**. 然后，实现 **DJIWaypointConfigViewControllerDelegate** 协议, 如下所示:

~~~objc
#import "DJIRootViewController.h"
#import "DJIGSButtonViewController.h"
#import "DJIWaypointConfigViewController.h"

#define kEnterNaviModeFailedAlertTag 1001

@interface DJIRootViewController ()<DJIGSButtonViewControllerDelegate, DJIWaypointConfigViewControllerDelegate>
@property (nonatomic, assign)BOOL isEditingPoints;
@property (nonatomic, strong)DJIGSButtonViewController *gsButtonVC;
@property (nonatomic, strong)DJIWaypointConfigViewController *waypointConfigVC;
@end
~~~

接下来，让我们加入一些代码来初始化**waypointConfigVC**实例变量并将其delegate在**initUI**方法下方设置为 **DJIRootViewController** :

~~~objc
-(void) initUI
{
    self.modeLabel.text = @"N/A";
    self.gpsLabel.text = @"0";
    self.vsLabel.text = @"0.0 M/S";
    self.hsLabel.text = @"0.0 M/S";
    self.altitudeLabel.text = @"0 M";
    
    self.gsButtonVC = [[DJIGSButtonViewController alloc] initWithNibName:@"DJIGSButtonViewController" bundle:[NSBundle mainBundle]];
    [self.gsButtonVC.view setFrame:CGRectMake(0, self.topBarView.frame.origin.y + self.topBarView.frame.size.height, self.gsButtonVC.view.frame.size.width, self.gsButtonVC.view.frame.size.height)];
    self.gsButtonVC.delegate = self;
    [self.view addSubview:self.gsButtonVC.view];
    
    self.waypointConfigVC = [[DJIWaypointConfigViewController alloc] initWithNibName:@"DJIWaypointConfigViewController" bundle:[NSBundle mainBundle]];
    self.waypointConfigVC.view.alpha = 0;
    self.waypointConfigVC.view.center = self.view.center;
    self.waypointConfigVC.delegate = self;
    [self.view addSubview:self.waypointConfigVC.view];
    
}
~~~

在以上代码中，我们将**waypointConfigVC**的画面的 **alpha** 属性为零来在开始时隐藏画面. 接下来，将它的位置设置在 **DJIRootViewController**的画面的中心.

更进一步，实现 **DJIWaypointConfigViewControllerDelegate** 方法, 如下所示:

~~~objc
#pragma mark - DJIWaypointConfigViewControllerDelegate Methods

- (void)cancelBtnActionInDJIWaypointConfigViewController:(DJIWaypointConfigViewController *)waypointConfigVC
{
    __weak DJIRootViewController *weakSelf = self;
    
    [UIView animateWithDuration:0.25 animations:^{
        weakSelf.waypointConfigVC.view.alpha = 0;
    }];
}

- (void)finishBtnActionInDJIWaypointConfigViewController:(DJIWaypointConfigViewController *)waypointConfigVC
{
    __weak DJIRootViewController *weakSelf = self;
    
    [UIView animateWithDuration:0.25 animations:^{
        weakSelf.waypointConfigVC.view.alpha = 0;
    }];

}
~~~

在第一个delegate方法中, 我们用一个UIView中的类方法来动画化**waypointConfigVC**的画面的变化的**alpha**数值:

~~~objc
+ (void)animateWithDuration:(NSTimeInterval)duration animations:(void (^)(void))animations NS_AVAILABLE_IOS(4_0);
~~~

在第二个 delegate 方法中, 我们做同样的事情.

最后，将

~~~objc
- (void)startBtnActionInGSButtonVC:(DJIGSButtonViewController *)GSBtnVC;
~~~
方法里的代码取代为下面的代码来在用户按下**start** 按钮是显示**waypointConfigVC**的画面:

~~~objc
- (void)startBtnActionInGSButtonVC:(DJIGSButtonViewController *)GSBtnVC
{
    __weak DJIRootViewController *weakSelf = self;

    [UIView animateWithDuration:0.25 animations:^{
        weakSelf.waypointConfigVC.view.alpha = 1.0;
    }];
        
}
~~~

当这些都完成了，让我们来创建并运行此project. 尝试按下**Edit** 按钮 and **Start** 按钮来显示**waypointConfigVC**的画面:

![waypointConfigView](https://raw.githubusercontent.com/dji-sdk/Mobile-SDK-Tutorial/master/iOS-GSDemo/en/images/waypointConfigView.png)

#### **2**. 操作地面站任务

现在，让我们回到 **DJIRootViewController.h** 文件. 首先，在interface里实现 **GroundStationDelegate** 和 **DJINavigationDelegate** 协议. 然后，创造一个**DJIGroundStationTask**类型的属性并将其命名为 **gsTask**. 同时，创造一个**UIAlertView**类型的属性并将其命名为**uploadProgressView**. UIAlertView将会显示地面站任务的状态. 完整的**DJIRootViewController**头文件将看起来像下面的代码:

~~~objc
#import <UIKit/UIKit.h>
#import <DJISDK/DJISDK.h>
#import "DJIMapController.h"
#import <MapKit/MapKit.h>
#import <CoreLocation/CoreLocation.h>

@interface DJIRootViewController : UIViewController<MKMapViewDelegate, CLLocationManagerDelegate, DJIDroneDelegate, DJIMainControllerDelegate, GroundStationDelegate, DJINavigationDelegate>

@property (nonatomic, strong) DJIMapController *mapController;
@property (weak, nonatomic) IBOutlet MKMapView *mapView;
@property(nonatomic, strong) CLLocationManager* locationManager;
@property(nonatomic, assign) CLLocationCoordinate2D userLocation;
@property(nonatomic, assign) CLLocationCoordinate2D droneLocation;
@property (nonatomic, strong) UITapGestureRecognizer *tapGesture;

@property (weak, nonatomic) IBOutlet UIView *topBarView;

@property(nonatomic, strong) IBOutlet UILabel* modeLabel;
@property(nonatomic, strong) IBOutlet UILabel* gpsLabel;
@property(nonatomic, strong) IBOutlet UILabel* hsLabel;
@property(nonatomic, strong) IBOutlet UILabel* vsLabel;
@property(nonatomic, strong) IBOutlet UILabel* altitudeLabel;

@property(nonatomic, strong) DJIDrone* inspireDrone;
@property(nonatomic, strong) DJIInspireMainController* inspireMainController;

@property(nonatomic, strong) DJIGroundStationTask* gsTask;
@property(nonatomic, strong) UIAlertView* uploadProgressView;

@end
~~~

家下来，来到 **DJIRootViewController.m** 文件并设置 **inspireMainController** 实例变量的 **groundStationDelegate** 和 **navigationDelegate** 为 **DJIRootViewController**， 如下所示:

~~~objc
- (void)initDrone
{
    self.inspireDrone = [[DJIDrone alloc] initWithType:DJIDrone_Inspire];
    self.inspireMainController = (DJIInspireMainController*)self.inspireDrone.mainController;
    self.inspireDrone.delegate = self;
    self.inspireMainController.mcDelegate = self;
    self.inspireMainController.groundStationDelegate = self;
    self.inspireMainController.navigationDelegate = self;
}
~~~

更进一步，在**startBtnActionInGSButtonVC** delegate方法下方加入以下代码:

~~~objc
- (void)startBtnActionInGSButtonVC:(DJIGSButtonViewController *)GSBtnVC
{
    __weak DJIRootViewController *weakSelf = self;

    [UIView animateWithDuration:0.25 animations:^{
        weakSelf.waypointConfigVC.view.alpha = 1.0;
    }];
    
    NSArray* wayPoints = self.mapController.wayPoints;
    if (wayPoints == nil || wayPoints.count == 0) {
        UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"No waypoint for mission" message:@"" delegate:self cancelButtonTitle:@"OK" otherButtonTitles:nil];
        [alert show];
        return;
    }

    self.gsTask = [DJIGroundStationTask newTask];
    for (int i = 0; i < wayPoints.count; i++) {
        CLLocation* location = [wayPoints objectAtIndex:i];
        if (CLLocationCoordinate2DIsValid(location.coordinate)) {
            DJIGroundStationWaypoint* waypoint = [[DJIGroundStationWaypoint alloc] initWithCoordinate:location.coordinate];
            [self.gsTask addWaypoint:waypoint];
        }
    }
    
}
~~~

以上新加入的代码中, 我们创造一个局部 **NSArray** 变量命名为 **wayPoints** 并将其数值定为 **mapController**的 **wayPoints** 数组. 接下来，查看数组是否存在或者数组是否为空白. 如果它是空白的或者不存在, 显示UIAlertView使用户知道此任务里没有航点. 

#####重要事项: 为了安全，在任务开始前加入逻辑来查看GPS卫星的数量非常重要，此事项已在此教程的第一部分中提到过. 如果卫星数量小于6，你应该防止用户开始地面站任务并且显示警告. 因为这里我们在用 DJI PC 模拟器, 我们在一个完美的环境下测试应用，GPS卫星数量将一直是10.

下一步，我们用**DJIGroundStationTask**的**newTask**类方法来初始化**gsTask** 实例变量. 接下来，我们用一个for循环来获取每一个**wayPoints**数组的航点的**CLLocation** 并且用一下方法检查其 **coordinate** 是否有效:

~~~objc
BOOL CLLocationCoordinate2DIsValid(CLLocationCoordinate2D coord);
~~~

最后，如果坐标有效，我们建立一个**DJIGroundStationWaypoint**类型的航点并将其加入到**gsTask**用以下的方法:

~~~objc
-(void) addWaypoint:(DJIGroundStationWaypoint*)waypoint;
~~~

当你完成以上任务是, 让我们来到 DJIWaypointConfigViewController delegate 方法的 **finishBtnActionInDJIWaypointConfigViewController** and 并将**gsTask**有关的代码加入到此方法的下方:

~~~objc
- (void)finishBtnActionInDJIWaypointConfigViewController:(DJIWaypointConfigViewController *)waypointConfigVC
{
    __weak DJIRootViewController *weakSelf = self;
    
    [UIView animateWithDuration:0.25 animations:^{
        weakSelf.waypointConfigVC.view.alpha = 0;
    }];
    
    for (int i = 0; i < self.gsTask.waypointCount; i++) {
        DJIGroundStationWaypoint* waypoint = [self.gsTask waypointAtIndex:i];
        waypoint.altitude = [self.waypointConfigVC.altitudeTextField.text floatValue];
        waypoint.horizontalVelocity = self.waypointConfigVC.speedSegmentedControl.selectedSegmentIndex * 3 + 3;
    }
    
    self.gsTask.isLoop = self.waypointConfigVC.repeatSwitcher.isOn;
    self.gsTask.headingMode = (DJIGSHeadingMode)self.waypointConfigVC.headingSegmentedControl.selectedSegmentIndex;
    self.gsTask.finishedAction = (DJIGSTaskFinishedAction)self.waypointConfigVC.actionSegmentedControl.selectedSegmentIndex;
    
    [self.inspireMainController uploadGroundStationTask:self.gsTask];
}
~~~

以上代码中，我们用for循环来设置**gsTask**航点数组的**DJIGroundStationWaypoint**的  **altitude** 以及 **horizontalVelocity** 属性根据**DJIWaypointConfigViewController**中的设置. 当你完成这些事，我们更新**gsTask**的 **isLoop**, **headingMode** 和 **finishedAction** 属性 . 最后，我们运行**DJIInspireMainController**的 **uploadGroundStationTask** 方法(因为我们在此教程使用Inspirel)来上传地面站任务.

更进一步，我们创造一个新方法命名为 **hideProgressView**，它将会隐藏 **uploadProgressView** 并且实现 **GroundStationDelegate** 方法来更新任务状态, 如下所示:

~~~objc
-(void) hideProgressView
{
    if (self.uploadProgressView) {
        [self.uploadProgressView dismissWithClickedButtonIndex:-1 animated:YES];
    }
}

#pragma mark - GroundStationDelegate
-(void) groundStation:(id<DJIGroundStation>)gs didExecuteWithResult:(GroundStationExecuteResult*)result
{
    if (result.currentAction == GSActionStart) {
        if (result.executeStatus == GSExecStatusFailed) {
            [self hideProgressView];
            NSLog(@"Mission Start Failed...");
        }
    }
    if (result.currentAction == GSActionUploadTask) {
        if (result.executeStatus == GSExecStatusFailed) {
            [self hideProgressView];
            NSLog(@"Upload Mission Failed");
        }
    }
}

-(void) groundStation:(id<DJIGroundStation>)gs didUploadWaypointMissionWithProgress:(uint8_t)progress
{
    if (self.uploadProgressView == nil) {
        self.uploadProgressView = [[UIAlertView alloc] initWithTitle:@"Mission Uploading" message:@"" delegate:nil cancelButtonTitle:nil otherButtonTitles:nil];
        [self.uploadProgressView show];
    }
    
    NSString* message = [NSString stringWithFormat:@"%d%%", progress];
    [self.uploadProgressView setMessage:message];
}
~~~

在上面的代码中，第一个delegate方法运行时获取地面站结果. 我们运行 **hideProgressView** 方法来隐藏 **uploadProgressView** 当我们在**GroundStationExecuteResult*里检查 **currentAction** 和 **executeStatus** 以后.

第二个delegate方法运行时检查上传航点任务的进程. 这里，我们初始化 **uploadProgressView** 实例变量并将它的**message** property设置为delegate方法的 **progress** 变量 . 更多细节，请查看 **DJIGroundStation.h** 文件.

最后，让我们实现 **DJINavigationDelegate** 方法，如下所示:

~~~objc
-(void) onNavigationMissionStatusChanged:(DJINavigationMissionStatus*)missionStatus
{
}

-(void) onNavigationPostMissionEvents:(DJINavigationEvent*)event
{
        if (event.eventType == NavigationEventMissionUploadFinished) {
        DJINavigationMissionUploadFinishedEvent* finishedEvent = (DJINavigationMissionUploadFinishedEvent*)event;
        [self.uploadProgressView setTitle:@"Mission Upload Finished"];
        if (!finishedEvent.isMissionValid) {
            [self.uploadProgressView setMessage:@"Mission Invalid!"];
        }
        else
        {
            [self.uploadProgressView setMessage:[NSString stringWithFormat:@"Estimate Time:%d s", (int)finishedEvent.eatimateTime]];
            [self.inspireMainController startGroundStationTask];
        }
        
        [self performSelector:@selector(hideProgressView) withObject:nil afterDelay:3.0];
    }
    else if (event.eventType == NavigationEventMissionExecuteFinished)
    {
        UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"Mission Finished" message:@"" delegate:self cancelButtonTitle:@"OK" otherButtonTitles:nil];
        [alert show];

    }
}
~~~

第一个delegate被用于检查任务状态. 你可以使用**missionStatus** 变量的 **missionType** 属性来检查任务类型, 此属性在**DJINavigation.h**里被定义, 如**DJINavigationMissionType**类型enum所示:

~~~objc
typedef NS_ENUM(uint8_t, DJINavigationMissionType)
{
    /**
     *  No mission
     */
    NavigationMissionNone,
    /**
     *  Waypoint mission
     */
    NavigationMissionWaypoint,
    /**
     *  Hotpoint mission
     */
    NavigationMissionHotpoint,
    /**
     *  Followme mission, Not support now
     */
    NavigationMissionFollowme,
    /**
     *  Unknown mission
     */
    NavigationMissionUnknown
};
~~~

第二个delegate方法被用来检查现在的任务. 你可以查看 **event** 变量的 **eventType** 属性来得知现在的任务目前在哪一部分. **DJINavigationEventType** 在**DJINavigation.h** 头文件里被定义了，如下所示:

~~~objc
/**
 *  Navigation event type
 */
typedef NS_ENUM(uint8_t, DJINavigationEventType){
    /**
     *  Mission upload finished event
     */
    NavigationEventMissionUploadFinished,
    /**
     *  Mission execute finished event
     */
    NavigationEventMissionExecuteFinished,
    /**
     *  Aircraft reach target waypoint event
     */
    NavigationEventWaypointReached,
};
~~~
在第二个delegate方法中, 当eventType等于 **NavigationEventMissionUploadFinished**, 因为since **DJINavigationMissionUploadFinishedEvent** 是 **DJINavigationEvent**的子类, 我们用方法里的**event** 变量创造一个 **DJINavigationMissionUploadFinishedEvent** 变量. 然后更新 **uploadProgressView** 的标题为 "Mission Upload Finished". 接下来，如果 **finishedEvent**的 **isMissionValid** bool 数值为 false, 我们设置 **uploadProgressView**的**message** 为 "Mission Invalid!". 不然的话, 我们设置**uploadProgressView**的 **message** 为一个排版过的 **NSString** 实例使用**finishedEvent**的 **eatimateTime** 属性. 

接下来，运行**inspireMainController**的 **startGroundStationTask** 方法来开始一个航点地面站任务! 然后，我们运行 **performSelector** 方法并且通过 **hideProgressView** 在3秒延迟来隐藏过程画面.

当eventType等于 **NavigationEventMissionExecuteFinished**, 我们显示 **UIAlertView** 来告诉用户航点地面站任务完成了!

最后，让我们实现 **stopBtnActionInGSButtonVC** 方法, 他是一个 **DJIGSButtonViewController** delegate 方法来停止地面站任务, 如下所示:

~~~objc
- (void)stopBtnActionInGSButtonVC:(DJIGSButtonViewController *)GSBtnVC
{
    [self.inspireMainController stopGroundStationTask];
}
~~~

## 4. 演出时间

你已经在此教程中做了很多, 现在是时候测试你的成果了.

#####重要事项: 确保你飞机的电池量高于10%, 不然地面站任务将会失败!

建造并运行此 project 来安装应用到你的移动设备上. 之后，请用usb线将你的飞机连接上使用Windows的PC或者虚拟机器. 然后，启动飞机再启动遥控器. 接下来，在DJI PC 模拟器按下 **Display Simulator** 按钮并且你可以输入你当前位置的经纬度信息到模拟器如果你喜欢.

![simulatorPreview](https://raw.githubusercontent.com/dji-sdk/Mobile-SDK-Tutorial/master/iOS-GSDemo/en/images/simulator_preview.png)

然后将你的遥控器连接上你的移动设备并且运行应用. 你将会看到下面的截图:

![enterNaviModeFailed](https://raw.githubusercontent.com/dji-sdk/Mobile-SDK-Tutorial/master/iOS-GSDemo/en/images/enterNaviModeFailed.jpg)

如果你晕倒这个问题，请在此教程的第一部分查看解决方案. 接下来，让我们回到DJI PC 模拟器在你的PC上并且按下 **Start Simulation** 按钮. 一个小型红色飞机将会在地图上显示如下图所示:

![aircraftOnMap1](https://raw.githubusercontent.com/dji-sdk/Mobile-SDK-Tutorial/master/iOS-GSDemo/en/images/aircraftOnMap1.jpg)

按下 "**Edit**" 按钮, 并且点图将会放大到你所在的地区以你的飞机为中心:

![aircraftOnMap2](https://raw.githubusercontent.com/dji-sdk/Mobile-SDK-Tutorial/master/iOS-GSDemo/en/images/aircraftOnMap2.jpg)

接下来，轻拍地图上任何地点来测试航点功能. 你拍的地方将会产生一个航点以及一个紫色的图钉, 如下图:

![waypoints](https://raw.githubusercontent.com/dji-sdk/Mobile-SDK-Tutorial/master/iOS-GSDemo/en/images/waypoints.jpg)

当年下 **Start** 按钮， **Waypoint Configuration** 画面将会出现. 当你做完适当的改变时，按下 **Finish** 按钮.

![waypointConfigStart](https://raw.githubusercontent.com/dji-sdk/Mobile-SDK-Tutorial/master/iOS-GSDemo/en/images/waypointConfigStart.jpg)

航点任务将开始上传并且当其借宿是，任务将会被处理. 你将会看到飞机朝你之前设置的航点开始移动, 如下所示:

![missionUploadFinished](https://raw.githubusercontent.com/dji-sdk/Mobile-SDK-Tutorial/master/iOS-GSDemo/en/images/missionUploadFinished.jpg)

![missionStart1](https://raw.githubusercontent.com/dji-sdk/Mobile-SDK-Tutorial/master/iOS-GSDemo/en/images/missionStart1.jpg)

![missionStart2](https://raw.githubusercontent.com/dji-sdk/Mobile-SDK-Tutorial/master/iOS-GSDemo/en/images/missionStart2.jpg)

同时，你将能够在DJI PC 模拟器上目睹Inspire 1 起飞并且开始自动飞行.

![simulatorMissionStart](https://raw.githubusercontent.com/dji-sdk/Mobile-SDK-Tutorial/master/iOS-GSDemo/en/images/simulatorMissionStart.jpg)

当航点任务结束时，一个名为**Mission Finished**的警告栏将会出现并且Inspire 1将开始返航！

![missionFinished](https://raw.githubusercontent.com/dji-sdk/Mobile-SDK-Tutorial/master/iOS-GSDemo/en/images/missionFinished.jpg) 

遥控器将开始发出哔声，然后遥控器上的 **Go Home** 键将开始闪烁白灯. 让我们看看现在的DJI PC 模拟器:

![landing](https://raw.githubusercontent.com/dji-sdk/Mobile-SDK-Tutorial/master/iOS-GSDemo/en/images/landing.jpg)
 
Inspire 1最终会回到起点，着陆，并且遥控器的哔声将会停止. 应用将会回到普通状态. 如果你按下 **Clear** 按钮，之前设置的全部航点都将会消失并且另外一个航点任务将会开始. 在任务过程中。如果你想要停止地面站任务，你可以按下 **Stop** 按钮.

## 5. 下一步是什么?
   
   在此教程中，你学会了如何修改 **DJIGroundStationWaypoint** 和 **DJIGroundStationTask**. 更进一步, 你学会了如何使用 **DJIGroundStationTask** 来加入航点因为你可以用**DJIInspireMainController**(因为我们在使用 Inspire 1)中的方法来 **upload** 至, **start** 和 **stop** 地面站的任务. 同时，你学习了如何使用 **DJINavigationDelegate** 和 **GroundStationDelegate** 方法来获取地面站任务的信息.
      
   恭喜你! 现在你完成了demo，你可以在现有的基础上创建你自己的地面站应用. 你可以改善加入航点的方法(比如说在地图上画线以自动生成航点), 尝试调整航点的属性 (比如说方向，水平速度等), 以及加入更多功能. 想要做出一个好玩的地面站应用，你还有很长的路要走. 祝你好运并且希望你喜欢我们的教程!