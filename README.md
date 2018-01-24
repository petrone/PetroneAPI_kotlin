# PetroneAPI_kotlin
![Version](https://img.shields.io/badge/version-0.0.1-green.svg)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](http://opensource.org/licenses/MIT)

## Requirements
- Android Studio 3.0 Beta
- Android OS Jellybean and more

## AndroidManifest permission require
```xml
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
    <uses-permission android:name="android.permission.CHANGE_WIFI_STATE" />
    <uses-permission android:name="android.permission.BLUETOOTH" />
    <uses-permission android:name="android.permission.BLUETOOTH_ADMIN" />


    <application
        ...
        <service
            android:name="kr.co.byrobot.petroneapi.BLE.BLEService"
            android:exported="true"
            android:permission="kr.co.byrobot.petroneapi.BLE.BLEService.BLEPERMISSION">
            <intent-filter>
                <action android:name="kr.co.byrobot.petroneapi.BLE.BLEService" />
            </intent-filter>
        </service>
    </application>
```

## Usage
New Project
![alt tag](http://byrobot.co.kr/Petrone/github/kotlin_example01.png)
Check Include Kotlin support
    
![alt tag](http://byrobot.co.kr/Petrone/github/kotlin_example02.png)
![alt tag](http://byrobot.co.kr/Petrone/github/kotlin_example03.png)
![alt tag](http://byrobot.co.kr/Petrone/github/kotlin_example04.png)

File -> New -> Import Module
![alt tag](http://byrobot.co.kr/Petrone/github/kotlin_example05.png)
Select PetroneAPI_kotlin

![alt tag](http://byrobot.co.kr/Petrone/github/kotlin_example06.png)
add build.gradle anko_version for anko library.

Open Module Setting -> Dependencies -> (+) Module Dependency -> Select :PetroneAPI_kotlin
![alt tag](http://byrobot.co.kr/Petrone/github/kotlin_example07.png)

```kotlin
   // sample code MainActivity.kt

import android.Manifest
import android.support.v7.app.AppCompatActivity
import android.os.Bundle
import android.view.View
import android.widget.ArrayAdapter
import kotlinx.android.synthetic.main.activity_main.*
import kr.co.byrobot.petroneapi.*
import kr.co.byrobot.petroneapi.Data.*
import org.jetbrains.anko.doAsync
import org.jetbrains.anko.uiThread
import pub.devrel.easypermissions.EasyPermissions

class MainActivity : AppCompatActivity(), PetroneCallback {
    var petrone : Petrone? = null
    private var deviceList : List<PetroneDevice>? = null
    private var selectedDevice : Int = -1

    var items = ArrayList<String>()
    var adapter : ArrayAdapter<String>? = null // = ArrayAdapter(this, android.R.layout.simple_list_item_single_choice, items)

    var isScanning : Boolean = false

    private fun loadAndShowDevice()  {
        doAsync {
            Thread.sleep( 1000 )
            while ( isScanning) {
                deviceList = petrone!!.getDeviceList()

                if( deviceList != null ) {
                    items.clear()
                    deviceList!!.forEach { existDevice -> items.add(existDevice.name + " : " + existDevice.rssi) }

                    uiThread {
                        adapter!!.notifyDataSetChanged()
                    }

                    if (selectedDevice > -1 && selectedDevice < deviceList!!.count()) list_devices.setSelection(selectedDevice)
                }
                Thread.sleep( 1000 )
            }

            items.clear()
            uiThread {
                adapter!!.notifyDataSetChanged()
            }
        }
    }

    private val RC_PERMISSION = 9001

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        if( EasyPermissions.hasPermissions(applicationContext, Manifest.permission.BLUETOOTH)) {
            EasyPermissions.requestPermissions(this, "permission", RC_PERMISSION, Manifest.permission.BLUETOOTH);
        }

        petrone = Petrone(applicationContext)
        petrone!!.delegate = this


        button_bluetooth.setOnClickListener {
            isScanning = !isScanning
            if( isScanning ) {
                petrone!!.onScan()
                loadAndShowDevice()
                button_bluetooth.text = "STOP"
            } else {
                petrone!!.onStopScan()
                button_bluetooth.text = "SCAN"
            }

        }

        button_connect.setOnClickListener {
            if( selectedDevice != -1 ) {
                var target = deviceList!!.get(selectedDevice)
                petrone!!.onConnect(target.ssid)
            }
        }

        adapter = ArrayAdapter(this, android.R.layout.simple_list_item_single_choice, items)
        list_devices.setAdapter(adapter)
        list_devices.setOnItemClickListener {
            adapterView, view, i, l ->
            selectedDevice = i
            list_devices.setSelection(i)

        }

        button_takeoff.setOnClickListener {
            if( petrone!!.isReadyForStart() ) {
                petrone!!.takeOff()
            } else {
                petrone!!.landing()
            }
        }

        button_disconnect.setOnClickListener {
            petrone!!.onDisconnect()
        }

        button_emstop.setOnClickListener {
            petrone!!.emergencyStop()
        }

        layout_connected.visibility = View.INVISIBLE
    }


    override fun disconnected(reason:String ) {
        layout_connected.visibility = View.INVISIBLE
        button_connect.visibility = View.VISIBLE
        button_bluetooth.visibility = View.VISIBLE
    }

    override fun connectionComplete(complete:String ) {
        isScanning = false
        layout_connected.visibility = View.VISIBLE
        petrone!!.requestState()
        button_connect.visibility = View.INVISIBLE
        button_bluetooth.visibility = View.INVISIBLE
    }

    override fun recvFromPetroneResponse(response:Byte ) {}

    override fun recvFromPetroneResponse(status: PetroneStatus) {}

    override fun recvFromPetroneResponse(trim: PetroneTrim) {}

    override fun recvFromPetroneResponse(trimFlight: PetroneTrimFlight) {}

    override fun recvFromPetroneResponse(trimDrive: PetroneTrimDrive) {}

    override fun recvFromPetroneResponse(attitude: PetroneAttitude) {}

    override fun recvFromPetroneResponse(gyroBias: PetroneGyroBias) {}

    override fun recvFromPetroneResponse(flightCount: PetroneCountFlight) {}

    override fun recvFromPetroneResponse(driveCount: PetroneCountDrive) {}

    override fun recvFromPetroneResponse(motor: PetroneImuRawAndAngle) {}

    override fun recvFromPetroneResponse(motor: PetronePressure) {}

    override fun recvFromPetroneResponse(motor: PetroneImageFlow) {}

    override fun recvFromPetroneResponse(motor: PetroneMotor) {}

    override fun recvFromPetroneResponse(temperature: PetroneTemperature) {}

    override fun recvFromPetroneResponse(range: PetroneRange) {}
}

```

```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="kr.co.byrobot.kotlin_example.MainActivity">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical"
        android:weightSum="8">

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="horizontal"
            android:layout_weight="1">

            <Button
                android:id="@+id/button_connect"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:text="Connect"/>

            <Button
                android:id="@+id/button_bluetooth"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:text="Scan" />
        </LinearLayout>

        <ListView
            android:id="@+id/list_devices"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:choiceMode="singleChoice"
            android:layout_weight="6" />

        <LinearLayout
            android:id="@+id/layout_connected"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="horizontal"
            android:layout_weight="1">

            <Button
                android:id="@+id/button_takeoff"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="Take Off"
                android:layout_weight="1" />

            <Button
                android:id="@+id/button_emstop"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="Stop"
                android:layout_weight="1" />

            <Button
                android:id="@+id/button_disconnect"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="Disconnect"
                android:layout_weight="1"/>
        </LinearLayout>
    </LinearLayout>

</android.support.constraint.ConstraintLayout>

```
# Reference
  https://github.com/captain-miao/bleYan ( Android Java BLE )
