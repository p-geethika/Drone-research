# Setting Up DJI Mobile SDK in Android Studio

This is how I tried to setup the DJI Mobile SDK in an Android Studio of version 3.1.20. 

1. **Android Studio**: We should install latest version of Android Studio.
2. **DJI Developer Account**: Register for a [DJI Developer Account](https://developer.dji.com/register/) to create an App Key.

## Steps

### 1. Generating an App Key

1. Log in to the [DJI Developer Website](https://developer.dji.com/).
2. Navigate to the **Developer** section and create a new App.
3. Fill in the required details such as App Name and Package Name.
4. Note down the App Key generated for your project.

### 2. Creating a New Project in Android Studio

1. Open Android Studio and select **Create New Project**.
2. Enter the Application Name as `sample` and the Package Name as `com.hotspotaerial.sample`.
3. Choose **Kotlin** as the programming language.
4. Set the **Minimum SDK** to API 21.

### 3. Configuring Gradle Scripts

1. Open the `build.gradle (Module: app)` file and add the following dependencies:

    ```gradle
    dependencies {
        implementation 'com.dji:dji-sdk:4.16'
        implementation 'androidx.multidex:multidex:2.0.1'
        implementation 'com.squareup:otto:1.3.8'
    }

    repositories {
        mavenCentral()
        maven { url 'https://developer.dji.com/maven2' }
    }
    ```

2. Create an `accessory_filter.xml` file in the `res/xml` directory with the following content:

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <resources>
        <usb-accessory model="DJI"/>
    </resources>
    ```

### 4. Application Registration

1. Create a new Kotlin class named `MApplication`:

    ```kotlin
    package com.hotspotaerial.sample

    import android.app.Application
    import com.secneo.sdk.Helper

    class MApplication : Application() {
        override fun onCreate() {
            super.onCreate()
            Helper.install(this)
        }
    }
    ```

2. Modify the `AndroidManifest.xml` to include necessary permissions and declare the `MApplication` class:

    ```xml
    <manifest xmlns:android="http://schemas.android.com/apk/res/android"
        package="com.hotspotaerial.sample">

        <uses-permission android:name="android
    .permission.INTERNET"/>
        <uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/>
        <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
        <uses-permission android:name="android.permission.VIBRATE"/>
        <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
        <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
        <uses-permission android:name="android.permission.READ_PHONE_STATE"/>
        <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>
        <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
        <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
        <uses-permission android:name="android.permission.BLUETOOTH"/>
        <uses-permission android:name="android.permission.BLUETOOTH_ADMIN"/>
        <uses-permission android:name="android.permission.BROADCAST_STICKY"/>

        <application
            android:name=".MApplication"
            android:allowBackup="true"
            android:icon="@mipmap/ic_launcher"
            android:label="@string/app_name"
            android:roundIcon="@mipmap/ic_launcher_round"
            android:supportsRtl="true"
            android:theme="@style/AppTheme">
            <meta-data
                android:name="com.dji.sdk.API_KEY"
                android:value="Your_App_Key_Here"/>

            <activity android:name=".MainActivity">
                <intent-filter>
                    <action android:name="android.intent.action.MAIN" />
                    <category android:name="android.intent.category.LAUNCHER" />
                </intent-filter>
            </activity>
        </application>
    </manifest>
    ```

### 5. Implementing MainActivity

1. Create a new Kotlin file named `MainActivity.kt` and implement the following code:

    ```kotlin
    package com.hotspotaerial.sample

    import android.Manifest
    import android.content.pm.PackageManager
    import android.os.Bundle
    import android.util.Log
    import androidx.appcompat.app.AppCompatActivity
    import androidx.core.app.ActivityCompat
    import androidx.core.content.ContextCompat
    import dji.sdk.sdkmanager.DJISDKManager

    class MainActivity : AppCompatActivity() {

        companion object {
            const val TAG = "MainActivity"
            const val REQUEST_PERMISSION_CODE = 12345
        }

        private val REQUIRED_PERMISSION_LIST = arrayOf(
            Manifest.permission.INTERNET,
            Manifest.permission.ACCESS_WIFI_STATE,
            Manifest.permission.ACCESS_NETWORK_STATE,
            Manifest.permission.VIBRATE,
            Manifest.permission.WRITE_EXTERNAL_STORAGE,
            Manifest.permission.READ_EXTERNAL_STORAGE,
            Manifest.permission.READ_PHONE_STATE,
            Manifest.permission.RECEIVE_BOOT_COMPLETED,
            Manifest.permission.ACCESS_COARSE_LOCATION,
            Manifest.permission.ACCESS_FINE_LOCATION,
            Manifest.permission.BLUETOOTH,
            Manifest.permission.BLUETOOTH_ADMIN,
            Manifest.permission.BROADCAST_STICKY
        )
        private val missingPermissions: MutableList<String> = ArrayList()

        override fun onCreate(savedInstanceState: Bundle?) {
            super.onCreate(savedInstanceState)
            setContentView(R.layout.activity_main)
            checkAndRequestPermissions()
        }

        private fun checkAndRequestPermissions() {
            for (permission in REQUIRED_PERMISSION_LIST) {
                if (ContextCompat.checkSelfPermission(this, permission) != PackageManager.PERMISSION_GRANTED) {
                    missingPermissions.add(permission)
                }
            }

            if (missingPermissions.isEmpty()) {
                startSDKRegistration()
            } else {
                ActivityCompat.requestPermissions(this, missingPermissions.toTypedArray(), REQUEST_PERMISSION_CODE)
            }
        }

        private fun startSDKRegistration() {
            DJISDKManager.getInstance().registerApp(this, object : DJISDKManager.SDKManagerCallback {
                override fun onRegister(djiError: dji.common.error.DJIError) {
                    if (djiError === dji.common.error.DJIError.REGISTRATION_SUCCESS) {
                        Log.i(TAG, "SDK Registration Success")
                        DJISDKManager.getInstance().startConnectionToProduct()
                    } else {
                        Log.e(TAG, "SDK Registration Failed: " + djiError.description)
                    }
                }

                override fun onProductDisconnect() {
                    Log.i(TAG, "Product Disconnected")
                }

                override fun onProductConnect() {
                    Log.i(TAG, "Product Connected")
                }

                override fun onComponentChange(componentKey: dji.common.bus.DJIComponentKey, oldComponent: dji.common.bus.DJIComponent?, newComponent: dji.common.bus.DJIComponent?) {
                    Log.i(TAG, "Component Changed")
                }

                override fun onInitProcess(djiInitEvent: dji.common.bus.DJIInitEvent, initStatus: dji.common.bus.DJIInitStatus) {
                    Log.i(TAG, "Init Process: " + djiInitEvent.name)
                }

                override fun onDatabaseDownloadProgress(current: Long, total: Long) {
                    Log.i(TAG, "Database Download Progress: $current/$total")
                }
            })
        }

        override fun onRequestPermissionsResult(requestCode: Int, permissions: Array<String>, grantResults: IntArray) {
            super.onRequestPermissionsResult(requestCode, permissions, grantResults)
            if (requestCode == REQUEST_PERMISSION_CODE) {
                if (grantResults.isNotEmpty() && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                    startSDKRegistration()
                } else {
                    Log.e(TAG, "Permissions not granted")
                }
            }
        }
    }
    ```

### Challenges I encountered

**1. Dependency Issues:**

- **Error:** Missing dependencies caused build failures.
- **Solution:** Ensure all necessary dependencies (`dji-sdk`, `multidex`, `otto`) are added to the `build.gradle` file correctly.

**2. Permission Problems:**

- **Error:** Application crashing due to missing permissions.
- **Solution:** Add all required permissions in the `AndroidManifest.xml`.

**3. SDK Registration Failures:**

- **Error:** App not registering with DJI SDK.
- **Solution:** Verify the App Key placement in the manifest file and ensure an active internet connection during registration.

**4. Gradle Sync Issues:**

- **Error:** Errors during Gradle sync after adding dependencies.
- **Solution:** Check and correct syntax in `build.gradle`, ensuring compatibility with the project’s SDK version.

---


