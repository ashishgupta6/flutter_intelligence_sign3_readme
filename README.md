# Sign3 SDK Integration Guide for Flutter

The Sign3 SDK is a fraud prevention toolkit designed to assess device security, detecting potential risks such as rooted devices, VPN connections, or remote access, and much more. By providing insights into the device's safety, it enhances security measures against fraudulent activities and ensures a robust protection system.

## Adding Sign3 SDK to Your Project

### Add the latest version to your `pubspec.yaml` file.
1. We continuously enhance our fraud library with new features, bug fixes, and security updates. To stay protected against evolving fraud risks, we recommend updating to the latest SDK version.
    - Visit [latest_version](https://pub.dev/packages/flutter_intelligence_sign3) for the latest version and check the Changelog for more details.

      ```dependency
      dependencies:
        flutter_intelligence_sign3: ^x.x.x (Refer to the latest version)
      ```

### Create `.env` file

1. In the root directory of your Flutter project, create a .env file
    - Add the following content to the `.env` file:

      ``` credential
      SIGN3_USERNAME=provided in credential doc
      SIGN3_PASSWORD=provided in credential doc
      SIGN3_REPO_URL=https://sign3.jfrog.io/artifactory/intelligence-generic-local/
       ```

### Using Project Level Gradle Dependency
1. **Add Sign3 SDK to the Dependency Block**
    - In the Android folder of your Flutter project, open the project-level `build.gradle` file, add the following line, and sync the project. You can collect the **username** and **password** from the credentials document.

      ```groovy
      def envProperties = new Properties()
      def envFile = rootProject.file("../.env")
      if (envFile.exists()) {
          envFile.withInputStream { stream ->
              envProperties.load(stream)
          }
          println ".env file loaded successfully."
      } else {
          println "Error: .env file not found."
      }
      
      allprojects {
         repositories {
             def sign3RepoUrl = envProperties['SIGN3_REPO_URL'] ?: ""
             def sign3Username = envProperties['SIGN3_USERNAME'] ?: ""
             def sign3Password = envProperties['SIGN3_PASSWORD'] ?: ""
             if (!sign3RepoUrl.isEmpty() && sign3RepoUrl.startsWith("https://") && !sign3Username.isEmpty() && !sign3Password.isEmpty()) {
                 maven {
                     url sign3RepoUrl
                     credentials {
                         username = sign3Username
                         password = sign3Password
                     }
                 }
             } else {
                 println "Error: Invalid repository URL or missing credentials in .env file."
             }
         }
      }
      ```
<br>

## Initializing the SDK
1. In Android, create an `Application` class in the Android folder of your Flutter project, initialize the Sign3 SDK in the `onCreate()` method of the class, and add the Application class to your `AndroidManifest.xml` file.
2. In iOS, initialize the SDK in your AppDelegate class within the application(_:didFinishLaunchingWithOptions:) method.
3. This SDK needs 2 required permissions for Android to achieve higher accuracy.
    - ACCESS_FINE_LOCATION
    - READ_PHONE_STATE
4. Use the ClientID and Client Secret shared with the credentials document.

## For Android
### For Java
```java
public class MyApplication extends FlutterApplication {
    @Override
    public void onCreate() {
        super.onCreate();
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
            Sign3IntelligencePlugin sign3IntelligencePlugin = new Sign3IntelligencePlugin();
            if (sign3IntelligencePlugin.stop()) return;
            OptionsBuilder.INSTANCE.build(
                    "<SIGN3_CLIENT_ID>",
                    "<SIGN3_CLIENT_SECRET>",
                    OptionsBuilder.ENV_DEV); // For Prod: Options.ENV_PROD, For Dev: Options.ENV_DEV
            sign3IntelligencePlugin.initAsync(this);
        }
    }
}
```

### For Kotlin
```kotlin
class MyApplication : FlutterApplication() {
    override fun onCreate() {
        super.onCreate()
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
            val sign3IntelligencePlugin = Sign3IntelligencePlugin()
            if (sign3IntelligencePlugin.stop()) return
            OptionsBuilder.build(
                clientId = "<SIGN3_CLIENT_ID>",
                secret = "<SIGN3_CLIENT_SECRET>",
                env = OptionsBuilder.ENV_DEV  // For Prod: Options.ENV_PROD, For Dev: Options.ENV_DEV
            )
            sign3IntelligencePlugin.initAsync(this)
        }
    }
}
```

### For AndroidManifest.xml
``` android menifest
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

<!-- Uncomment below line if minSdk is lower than 23 -->
<!-- <uses-sdk tools:overrideLibrary="com.sign3.intelligence.flutter_intelligence_sign3"/> -->

    <application
        android:name=".MyApplication">
        <!-- Add other components like activities here -->
    </application>

</manifest>
```


## For iOS
### Import Library

```swift
import flutter_intelligence_sign3
```

```swift
let flutterIntelligenceSign3Plugin = FlutterIntelligenceSign3Plugin()
OptionsBuilder.build(
    clientId: "<SIGN3_CLIENT_ID>",
    secret: "<SIGN3_CLIENT_SECRET>",
    env: OptionsBuilder.ENV_DEV)  // For Prod: OptionsBuilder.ENV_PROD, For Dev: OptionsBuilder.ENV_DEV
flutterIntelligenceSign3Plugin.initAsync()
```

<br>

## Get Session ID

1. The Session ID is the unique identifier of a user's app session and serves as a reference point when retrieving the device result for that session.
2. The Session ID follows the OS lifecycle management, in line with industry best practices. This means that a user's session remains active as long as the device maintains it, unless the user terminates the app or the device runs out of memory and has to kill the app.

 ```dart
Future<void> getSessionID() async {
  var sessionId = await Sign3Intelligence.getSessionId();
}
```
<br>

## Fetch Device Intelligence Result

1. To fetch the device intelligence data refer to the following code snippet.
2. IntelligenceResponse and IntelligenceError models are exposed by the SDK.

```dart
Future<void> getIntelligence() async {
  try {
    var sign3IntelligenceResponse = await Sign3Intelligence.getIntelligence();
    // Do something with the response
  } catch (e) {
    // Handle the error message 
  }
}
```
<br>

## Optional
1.	You can add optional parameters like UserId, Phone Number, etc., at any time and update the instance of Sign3Intelligence.
2.	Once the options are updated, they get reset. Clients need to explicitly update the options again to ingest them, or else the default value of OTHERS in userEventType will be sent to the backend.
3.	You need to call **getIntelligence()** function whenever you update the options.
4.	To update the Sign3Intelligence instance with optional parameters, including additional attributes, you can use the following examples.

```dart
UpdateOptions getUpdatedOptions() {
  Map<String, String> additionalAttributes = {
    "TRANSACTION_ID": "<TRANSACTION_ID>",
    "DEPOSIT": "<AMOUNT>",
    "WITHDRAWAL": "<AMOUNT>",
    "METHOD": "UPI/CARD/NET_BANKING/WALLET",
    "STATUS": "SUCCESS/FAILURE",
    "CURRENCY": "USD/INR/GBP/etc.",
    "TIMESTAMP": DateTime.now().millisecondsSinceEpoch.toString(),
  };

  UpdateOptions updateOptions = UpdateOptionsBuilder()
      .setPhoneNumber("<phone_number>")
      .setUserId("<user_id>")
      .setPhoneInputType(PhoneInputType.GOOGLE_HINT)
      .setOtpInputType(OtpInputType.AUTO_FILLED)
      .setUserEventType(UserEventType.TRANSACTION)
      .setMerchantId("<merchant_id>")
      .setAdditionalAttributes(additionalAttributes)
      .build();
  return updateOptions;
}

Future<void> updateOptions() async {
  await Sign3Intelligence.updateOptions(getUpdatedOptions());
}

Future<void> getIntelligence() async {
  try {
    var sign3IntelligenceResponse = await Sign3Intelligence.getIntelligence();
    // Do something with the response
  } catch (e) {
    // Handle the error message 
  }
}
```
<br>

## Sample Device Result Response

### Successful Intelligence Response

```response
{
    "sessionId": "e68caf89-2f68-480a-9368-2036d833c86b",
    "requestId": "403ad427-5018-47b9-b6e8-790e17a78201",
    "newDevice": false,
    "deviceId": "43fccb70-d64a-4c32-a251-f07c082d7034",
    "vpn": false,
    "proxy": false,
    "emulator": true, // For Android
    "simulator": false, // For iOS
    "jailBroken": true, // For iOS
    "cloned": false,
    "geoSpoofed": false,
    "rooted": false,
    "ip": "106.219.161.71",
    "remoteAppProviders": false,
    "remoteAppProvidersCount": 3,
    "mirroredScreen": false,
    "hooking": true,
    "factoryReset": true,
    "appTampering": true,
    "sessionRiskScore": 99.50516,
    "deviceRiskScore": 99.50516,
    "clientUserIds": [
        "difansd23r32",
        "2390ksdfaksd"
    ],
    "sign3UserIds": [
        "13asefnn324"
    ],
    "gpsLocation": {
        "address": "F2620, Block F, Sushant Lok III, Sector 57, Gurugram, Haryana 122011, India",
        "adminArea": "Haryana",
        "countryCode": "IN",
        "countryName": "India",
        "featureName": "F2620",
        "latitude": "28.420385999999997",
        "locality": "Gurugram",
        "longitude": "77.088926",
        "postalCode": "122011",
        "subAdminArea": "Gurgaon Division",
        "subLocality": "Sector 57"
    },
    "ipDetails": {
        "country": "IN",
        "fraudScore": 27.0,
        "city": "New Delhi",
        "isp": null,
        "latitude": 28.60000038,
        "region": "National Capital Territory of Delhi",
        "asn": "",
        "longitude": 77.19999695
    },
    "simInfo": {
        "simIds": [
            {
                "simSlotIndex": 0,
                "carrierName": "Android",
                "id": 1
            }
        ],
        "totalSimUsed": 10
    },
    "appAnalytics": {
        "affinity": {
            "entertainment": 0.5,
            "gaming": 0.6,
            "productivity": 0.8,
            "food_and_drink": 0.4
        }
    },
    "deviceMeta": {
        "cpuType": "Qualcomm Technologies, Inc SM7325",
        "product": "I2126i",
        "androidVersion": "14",
        "storageAvailable": "29340553216",
        "storageTotal": "111156146176",
        "model": "I2126",
        "screenResolution": "1080x2316",
        "brand": "iQOO",
        "totalRAM": "7679795200"
    },
    "additionalData": {},
    "factoryResetTime": 1743419662000
}
```
### Error Response

```error
{
  "requestId": "7e12d131-2d90-4529-a5c7-35f457d86ae6",
  "errorMessage": "Sign3 Server Error"
}
```
<br>

## Intelligence Response

The intelligence response includes the following keys:

- **requestId**: A unique identifier for the specific request.
- **newDevice**: Indicates if the device is new.
- **deviceId**: A unique identifier for the device.
- **vpn**: Indicates whether a VPN is active on the device.
- **proxy**: Indicates whether a proxy server is in use.
- **emulator**: Indicates if the app is running on an emulator.
- **remoteAppProviders**: Indicates whether any remote applications are installed on the device.
- **mirroredScreen**: Indicates if the device's screen is being mirrored.
- **cloned**: Indicates if the user is using a cloned instance of the app.
- **geoSpoofed**: Indicates if the device's location is being faked.
- **rooted**: Indicates if the device has been modified for root access.
- **sessionRiskScore**: A score representing the risk level of the session.
- **hooking**: Indicates if the app has been altered by malicious code.
- **factoryReset**: Indicates if a suspicious factory reset has been performed.
- **appTampering**: Indicates if the app has been modified in an unauthorized way.
- **clientUserIds**: An array of user IDs assigned by the client that a device has seen till now.
- **gpsLocation**: Details of the device's current GPS location, including latitude, longitude, and address information.
- **ip**: The current IP address of the device.
- **ipDetails**: Object added to capture ip related information and fraudScore related to ip address.
- **sign3UserIds**: This will contain Sign3 generated userIds list till now the device has seen. Note: The logic for generating userId will be configured as per your business logic and can be customized.
- **simInfo**: It will contain information like total sims used in a phone in its lifecycle, current sim+slot details.
- **remoteAppProvidersCount**: The number of remote application providers detected on the device.
- **deviceRiskScore**: The risk score of the device. Note: sessionRiskScore is derived from the latest state of the device but deviceRiskScore also factors in the historical state of the device (whether a device was rooted in any of the past sessions).
- **deviceMeta**: Contains all device-related information such as brand, model, screen resolution, total storage, etc.
- **appAnalytics**: This object provides a categorized mapping of applications, as classified by the Google Play Store. Categories may include, but are not limited to, books_and_reference, business, communication, lifestyle, etc. For applications lacking detailed categorization on the Play Store, their packages are grouped under a category labeled others (e.g., "com.onecode.battle.gfx.pro"). Note that by default, it is not included in the response. Please get in touch with us to enable the feature.
- **additionalData**: Reserved for any extra or custom data not present in the IntelligenceResponse, providing a customized response based on specific requirements.
