# Working with Bluetooth and WiFi

## Bluetooth

- The Android platform includes support for the Bluetooth network stack, which allows a device to wirelessly exchange data with other Bluetooth devices.

- In order for Bluetooth-enabled devices to transmit data between each other, they must first form a **channel of communication** using a ***pairing*** process. 

- One device, a ***discoverable device***, makes itself available for incoming connection requests. 

- Another device **finds** the **discoverable** device using a ***service discovery*** process. 

- After the discoverable device accepts the pairing request, the two devices complete a ***bonding* process** where they **exchange security keys**. 

  - The devices cache these keys for later use. 

- After the pairing and bonding processes are complete, the **two devices exchange information.** 

- When the session is complete, the device that initiated the pairing request releases the channel that had linked it to the discoverable device. 

- The two devices remain bonded, however, so they can reconnect automatically during a future session as long as 

  - they're in range of each other 

    ​	and 

  - neither device has removed the bond.

## BluetoothAdapter

- First of all, to use Bluetooth, the application must declare the following permissions in the manifest:

  ```xml
  <manifest ... >
    <uses-permission android:name="android.permission.BLUETOOTH" />
    <uses-permission android:name="android.permission.BLUETOOTH_ADMIN" />
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
    ...
  </manifest>
  ```

- Android provides BluetoothAdapter class to communicate with Bluetooth. 

  ```java
  private BluetoothAdapter BA;
  BA = BluetoothAdapter.getDefaultAdapter();
  ```

- Send an Intent to the system, which will generate a popup requesting to turn on Bluetooth.

  ```java
  Intent turnOn = new Intent(BluetoothAdapter.ACTION_REQUEST_ENABLE);
  startActivityForResult(turnOn, 0);     
  ```

- Once done, you can get a list of existing ***bonded*** devices using...

  ```java
  private Set<BluetoothDevice>pairedDevices;
  pairedDevices = BA.getBondedDevices();
  ```

- There's another Intent, that can make your own phone visible to other devices instead:

  ```java
  Intent discoverableIntent =
          new Intent(BluetoothAdapter.ACTION_REQUEST_DISCOVERABLE);
  //INTENT

  discoverableIntent.putExtra(BluetoothAdapter.EXTRA_DISCOVERABLE_DURATION, 300);
  //HOW LONG TO KEEP DEVICE DISCOVERABLE (SECONDS)

  startActivity(discoverableIntent);
  //CALL
  ```

## Wi-Fi

- Android allows applications to access to view the access the state of the wireless connections at very low level. 
- Application can access almost all the information of a wifi connection.
- The information that an application can access includes 
  - connected network's link speed
  - IP address
  - negotiation state
  - etc
- Applications can also 
  - scan,
  - add,
  - save,
  - and terminate Wi-Fi connections.
- Android provides **WifiManager** API to manage all aspects of WIFI connectivity. 

### WiFiManager

- We can create WifiManager object using

  ```java
  WifiManager mainWifiObj;
  mainWifiObj = (WifiManager) getSystemService(Context.WIFI_SERVICE); 
  ```

- To scan for Wi-Fi networks, use the `startScan()` method of the WifiManager object. 

- It returns ScanResult objects.

- You can access any object via the `get` method of the List class.

  ```java
  List<ScanResult> wifiScanList = mainWifiObj.getScanResults();
  String data = wifiScanList.get(0).toString();
  ```

- To connect a a Wi-Fi network, first create it's configuration.

  ```java
  WifiConfiguration conf = new WifiConfiguration();
  //create conf object

  conf.SSID = "\"" + networkSSID + "\"";
  //networkSSID is the name of the network

  conf.preSharedKey = "\""+ networkPass +"\"";
  //add a password to the conf object

  mainWifiObj.addNetwork(conf);
  //tell android to save this configuration
  ```

- Now, to actually connect to the network, we need to retrieve a list of configured networks.

  ```java
  List<WifiConfiguration> list = wifiManager.getConfiguredNetworks();
    
  }
  ```

- Loop through, to find the network we configured earlier...

  ```java
  List<WifiConfiguration> list = wifiManager.getConfiguredNetworks();

  for( WifiConfiguration i : list ) {
      if(i.SSID != null && i.SSID.equals("\"" + networkSSID + "\"")) {
           
      }           
   }
  ```

- Disconnect from any existing networks, enable the one we want, and connect again.

  ```java
  List<WifiConfiguration> list = wifiManager.getConfiguredNetworks();

  for( WifiConfiguration i : list ) {
      if(i.SSID != null && i.SSID.equals("\"" + networkSSID + "\"")) {
        
           wifiManager.disconnect();
           wifiManager.enableNetwork(i.networkId, true);
           wifiManager.reconnect();               

           break;
        
      }           
   }
  ```

---

# Processes and threads

- When an **application starts** and it does not have any other components running, the Android system **starts** a **new Linux process**.
- This process has a **single thread of execution**. 
- By default, all components of the same application run in the same process and thread (called the "main" thread). 
- If an application component starts and there already exists a process for that application (because another component from the application exists), then the component is started within that process and uses the same thread of execution. 
- However, you can arrange for different components in your application to run in separate processes, and you can create additional threads for any process.

> Note: The section below, called **Processes**, is not mentioned in the Syllabus. However, it is impossible to understand Threads without it, so I included it anyways.

## Processes

- By default, all components of the same application run in the same process and most applications should not change this. 
- However, if you find that you need to control which process a certain component belongs to, you can do so in the manifest file.
- The manifest entry for all components supports an `android:process` attribute that can specify a process in which that component should run. 
- Setting the `android:process` attribute to the `<application>` component sets that process to all it's children.
- Android might decide to shut down a process at some point, when memory is low and required by other processes that are more immediately serving the user. 
  - Application components running in the process that's killed are consequently destroyed. 
  - A process is started again for those components when there's again work for them to do.
  - When deciding which processes to kill, the Android system weighs their relative importance to the user. 
    - For example, it more readily shuts down a process hosting activities that are no longer visible on screen, compared to a process hosting visible activities. 
    - The decision whether to terminate a process, therefore, depends on the state of the components running in that process.

## Threads

- When an **application** is **launched**, the system **creates** a **thread** of execution for the application, called "**main**." 

- This thread is **very important** because it is in charge of dispatching events to the **user interface.**

-  As such, the **main thread** is also sometimes **called** the **UI thread.** 

- The system does ***not*** create a **separate** thread for **each instance** of a component. 

- All components that run in the same process are instantiated in the UI thread

- System calls to each component are dispatched from that thread. 

- Consequently, methods that respond to system callbacks (such as `onCreate()`) always run in the UI thread of the process.

- When your app performs **intensive work** in response to user interaction, this **single thread** model can yield **poor performance**.

- Specifically, **if everything is happening in the UI thread**, performing **long operations** such as network access or database queries **will block the whole UI.** 

  > **Extra GK:**
  >
  > When the thread is blocked, no events can be dispatched, including drawing events. From the user's perspective, the application appears to hang. Even worse, if the UI thread is blocked for more than a few seconds (about 5 seconds currently) the user is presented with the infamous "[application not responding](http://developer.android.com/guide/practices/responsiveness.html)" (ANR) dialog. The user might then decide to quit your application and uninstall it if they are unhappy.

## Worker Threads

- Because of the single threaded model described above, if you want to keep the app responsive,  you should not block the UI thread. 
- If you have operations to perform that are not instantaneous, you should make sure to do them in separate threads ("**background**" or "**worker**" threads).
- However, note that you cannot update the UI from any thread other than the UI thread or the "main" thread.

## Async Tasks

- `AsyncTask` allows you to perform asynchronous work on your user interface. 

- It performs the blocking operations in a worker thread and then **publishes** the **results** on the **UI thread**, without requiring you to handle threads and/or handlers yourself.

- To use it, you must 

  - subclass `AsyncTask` 

  - and implement the `doInBackground()` callback method, 

    which runs in a pool of background threads. 

- To update your UI, you should implement `onPostExecute()`, which delivers the result from `doInBackground()` and runs in the UI thread, so you can safely update your UI. 

- You can then run the task by calling `execute()` from the UI thread.

## Interprocess Communication

- Android offers a mechanism for **InterProcess Communication (IPC)** using **Remote Procedure Calls (RPCs)**.
- RPCs basically let the activity, or any application component, call a method. 
  - This method is executed in a different process (**remotely**), and the results are returned back to the caller activity/component.
- This process 
  - **breaks down a method** and its **data** to a level the operating system can understand, 
  - **transmits** it **from** the **local** process and address space **to** the **remote** process and address space,
  - then **reassembles and reenacts the call** there. 
- **Return values** are then **transmitted** in the **opposite direction.** 
- Android provides all the code to perform these IPC transactions, the developer only needs to define and implement the RPC programming interface.
- To perform IPC, your application must bind to a service, using `bindService()`.
  - A `Service` is an application component that can perform long-running operations in the background, and it does not provide a user interface.
  - Another application component can start a service, and it continues to run in the background even if the user switches to another application.

# Working with Graphics and Animation

## Working with Graphics

### Drawable

- A drawable resource is a general concept for a graphic that can 

  - be drawn to the screen (and which you can retrieve with APIs such as `getDrawable(int)`)

    ​	or 

  - apply to another XML resource with attributes such as `android:drawable` and `android:icon`. 

- There are several different types of drawables:

- [Bitmap File](https://developer.android.com/guide/topics/resources/drawable-resource.html#Bitmap)

  ​	A bitmap graphic file (`.png`, `.jpg`, or `.gif`). Creates a `BitmapDrawable`.

- [Nine-Patch File](https://developer.android.com/guide/topics/resources/drawable-resource.html#NinePatch) 

  ​	A PNG file with stretchable regions to allow image resizing based on content. Certain parts of the image resize, while other simply stretch like vectors. 	

- [Layer List](https://developer.android.com/guide/topics/resources/drawable-resource.html#LayerList)

    A Drawable that manages an array of other Drawables. These are drawn in array order, so the element with the largest index is be drawn on 	top. Creates a `LayerDrawable`.

- [State List](https://developer.android.com/guide/topics/resources/drawable-resource.html#StateList)

    ​	  An XML file that references different bitmap graphics for different states (for example, to use a different image when a button is pressed). Creates a `StateListDrawable`.

- [Transition Drawable](https://developer.android.com/guide/topics/resources/drawable-resource.html#Transition)

      An XML file that defines a drawable that can cross-fade between two drawable resources. Creates a `TransitionDrawable`.

- [Shape Drawable](https://developer.android.com/guide/topics/resources/drawable-resource.html#Shape)

      An XML file that defines a geometric shape, including colors and gradients. Creates a `ShapeDrawable`.

### ShapeDrawable

This is a generic shape defined in XML.

Syntax as follows:

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape=["rectangle" | "oval" | "line" | "ring"] >
    <corners
        android:radius="integer"
        android:topLeftRadius="integer"
        android:topRightRadius="integer"
        android:bottomLeftRadius="integer"
        android:bottomRightRadius="integer" />
    <gradient
        android:angle="integer"
        android:centerX="float"
        android:centerY="float"
        android:centerColor="integer"
        android:endColor="color"
        android:gradientRadius="integer"
        android:startColor="color"
        android:type=["linear" | "radial" | "sweep"]
        android:useLevel=["true" | "false"] />
    <padding
        android:left="integer"
        android:top="integer"
        android:right="integer"
        android:bottom="integer" />
    <size
        android:width="integer"
        android:height="integer" />
    <solid
        android:color="color" />
    <stroke
        android:width="integer"
        android:color="color"
        android:dashWidth="integer"
        android:dashGap="integer" />
</shape>
```

Example:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<shape
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="rectangle">
    <stroke
        android:width="2dp"
        android:color="#FFFFFFFF" />
    <gradient
        android:endColor="#DDBBBBBB"
        android:startColor="#DD777777"
        android:angle="90" />
    <corners
        android:bottomRightRadius="7dp"
        android:bottomLeftRadius="7dp"
        android:topLeftRadius="7dp"
        android:topRightRadius="7dp" />
</shape>
```

## Hardware Acceleration

- Beginning in Android 3.0 (API level 11), the Android 2D rendering pipeline supports hardware acceleration, **meaning that all drawing operations** that are performed on a `View`'s canvas **use the GPU**. 

- Because of the increased resources required to enable hardware acceleration, your app will consume more RAM.

- You can control hardware acceleration at the following levels:

  - Application

    - In your Android manifest file, add the following attribute to the [``](https://developer.android.com/guide/topics/manifest/application-element.html) tag to enable hardware acceleration for your entire application:

      ```xml
      <application android:hardwareAccelerated="true" ...>
      ```

  - Activity

    - If your application does not behave properly with hardware acceleration turned on globally, you can control it for individual activities as well. 

    - To enable or disable hardware acceleration at the activity level, 

      ```xml
      <application android:hardwareAccelerated="true">
          <activity ... />
          <activity android:hardwareAccelerated="false" />
      </application>
      ```

      ​

  - Window

    - If you need even more fine-grained control, you can enable hardware acceleration for a given window with the following code:

      ```java
      getWindow().setFlags(
          WindowManager.LayoutParams.FLAG_HARDWARE_ACCELERATED,
          WindowManager.LayoutParams.FLAG_HARDWARE_ACCELERATED);
      ```

      ​

  - View

    - You can disable hardware acceleration for an individual view at runtime with the following code:

      ```java
      myView.setLayerType(View.LAYER_TYPE_SOFTWARE, null);
      ```

### Hardware Accelerated Drawing Model

- Instead of executing the drawing commands immediately, the Android system records them inside display lists
- They contain the output of the view hierarchy’s drawing code. 
- Another optimization is that the Android system only needs to record and update display lists for views marked **dirty** by an `invalidate()` call. 
- Views that have not been invalidated can be redrawn simply by re-issuing the previously recorded display list. 
- Using display lists also benefits animation performance because setting specific properties, such as alpha or rotation, does not require invalidating the targeted view (it is done automatically). 
- This optimization also applies to views with display lists (any view when your application is hardware accelerated.)

## Animations

- The Android framework provides two animation systems: 
  - property animation  
  - view animation. 
- Both animation systems are viable options, but the property animation system, in general, is the preferred method to use.
  - This is because it is more flexible and offers more features. 
- In addition to these two systems, you can utilize Drawable animation, which allows you to load drawable resources and display them one frame after another.

### Property Animation

- The property animation system is a robust framework that allows you to animate almost anything. 

- You can define an animation to change any object property over time, regardless of whether it draws to the screen or not. 

- A property animation changes a property's (a field in an object) value over a specified length of time. 

- To animate something, you specify the object property that you want to animate, *(such as an object's position on the screen)*, how long you want to animate it for, and what values you want to animate between.

- Eg:

  - Set the animation properties in xml:

    ```xml
    <animator xmlns:android="http://schemas.android.com/apk/res/android"
        android:duration="1000"
        android:valueType="floatType"
        android:valueFrom="0f"
        android:valueTo="-100f" />
    ```

  - Animate!

    ```java
    ValueAnimator xmlAnimator = (ValueAnimator) AnimatorInflater.loadAnimator(this,
            R.animator.animator);
    //LOAD FROM XML FILE

    xmlAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
        @Override
        public void onAnimationUpdate(ValueAnimator updatedAnimation) {
            float animatedValue = (float)updatedAnimation.getAnimatedValue();
            textView.setTranslationX(animatedValue);
          //MOVE TEXTVIEW TO NEW VALUE
        }
    });
    //ABOVE CODE SETS UP THE ANIMATION

    xmlAnimator.start();
    //THIS IS THE LINE THAT ACTUALLY STARTS THE ANIMATION
    ```

### View Animation

- You can use the view animation system to perform **tweened animation** on Views. 

- Tween animation calculates the animation with information such as the start point, end point, size, rotation, and other common aspects of an animation.

- A tween animation can perform a series of simple transformations (position, size, rotation, and transparency) on the contents of a View object. 

  - So, if you have a `TextView` object, you can move, rotate, grow, or shrink the text. 
  - If it has a background image, the background image will be transformed along with the text. 

- The `animation package` provides all the classes used in a tween animation.

- Eg:

  - xml file:

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <set xmlns:android="http://schemas.android.com/apk/res/android">

       <rotate xmlns:android="http://schemas.android.com/apk/res/android"
          android:fromDegrees="0"
          android:toDegrees="360"
          android:pivotX="50%"
          android:pivotY="50%"
          android:duration="5000" >
       </rotate>
       
       <rotate xmlns:android="http://schemas.android.com/apk/res/android"
          android:startOffset="5000"
          android:fromDegrees="360"
          android:toDegrees="0"
          android:pivotX="50%"
          android:pivotY="50%"
          android:duration="5000" >
       </rotate>
       
    </set>
    ```

  - java code:

    ```java
    ImageView image = (ImageView)findViewById(R.id.imageView);
    //THE THING THAT WILL SPIN ROUND ROUND

     Animation animation = AnimationUtils.loadAnimation(
       getApplicationContext(), R.anim.myanimation);
    //LOAD THE ANIMATION FROM XML

    image.startAnimation(animation);
    //START ANIMATION
    ```



### Drawable Animation

- Drawable animation lets you load a series of Drawable resources one after another to create an animation. 

- This is a traditional animation in the sense that it is created with a sequence of different images, played in order, like a roll of film. 

- The XML file consists of 

  - an `<animation-list>` element as the root node 
  - a series of child `<item>` nodes that each define a frame: 
    - Each frame is a drawable resource for the frame 
  - and the frame duration. 

- Eg xml:

  ```xml
  <animation-list xmlns:android="http://schemas.android.com/apk/res/android"
      android:oneshot="true">
      <item android:drawable="@drawable/rocket_thrust1" android:duration="200" />
      <item android:drawable="@drawable/rocket_thrust2" android:duration="200" />
      <item android:drawable="@drawable/rocket_thrust3" android:duration="200" />
  </animation-list>
  ```

- This animation runs for just three frames. 

- By setting the `android:oneshot` attribute of the list to **true**, it will cycle just once then stop and hold on the last frame. 

- If it is set **false** then the animation will loop. 

- Eg java:

  ```java
  ImageView rocketImage = (ImageView) findViewById(R.id.rocket_image);
  //HAVE AN IMAGEVIEW

  rocketImage.setBackgroundResource(R.drawable.rocket_thrust);
  //SET THE BACKGROUND TO USE THE XML FILE WE CREATED ABOVE

  AnimationDrawable rocketAnimation;
  rocketAnimation = (AnimationDrawable) rocketImage.getBackground();
  //CREATE OUR ANIMATION OBJECT
  //AND LINK IT TO THE IMAGEVIEW

  rocketAnimation.start();
  //ANIMAAAAATTTTEEEEEE
  ```



---

> **Excluded topics**: Cloud to Device Messaging using Google Firebase
> Cloud Messaging, Publishing the App, Best Practices for Performance.
>
> Refer to CC's notes, or ask Google bhaiyya.

