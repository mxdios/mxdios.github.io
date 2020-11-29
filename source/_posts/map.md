---
title: iOS中的地图与坐标
date: 2020-11-28 14:31:49
tags: [地图]
categories: [iOS开发]
toc: true
---

## 概念

- 地图：系统地图（MKMapView），百度地图（BMKMapView），高德地图（MAMapView）
- 坐标系：GPS坐标系（WGS84），火星坐标系（GCJ02），百度坐标系（BD09）
- 定位：系统定位（CLLocationManager），百度定位SDK（BMKLocationManager），高德定位SDK（AMapLocationManager）

<!--more-->

## 地图

苹果系统地图在海外全球通用，为GPS坐标系。在国内使用的是高德数据，所以返回的经纬度是火星坐标系的。国内测试，超出国境线就无法获取经纬度数据。

百度地图的坐标系是自己的BD09，无法切换为其他坐标系。

高德地图坐标系是火星坐标系，无法切换为其他坐标系。

**地图的坐标系无法更改！！！**

## 坐标系

GPS坐标系是全球坐标系，国际标准。火星坐标系是中国的标准，是在GPS坐标的基础上做了一层加密。法律规定：在中国，地图产品不允许直接使用GPS坐标，至少使用一层加密的火星坐标系（GCJ02）。百度坐标系（BD09）则是在GCJ02的基础上又做了一层转换。

GPS坐标转为火星坐标或百度坐标，以及火星坐标和百度坐标相互转换，都有官方提供的方法：

[高德提供的坐标转换](https://lbs.amap.com/api/ios-sdk/guide/computing-equipment/amap-calculate-tool/?sug_index=3)

[百度提供的坐标转换](http://lbs.baidu.com/index.php?title=iossdk/guide/tool/coordinate)

但是！火星坐标和百度坐标不允许转换GPS坐标，至少没有官方提供转换方法。

网上有很多开源的转换工具，亲测了几种：

[JZLocationConverter-Swift](https://github.com/JackZhouCn/JZLocationConverter-Swift) ： 在转换WGS84时会有误差，官方有说明

[iOS地图开发2-坐标系的转换(swift)](https://cloud.tencent.com/developer/article/1524369) ： 里面的一些参数可做优化，亲测 GCJ02 -> WGS84 非常精准。

虽然法律规定国内不能使用GPS坐标系，这仅限于地图产品。**定位是可以直接获取GPS坐标的！**

如果定位数据要放到地图中使用，则要保证统一坐标系，不然会出现偏差。

## 定位

### 系统定位

系统定位返回的经纬度为GPS坐标。

```swift
let locationM = CLLocationManager()
locationM.delegate = self
locationM.requestWhenInUseAuthorization() //使用app时定位
locationM.startUpdatingLocation() //开始定位

/// 定位的代理方法
func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
    if locations.last != nil {
        print("经纬度 = \(locations.last!.coordinate)")
        manager.stopUpdatingLocation() //停止定位
    }
}
```

### 百度定位SDK

百度定位SDK获取的经纬度默认为火星坐标系（GCJ02），百度地图中是百度坐标系（BD09），如果要定位的坐标放到地图里用，则需要统一坐标系，指定坐标系为`.BMK09LL`。

也可以获取其他坐标系中的定位坐标：

```swift
let locationM = BMKLocationManager()
locationM.delegate = self
locationM.coordinateType = .BMK09LL //指定坐标系
locationM.startUpdatingLocation()

func bmkLocationManager(_ manager: BMKLocationManager, didUpdate location: BMKLocation?, orError error: Error?) {
  if let coordinate = location?.location?.coordinate {
    print("经纬度 = \(coordinate)")
  }
}

```

### 高德定位SDK

高德的定位和地图SDK只使用了一种坐标系：火星坐标系（GCJ02）

```swift
let locationM = AMapLocationManager()
locationM.delegate = self
locationM.locatingWithReGeocode = true //返回逆地理信息
locationM.startUpdatingLocation() //开始定位

/// 代理方法
func amapLocationManager(_ manager: AMapLocationManager!, didUpdate location: CLLocation!, reGeocode: AMapLocationReGeocode!) {
  print("经纬度 = \(location.coordinate)")
}
```

参考资料：

[iOS 火星坐标/地球坐标/百度坐标整理及解决方案汇总？如何转火星坐标？](https://www.jianshu.com/p/3cd701299cef)