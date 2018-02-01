
## Prox SDK Android

### Requirements
The Android SDK is available for applications targeting API level 14 and above. Please not that BLE scanning is only available for users running on API level 18 and later.

### Dependencies
The Android SDK uses the Google Awareness API 11.8.0 (play-services-awareness) to gather location data. The dependency is included in the package, but may be overriden by your application. Please note that version 11 or later is required.

As the SDK is providing the possibility to target users in other contexts, it also relies on the AdvertisingId of the device. For this reason, the SDK also relies on Google Play Services Ads (play-services-ads).

Both dependencies are a part of Google Play Services (com.google.android.gms), but only those two artifacts are included with the release too make it as small as possible.

### Installation

#### Adding to your project

Add the following line to your app's `build.gradle`:

```groovy
repositories {
    maven {
        url "https://ganymede.blob.storage.net/maven/"
    }
}
```

Add the below line to your app's `build.gradle` inside the `dependencies` section:

```groovy
compile 'com.prox:sdk:1.+'
```

### Usage
The SDK is initialized automatically on launch, but it will not gather any data unless you choose to start tracking movement.
Starting tracking of the users movement is done by the following snippet:

```java
Prox.startTracking()
```

Likewise, if you for some reason wants to stop tracking the device, you can do that by calling:

```java
Prox.stopTracking()
```

*The SDK stores the previous state, so you don't have to call Prox.startTracking() every time the app launches.*

On Android 6.0 and later, user permissions are required to be able to collect location data. Even if startTracking() is called, it won't be able to gather any data for these devices. To request location authorization on Android 6.0 and above, you need to do so programatically. This is done by using standard runtime permission code requesting ACCESS_FINE_LOCATION. An example implementation is provided here:

```java
public static final int MY_PERMISSIONS_REQUEST_LOCATION = 99;

public boolean checkLocationPermission() {
    if (ContextCompat.checkSelfPermission(this,
            Manifest.permission.ACCESS_FINE_LOCATION)
            != PackageManager.PERMISSION_GRANTED) {

        // Should we show an explanation?
        if (ActivityCompat.shouldShowRequestPermissionRationale(this,
                Manifest.permission.ACCESS_FINE_LOCATION)) {

            // Show an explanation to the user
            new AlertDialog.Builder(this)
                    .setTitle(R.string.title_location_permission)
                    .setMessage(R.string.text_location_permission)
                    .setPositiveButton(R.string.ok, new DialogInterface.OnClickListener() {
                        @Override
                        public void onClick(DialogInterface dialogInterface, int i) {
                            //Prompt the user once explanation has been shown
                            ActivityCompat.requestPermissions(MainActivity.this,
                                    new String[]{Manifest.permission.ACCESS_FINE_LOCATION},
                                    MY_PERMISSIONS_REQUEST_LOCATION);
                        }
                    })
                    .create()
                    .show();


        } else {
            // No explanation needed, we can request the permission.
            ActivityCompat.requestPermissions(this,
                    new String[]{Manifest.permission.ACCESS_FINE_LOCATION},
                    MY_PERMISSIONS_REQUEST_LOCATION);
        }
        return false;
    } else {
        return true;
    }
}

@Override
public void onRequestPermissionsResult(int requestCode,
                                       String permissions[], int[] grantResults) {
    switch (requestCode) {
        case MY_PERMISSIONS_REQUEST_LOCATION: {
            // If request is cancelled, the result arrays are empty.
            if (grantResults.length > 0
                    && grantResults[0] == PackageManager.PERMISSION_GRANTED) {

                // permission was granted
                if (ContextCompat.checkSelfPermission(this,
                        Manifest.permission.ACCESS_FINE_LOCATION)
                        == PackageManager.PERMISSION_GRANTED) {

                    //Request location updates:
                    Prox.startTracking();
                }

            } else {

                // permission denied. Stop tracking if it was started for some reason.
                Prox.stopTracking();
            }
            return;
        }

    }
}
```

**Please note that for tracking to start immediately after location permission is accepted, call Prox.startTracking() as soon as the permission is granted.**

It's very important to ask for these permissions at the right time to get as many users as possible to opt in. Some guidelines and best pracices are published here: <PLACEHOLDER> 

### Configuration
All configurations used by the SDK is provided by an external endpoint. This makes it possible to change the configuration without releasing a new version of the application. It also makes it possible to use different configurations based on things like device type, android version, etc.

#### Advanced
The SDK relies on scanning jobs running in the background. As these needs an application unique jobid, it's possible to change the range being used in case of collision with your own application. This is done by adding the following in your AndroidManifest.xml:

```xml
<metadata android:name="com.prox.sdk.JobIdStartId">433453</metadata>
```
The default start index is 454343 and the SDK reserves 20 ids starting from the default.


## Permissions
The SDK used the following permissions to collect and report data:

```xml
<!-- Access the approximate location of the device  -->
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
<!-- Access an accurate location of the device  -->
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<!-- To allow reading bluetooth data  -->
<uses-permission android:name="android.permission.BLUETOOTH" />
<!-- To allow requesting bluetooth scans  -->
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN" />
<!-- To get information about the network state of the device  -->
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<!-- To get events when nearby wifis changes  -->
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<!-- To be able to trigger wifi scans -->
<uses-permission android:name="android.permission.CHANGE_WIFI_STATE" />
<!-- To send any gathered data -->
<uses-permission android:name="android.permission.INTERNET" />
<!-- To detect the current user activty (still, on foot, etc) -->
<uses-permission android:name="com.google.android.gms.permission.ACTIVITY_RECOGNITION" />


```

#### Advanced
As these permissions are automatically merged into your androidmanifest.xml, you need to remove them in your manifest if you for some reason wants to limit the number of permissions. This can be done by the following syntax:

```xml
<uses-permission android:name="android.permission.CHANGE_WIFI_STATE" tools:node="remove" />
```
*Note that you need to add the tools namespace (xmlns:tools="http://schemas.android.com/tools") in the manifest element to get this option*


## What kinds of data does the SDK collect
With the default configuration, the SDK will collect information about nearby wifis, geo location and BLE devices (iBeacon, Eddystone and RawBle). These data comes with information about the device, battery levels, etc.

The following is an example of an geo location event. All of these data are sent with each event regardless of type. So when an WIFI event is reported, so are device info and geo location info. 

```json
{
"connection": {
        "bssid": "10:3a:cb:f5:00:2a",
        "cellType": "4G",
        "level": -59,
        "mcc": 242,
        "mnc": 12,
        "ssid": "prox-guest",
        "timestamp": "2018-02-01T11:49:31+00:00",
        "type": "WIFI"
      },
      "device": {
        "activity": true,
        "adid": "1e90927e-8e19-1348-b7e0-f36732db49bc",
        "adidLimited": false,
        "appId": "com.prox.proxdemo",
        "appVersion": "1.1",
        "location": "always",
        "manufacturer": "Sony",
        "mcc": 242,
        "mnc": 12,
        "model": "F8331",
        "os": "android",
        "osSdkVersion": "24",
        "osVersion": "7.0",
        "sdkVersion": "1.0",
        "timestamp": "2018-02-01T11:49:31+00:00",
        "trackingEnabled": true
      },
      "event": {
        "altitude": 23.2,
        "bearing": 0,
        "hacc": 21.9,
        "lat": 59.9135209,
        "lon": 10.7478901,
        "provider": "SNAPSHOT",
        "speed": 0,
        "timestamp": "2018-02-01T11:49:31+00:00",
        "updated": "2018-02-01T11:48:31+00:00",
        "vacc": 12.5,
        "type": "LOCATION"
      },
      "location": {
        "altitude": 0,
        "bearing": 0,
        "hacc": 21.9,
        "lat": 59.9135209,
        "lon": 10.7478901,
        "provider": "SNAPSHOT",
        "speed": 0,
        "timestamp": "2018-02-01T11:49:31+00:00",
        "updated": "2018-02-01T11:48:31+00:00",
        "vacc": null,
        "type": "LOCATION"
      },
      "state": {
        "activity": "Still",
        "battery": 1,
        "charging": false,
        "deviceInfo": "1.1, , Sony, F8331, 24",
        "foreground": false,
        "powersave": false,
        "timestamp": "2018-02-01T11:49:31+00:00"
      }
}
```

## How does it work?
The SDK relies on Google Awareness API, and not without reason. It's using the API to look at the current state of the device, and make sure scanning is triggered less frequently if e.g. the device is still and not moving. It's also using the Awareness API to trigger scanning on intervals and when the device has moved a certain threshold. All scanning intervals and movement thresholds are configured from the cloud.

In the default configuration, the SDK will used JobScheduler on Android 5+ to further preserve battery. This makes the OS stack up any pending jobs, and make sure it only runs on optimal times. It is possible to override this behaviour through the cloud config, but recommended behaviour is to allow the OS to pick the best windows for scanning and reporting data.
