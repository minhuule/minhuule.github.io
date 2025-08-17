---
title: AOSP 14 - Add new Vendor Property
date: 2024-12-23 13:08:00 +0800
categories: [Android Automotive]
tags: [property, aosp, raspberry pi 4]
image: /assets/img/posts/vendor_property_aosp14/cover.png
---
<div style="text-align: justify">
Properties is a key-value system that allows different parts of the Android system to communicate and share configuration data. For example, in my case it is for service to talk with HAL (Hardware Abstraction Layer) to control hardware. A property might hold information about the device’s hardware, network settings, or boot parameters. Here is how to add a new property to AOSP, then control it by built-in application or ADB command.
</div>

## Add a new vendor property
<div style="text-align: justify">
There are many types of properties in AOSP like system property, vendor property, product property, … But to execute basic tasks, vendor property is the common and simple property to implement. Here are some files you need to update to add a new vendor property:
</div>

> **hardware/interfaces/automotive/vehicle/aidl/impl/utils/test_vendor_properties/android/hardware/automotive/vehicle/TestVendorProperty.aidl**
```
enum TestVendorProperty {
    ...
    /**
    * VENDOR_DUMMY_VALUE, used for custom property function check
    *
    * VehiclePropertyGroup.VENDOR | VehicleArea.GLOBAL | VehiclePropertyGroup.INT32
    */
    VENDOR_DUMMY_VALUE = 0x0111 + 0x20000000 + 0x01000000 + 0x00400000,
    ...
}
```
* `0x0111`: This can be any number, just make sure that your property is unique in the whole system.
* `0x20000000`: Property Group: VENDOR. Property groups are defined in `hardware/interfaces/automotive/vehicle/aidl_property/android/hardware/automotive/vehicle/VehiclePropertyGroup.aidl`
* `0x01000000`: Property Area: GLOBAL, which means it can be accessed by the whole system. Property areas are defined in `hardware/interfaces/automotive/vehicle/aidl_property/android/hardware/automotive/vehicle/VehicleArea.aidl`
* `0x00400000`: property data type: INT32. Property data types are defined in `hardware/interfaces/automotive/vehicle/aidl_property/android/hardware/automotive/vehicle/VehiclePropertyType.aidl`

> **hardware/interfaces/automotive/vehicle/aidl/impl/default_config/config/VendorClusterTestProperties.json**
```
{
    "properties": [
        ...
        {
            "property": "TestVendorProperty::VENDOR_DUMMY_VALUE",
            "defaultValue": {
                "int32Values": [
                    10
                ]
            },
            "access": "VehiclePropertyAccess::READ_WRITE",
            "changeMode": "VehiclePropertyChangeMode::ON_CHANGE"
        },
        ...
    ]
}
```
* `int32Values`: data type (int32 is also used to boolean type). Declare initial value of the property, in my example it is 10.
* `READ_WRITE`: access type, which means the property can be both read and written. Other access types are READ, WRITE or NONE. They are defined in `hardware/interfaces/automotive/vehicle/aidl/android/hardware/automotive/vehicle/VehiclePropertyAccess.aidl`
* `ON_CHANGE`: change mode. Other change modes are STATIC and CONTINUOUS. They are defined in `hardware/interfaces/automotive/vehicle/aidl/android/hardware/automotive/vehicle/VehiclePropertyChangeMode.aidl`

> **packages/services/Car/car-lib/src/android/car/VehiclePropertyIds.java**
```
public final class VehiclePropertyIds {
...
    /**
    * VENDOR_DUMMY_VALUE, used for custom property function check
    * 
    * <p>Property Config:
    * <ul>
    * <li>{@link android.car.hardware.CarPropertyConfig#VEHICLE_PROPERTY_ACCESS_READ_WRITE}
    * <li>{@link VehicleAreaType#VEHICLE_AREA_TYPE_GLOBAL}
    * <li>{@link android.car.hardware.CarPropertyConfig#VEHICLE_PROPERTY_CHANGE_MODE_ONCHANGE}
    * <li>{@code Object[]} property type
    * </ul>
    *
    * @hide
    */
    public static final int VENDOR_DUMMY_VALUE = 557842705;
...
}
```
* This file is used to expose the property to Java service/framework/application layer, so that it can be called by name, not prop ID value.

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> Note: remember to add `@hide` in comment section to prevent build error.
{: .prompt-tip }
<!-- markdownlint-restore -->

> **packages/services/Car/car-lib/api/lint-baseline.txt**
```
UnflaggedApi: android.car.VehiclePropertyIds#VENDOR_DUMMY_VALUE:
    New API must be flagged with @FlaggedApi: field android.car.VehiclePropertyIds.VENDOR_DUMMY_VALUE
```
* Add the entry to lint-baseline.txt suppresses the Lint warning for that specific API, allowing the build to proceed while you address or document the new property correctly.

That’s all, now build and flash your image, then boot your device.

## Control Property By Kichensink
<div style="text-align: justify">
Kitchensink app in AOSP is to serve as a test application that showcases and exercises a wide range of Android framework features and APIs. It includes various components, activities, and UI elements to help developers test and demonstrate the functionality of different Android features in a single, unified app.
</div>

![](/assets/img/posts/vendor_property_aosp14/kitchensink1.png)

One of its function is to interact with properties.

Scroll down and click on Property Test

![](/assets/img/posts/vendor_property_aosp14/kitchensink2.png)

![](/assets/img/posts/vendor_property_aosp14/kitchensink3.png)

## Control Property By ADB Command

To write value
```
adb shell dumpsys car_service set-property-value <prop-value> 0x01000000 <value>
```

To read value
```
adb shell dumpsys car_service get-property-value <prop-value>
```

For example the property that I added above has `prop-value = 557842705` with initial value is `10`
![](/assets/img/posts/vendor_property_aosp14/adb.png)

<div style="text-align: justify">
Okay that’s all. I hope this tutorial has provided you with a clear understanding of how to add a new property to AOSP and made the process easier for you to follow. Wishing you success!
</div>
