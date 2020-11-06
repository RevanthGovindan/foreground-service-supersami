
# @supersami/react-native-foreground-service


### What this service do?
It is a library for react-native android only, specifically designed to make things easier for people who are looking for a service that always kept on running regardless of an app running in front or background. This service will run the application is foreground with a notification (To run a foreground service you have to have a notification ) that will run a headless task, which runs your javascript function after an interval of time as well as just one time. The possibilities are countless of how you can use this service, for the one you can use this to make a tracker app.


This service is not fully created by me it is the result of a mixture of many different sources and articles.

I modified this for my personal use.


[Medium Article](https://medium.com/reactbrasil/how-to-create-an-unstoppable-service-in-react-native-using-headless-js-93656b6fd5d1)
[Voximplant](https://github.com/voximplant/react-native-foreground-service)
[react-native-push-notification](https://github.com/zo0r/react-native-push-notification/)
[react-native-foreground-service (modified this repository))](https://github.com/cristianoccazinsp/react-native-foreground-service.git)
  
  

# Installation

## Step 1  
```
yarn add @supersami/react-native-foreground-service
````

## Step 2

Add Permissions for foreground service as well as for wake lock by adding these 2 lines in your android manifest.
```
<uses-permission android:name="android.permission.FOREGROUND_SERVICE"/> 
<uses-permission android:name="android.permission.WAKE_LOCK" />
```

## Step 3

Register Service in your manifest inside application.

```
<meta-data android:name="com.supersami.foregroundservice.notification_channel_name" android:value="supersami Service"/> 
<meta-data android:name="com.supersami.foregroundservice.notification_channel_description" android:value="supersami Service."/> 
<meta-data android:name="com.supersami.foregroundservice.notification_color" android:resource="@color/orange"/>
<service android:name="com.supersami.foregroundservice.ForegroundService"></service>
<service android:name="com.supersami.foregroundservice.ForegroundServiceTask"></service>

```


Go to your Android -> App -> src -> main - res
looks for the values-color.xml, If not exist Create one.
and then add this, otherwise, just add the orange color and integer array

```
<?xml version="1.0" encoding="utf-8"?>
<resources>
<item  name="orange"  type="color">#FF4500
</item>
<integer-array  name="androidcolors">
<item>@color/orange</item>
</integer-array>
</resources>
```

## Step 4

go to your main Activity and declare a variable
```
	public  boolean  isOnNewIntent = false;
```
 and you need to override two methods of MainActivity. 


> Make sure you override onNewIntent First and then OnStart.

- onNewIntent
```
  

	@Override
	public  void  onNewIntent(Intent  intent) {
		super.onNewIntent(intent);
		isOnNewIntent = true;
    ForegroundEmitter(intent);
	}

```
- onStart
``` 

	@Override
	protected  void  onStart() {
		super.onStart();
		if(isOnNewIntent == true){}else {
      ForegroundEmitter(getIntent());
		}
	}
```

and in last our main Function - ForegroundEmitter

```
  
 public void ForegroundEmitter(Intent intent){
    // this method is to send back data from java to javascript so one can easily 
    // know which button from notification or from the notification btn is clicked

    String main = intent.getStringExtra("mainOnPress");
    String btn = intent.getStringExtra("buttonOnPress");
	WritableMap  map = Arguments.createMap();
	if (main != null) {
		// Log.d("SuperLog A", main);
		map.putString("main", main);
	} 
	if (btn != null) {
		// Log.d("SuperLog B", btn);
		map.putString("button", btn);
	}
	try {
		getReactInstanceManager().getCurrentReactContext()
		.getJSModule(DeviceEventManagerModule.RCTDeviceEventEmitter.class)
		.emit("notificationClickHandle", map);  	
	} catch (Exception  e) {	
	Log.e("SuperLog", "Caught Exception: " + e.getMessage());
	}
}
```

and then in your Js file in your root which is index file you need to add this like 
```
import ForegroundService from '@supersami/react-native-foreground-service';
    let foregroundTask = async (data) => {
    console.log("I am here",data)
    await myTask();
}

const myTask = () => {
    let i = 0
    
    console.log("Hi")
}
ForegroundService.registerForegroundTask("myTaskName", foregroundTask);
```




# Usage:

It's Easy to use the Foreground Service now.

To Start The Service 
```
let obj = { routeName : "mainActivity", routeParams : {data:""} }
let obj1  = { routeName : "mainActivity 1", routeParams : {data:""} }

let notificationConfig = {
  id: 434,
  title: 'SuperService',
  message: `I hope you are doing your `,
  vibration: false,
  visibility: 'public',
  icon: 'ic_launcher',
  importance: 'max',
  number: String(1),
  button: false,
  buttonText: 'Checking why are you repeating your self',
  buttonOnPress :    JSON.stringify(obj),
  mainOnPress :    JSON.stringify(obj1)
};
// Above is the notification config 

const start = async check => {
  await ForegroundService.startService(notificationConfig);

  await ForegroundService.runTask({
    taskName: 'myTaskName',
    delay: 0,
    loopDelay : 5000,
    onLoop: true,
  });
};

//start with
start();

// stop with 
await  ForegroundService.stopService()


// Receiving Clicks in the App so you can do what ever like redirecting to any 
// route with any specific data.

 useEffect(() => {
    let subscip = DeviceEventEmitter.addListener(
      'notificationClickHandle',
      function (e: Event) {
        console.log('json', e);
      },
    );
    return function cleanup() {
      subscip.remove();
    };
  }, []);


```

there are multiple functions in the package which are as follow

 - startService
- updateNotification
- cancelNotification
- stopService
- stopServiceAll
- isRunning



### startService
 Start foreground service
 Multiple calls won't start multiple instances of the service but will increase its internal counter
 so calls to stop won't stop until it reaches 0.
 Note: notification config can't be re-used (becomes immutable)
    
### updateNotification
 Updates a notification of a running service. Make sure to use the same ID  or it will rigger a separate notification.
 **Note**: this method might fail if called right after starting the service since the service might not be yet ready.
 If service is not running, it will be started automatically like calling startService.
     
### cancelNotification
 Cancels/dismisses a notification given its id. Useful if the service used
 more than one notification
    
### stopService
 Stop foreground service. Note: Pending tasks might still complete.
 If startService will be called multiple times, this needs to be called many times.
    
### stopServiceAll
 Stop foreground service. Note: Pending tasks might still complete.
 This will stop the service regardless of how many times start was called
   
### isRunning
 Returns an integer indicating if the service is running or not.
 The integer represents the internal counter of how many startService
 calls were done without calling stopService
    
### Notification Config
(*) represent required 


|Parameter| what they do |
|--|--|
|  *id | An Id of a unique notification, this id can be used to latter cancel the notification from function describe above |
|  *title | title of the notification |
|  *message | Description of the notification |
|  vibration | Boolean |
|  visibility | private - public - secret |
|  largeicon | Large icon name - ic_launcher |
|  icon | Small icon name - ic_notification |
|  *importance | Importance (and priority for older devices) of this notification. This might affect notification sound One of: *                                  none - IMPORTANCE_NONE (by default),    *                               min - IMPORTANCE_MIN,    *                               low - IMPORTANCE_LOW,    *                               default - IMPORTANCE_DEFAULT    *                               high - IMPORTANCE_HIGH,    *                               max - IMPORTANCE_MAX |
|  *number | int specified as string > 0, for devices that support it, this might be used to set the badge counter |
|  *button | Button visibility, Either True or False |
|  buttonText | Button Text |
|  buttonOnPress | Stringify Json that latter can be converted into JSON after triggering from a button and can be processed, look at the above example |
|  *mainOnPress | Stringify Json that latter can be converted into JSON after triggering from a button and can be processed, look at the above example |

