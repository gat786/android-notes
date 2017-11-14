# Networking In Android

- Android lets your application connect to the internet or any other local network and allows you to perform network operations.
- Before you add networking functionality to your app, you need to ensure that data and information within your app stays safe when transmitting it over a network. To do so, follow these networking security best practices:
  - Minimize the amount of sensitive or personal [user data](https://developer.android.com/training/articles/security-tips.html#UserData) that you transmit over the network.
  - Send all network traffic from your app over [SSL](https://developer.android.com/training/articles/security-ssl.html).
  - Consider creating a [network security configuration](https://developer.android.com/training/articles/security-config.html), which allows your app to trust custom CAs or restrict the set of system CAs that it trusts for secure communication.
- Most network-connected Android apps use HTTP to send and receive data. The Android platform includes the `HttpsURLConnection` client.

## Permissions

- In order to connect to the network, your app must have the following permissions in the manifest file:

  ```xml
  <uses-permission android:name="android.permission.INTERNET" />
  <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
  ```

## Checking the network:

- Before you perform any network operations, you must first check that are you connected to that network or internet e.t.c. For this android provides **ConnectivityManager** class. 

  ```java
  ConnectivityManager check = (ConnectivityManager) 
  this.context.getSystemService(Context.CONNECTIVITY_SERVICE);  
  ```

- Then, use **getAllNetworkInfo** to get information about all the networks.

  ```java
  NetworkInfo[] info = check.getAllNetworkInfo();
  ```

- The last thing you need to do is to check **Connected State** of the network.

  ```java
  for (int i = 0; i<info.length; i++){
     if (info[i].getState() == NetworkInfo.State.CONNECTED){
        Toast.makeText(context, "Internet is connected
        Toast.LENGTH_SHORT).show();
     }
  }
  ```




## Accessing the network


- After checking that you are connected to the internet, you can perform any network operation. Here we are fetching the html of a website from a url.

- Android provides **HttpURLConnection** and **URL** class to handle these operations. You need to instantiate an object of URL class by providing the link of website.

  _This exactly the same as URL in java, hence the full explanation is not given._

  ```java
  String link = "http://www.google.com";
  URL url = new URL(link);

  HttpURLConnection conn = (HttpURLConnection) url.openConnection();
  conn.connect();	

  InputStream is = conn.getInputStream();
  BufferedReader reader = new BufferedReader(new InputStreamReader(is, "UTF-8"));
  String webPage = "",data="";

  while ((data = reader.readLine()) != null){
     webPage += data + "\n";
  }
  ```

  ​


## Sending Email

- One can send emails on Android using implicit intents.

- Implicit intents broadcast to all installed apps on the phone, the _intention_ to perform a certain action.

- You will use **ACTION_SEND** action to launch an email client installed on your Android device. Following is simple syntax to create an intent with ACTION_SEND action.

  ```java
  Intent emailIntent = new Intent(Intent.ACTION_SEND);
  ```

- To send an email you need to 

  - use setData() method to specify **mailto:** as URI
  - use setType() method to set data type to **text/plain** 

- Android has built-in support to add TO, SUBJECT, CC, TEXT etc. fields which can be attached to the intent before sending the intent to a target email client. 

- ```java
  emailIntent.putExtra(Intent.EXTRA_EMAIL  , new String[]{"Recipient"});
  emailIntent.putExtra(Intent.EXTRA_SUBJECT, "subject");
  emailIntent.putExtra(Intent.EXTRA_TEXT   , "Message Body");
  ```

  ​