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

- When an application starts and it does not have any other components running, the Android system starts a new Linux process.
- This process has a single thread of execution. 
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

- When an application is launched, the system creates a thread of execution for the application, called "main." 

- This thread is very important because it is in charge of dispatching events to the user interface.

-  As such, the main thread is also sometimes called the UI thread. 

- The system does *not* create a separate thread for each instance of a component. 

- All components that run in the same process are instantiated in the UI thread

- System calls to each component are dispatched from that thread. 

- Consequently, methods that respond to system callbacks (such as `onCreate()`) always run in the UI thread of the process.

- When your app performs intensive work in response to user interaction, this single thread model can yield poor performance

- Specifically, if everything is happening in the UI thread, performing long operations such as network access or database queries will block the whole UI. 

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
  - This method is executed in a different process (remotely), and the results are returned back to the caller activity/component.
- This process 
  - breaks down a method and its data to a level the operating system can understand, 
  - transmits it from the local process and address space to the remote process and address space,
  - then reassembles and reenacts the call there. 
- Return values are then transmitted in the opposite direction. 
- Android provides all the code to perform these IPC transactions, the developer only needs to define and implement the RPC programming interface.
- To perform IPC, your application must bind to a service, using `bindService()`.
  - A `Service` is an application component that can perform long-running operations in the background, and it does not provide a user interface.
  - Another application component can start a service, and it continues to run in the background even if the user switches to another application.