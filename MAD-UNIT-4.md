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

