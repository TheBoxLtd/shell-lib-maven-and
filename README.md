# Screenz SDK

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
    maven { url 'https://maven.fabric.io/public' }
    flatDir {
        dirs 'libs'
    }
}
```

Also, you need to include the following dependency:

```Java
compile ('com.screenz:shell_library:[VERSION]@aar'){
    transitive=true;
}
```

**Important : Replace [VERSION] with the framework's version you want to use**


**Important: the framework requires SDK version 24 and minSdkVersion 15**

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

    public int pid;//Your Application Identifier

    public String gcmSenderId;//SENDER_ID used for push notifications

    public String noInternetMessage;//Message to display when no internet access

    public String hockeyAppKey;//Key for Hockey App

    public String newRelicAppKey;//Key for New Relic

    public boolean waitForPageLoadEvent = true;// Flag to indicate if the framework will wait for a page load event while playing the dynamic splash. True by default.
    
    public String video_upload_key;//API Key for the upload video service (like cameraTag API key)
}


.
.
.

    CoreData coreData = new CoreData();
    //setup your core data
    ConfigManager configManager = ConfigManager.getInstance();
    configManager.setCoreData(coreData);

```


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

To receive push notifications you will need to add the following tag to your Android Manifest file


```xml

<receiver
    android:name="com.google.android.gms.gcm.GcmReceiver"
    android:exported="true"
    android:permission="com.google.android.c2dm.permission.SEND">
    <intent-filter>
        <action android:name="com.google.android.c2dm.intent.RECEIVE" />
        <category android:name="YOUR_PACKAGE_NAME" />
    </intent-filter>
</receiver>

```

**Please before copy and pasting it change 'YOUR_PACKAGE_NAME' for your application's package name**

You will need to add a icon named "small_icon.png" in your assets folder to display it as the big image which appears once the notification is displayed to the user,
and also an image in your res/drawable (or one per density if you want to) named "small_notification_icon.png" used to display while your notification appears in the navigation bar.


**When the user clicks on the notification, you application will be opened using the launcher activity**

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

#New Relic
-keep class com.newrelic.** { *; }
-dontwarn com.newrelic.**
-keepattributes Exceptions, Signature, InnerClasses, LineNumberTable
```
