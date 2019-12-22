# Screenz SDK
## [~>3.0]

Screenz SDK allows you to implement your own Screenz client that will live in the Screenz environment.

This environment needs a server side component, that will be provided by the infraestructure department in order to create the app in the server, which will be referenced by an id in the framework.

In order to make your own Screenz client we will guide you through the following steps:
  - Adding the Screenz Framework
  - Configuring the Framework
  - Displaying the Framework
  - Push Notifications

#### Adding the Screenz Framework

First of all, create an Android project if you don't have one. After that, add the following on your project (app generally called) build.gradle file:

```Java
repositories {
    maven {
        url "https://raw.github.com/TheBoxLtd/shell-lib-maven-and/master/"
    }
    maven { url 'https://dl.bintray.com/drummer-aidan/maven'}
    flatDir {
        dirs 'libs'
    }
}
```

Also, you need to include the following dependency:

```Java
implementation ('com.screenz:shell_library:[VERSION]@aar'){
    transitive=true;
}
```

**Important : Replace [VERSION] with the framework's version you want to use**


**Important: the framework requires SDK version 28 and minSdkVersion 19**

Is mandatory to include the following libraries in the application gradle (use the higher version available at the moment):

```
implementation 'com.google.android.gms:play-services-location:15.0.1'
implementation 'com.google.firebase:firebase-core:16.0.1'
implementation 'com.google.firebase:firebase-messaging:17.3.0'
```

If you want to use HockeyApp or AppsFlyer you need to include those dependencies in your gradle. The framework will use any of them if they are available.

```
implementation 'net.hockeyapp.android:HockeySDK:5.1.0'

implementation 'com.appsflyer:af-android-sdk:4.8.13@aar'
```

#### Configuring the Framework

In order to use the framework you must setup certain parameter configuration first, and there are several configuration class for this.
All these configuration are entered through the ConfigManager class:

```Java

    ConfigManager configManager = ConfigManager.getInstance();

```

The above code shows how to instantiate the configuration manager. Each social network
requires a configuration of its own, and that's why we see a setter for each one. 


```Java

    ConfigManager configManager = ConfigManager.getInstance();
    configManager.setFacebookData(facebook);
    configManager.setGooglePlusData(google);
    configManager.setInstagramData(instagram);
    configManager.setTwitterData(twitter);
    
```
All these configurations are optional, so if your application doesn't use Twitter, you don't have to setup any dummy data to it.


**Important: each configuration class will be validated in order to avoid future problems due to missing credentials and if the
validation results in a failure you will get an exception about it**


Besides the social networks configuration there is another configuration class called CoreData, which contains the minimum information
the framework must have to run, if it is not added an exception will be thrown.
 
```Java

public class CoreData implements SetUpData {

    public String backgroundColor;//Used as the background color of the webview while loading

    public boolean productionMode;//Flag to indicate in which mode the framework must run

    public boolean productionEnvironment = true;//Flag to indicate against which environment the framework must run. True by default

    public List<Integer> pids;//Your Application Identifier

    public String gcmSenderId;//SENDER_ID used for push notifications

    public String noInternetMessage;//Message to display when no internet access

    public String hockeyAppKey;//Key for Hockey App

    public boolean waitForPageLoadEvent = true;// Flag to indicate if the framework will wait for a page load event while playing the dynamic splash. True by default.
    
    public String video_upload_key;//API Key for the upload video service
}


.
.
.

    CoreData coreData = new CoreData();
    //setup your core data
    ConfigManager configManager = ConfigManager.getInstance();
    configManager.setCoreData(coreData);

```
#### Adding permissions to Android Manifest

Need to add the following permissions to the app manifest:

```xml

  <uses-permission android:name="android.permission.INTERNET" />
  <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
  <uses-permission android:name="android.permission.READ_PHONE_STATE" />
  <uses-permission android:name="android.permission.CAMERA" />
  <uses-permission android:name="android.permission.RECORD_AUDIO" />
  <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
  <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
  <uses-permission android:name="android.permission.CAPTURE_VIDEO_OUTPUT" />

```

#### Sending Data to the webview

You can send any data that the webview needs to consume using the following method:
 
```
configManager.setExtraData(this,"data to store");
```

You can also set the page to be opened and pid with these methods:

```
configManager.setLaunchPageID(this,"[PAGEID]");
configManager.setPid(this,[PID]);
```

In this examle, the webview will have access to "data to store" when is run.
This data needs to be set before launching the framework

#### Receiving Data from the webview

In order to receive data from the webview you need to setup a broadcast receiver for the PUBLISH_DATA event.

This can be done, for example, like this:

```Java
IntentFilter intentFilter = new IntentFilter(Constants.Event.PUBLISH_DATA);
registerReceiver(dataReceiver , intentFilter);

BroadcastReceiver dataReceiver = new BroadcastReceiver() {
   @Override
   public void onReceive(Context context, Intent intent) {
       String data = intent.getStringExtra(Constants.Event.PUBLISH_DATA_PARAMETER);
       Log.d("DATA RECEIVER",data);
   }
};
```

Take notice that you need to register the broadcast to the Constants.Event.PUBLISH_DATA event, and read the data from the parameter Constants.Event.PUBLISH_DATA_PARAMETER.

#### Exit SDK Event

Using the described above receiver, the webview will request to exit with the data 'sdk-exit-new'.
You should subscribe to this event, close the sdk when received and handle the views lifecycle accordingly 

#### Displaying the Framework

Now that you have the framework already setup, you are able to display it. To do so, you need to invoke
the ShellLibraryBuilder class to get a fragment (android.support.v4.app.Fragment) which will be in charge of the entire UI work of the framework:

```Java

Fragment fragment = ShellLibraryBuilder.create(context);

```

Just add this fragment to your activity and that's pretty much it.

```Java


    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.my_layout);
        getSupportFragmentManager().beginTransaction().replace(R.id.fragment_container, ShellLibraryBuilder.create(this)).commit();
    }

```

There's only one more step to finish the displaying, which is the callbacks thrown by some libraries inside the framework will be pinging
the activity and not the framework, therefore the following code must be added inside your activity

```Java

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        Fragment currentFragment = getSupportFragmentManager().findFragmentById(R.id.fragment_container);
        if (currentFragment != null){
            currentFragment.onActivityResult(requestCode, resultCode, data);
        }
    }


```

** Due to the fact that the fragment will follow the activity container orientation, we advice to only use it to portrait **

#### Push Notifications

Notifications are handled by FCM (Firebase Cloud Messaging).

In order to configure notifications you must provide the configuration file from Firebase Console. The framework should take care of the rest.

Check https://firebase.google.com/docs/android/setup for setup instructions

#### Proguard

If your application has proguard enabled, you need to add the following:


```Java
-keepattributes Exceptions, Signature, InnerClasses
# Application config classes and ShellLibraryBuilder
-keep class com.screenz.shell_library.config.** { *; }
-keep class com.screenz.shell_library.model.** { *; }
-keep class com.screenz.shell_library.ShellLibraryBuilder { *; }

#Retrofit
-keep class com.squareup.okhttp.** { *; }
-keep interface com.squareup.okhttp.** { *; }
-dontwarn com.squareup.okhttp.**

-dontwarn rx.**
-dontwarn retrofit.**
-dontwarn okio.**
-keep class retrofit.** { *; }
-keepclasseswithmembers class * {
    @retrofit.http.* <methods>;
}

#WebView Native calls
-keepclassmembers class * {
    @android.webkit.JavascriptInterface <methods>;
}
#Picasso
-dontwarn com.squareup.okhttp.**

-keepattributes Exceptions, Signature, InnerClasses, LineNumberTable
```
