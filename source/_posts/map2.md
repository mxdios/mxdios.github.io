---
title: 火星坐标转GPS坐标
date: 2020-12-06 14:31:49
tags: [地图]
categories: [iOS开发]
toc: true
---

火星坐标（GCJ02）转GPS坐标（WGS84）没有标准的方法，网上倒是有很多第三方的。最近在做一款产品，要求非常精确的将火星坐标转换成GPS坐标。

<!--more-->

我试过很多种方法，[JZLocationConverter-Swift](https://github.com/JackZhouCn/JZLocationConverter-Swift) 算误差比较小了，有几米的误差。我找到另一种方法：二分法取中心点，逐步逼近。几乎可以算是零误差了，但有个问题。

## 算法详解

先说这个转换方法的逻辑，步骤如下：

1. 火星坐标 A 转换GPS坐标 A1，对 A 的经纬度（lng，lat）分别 + - 0.5，得到4个值:

   ```
   最小经度 minLng: lng - 0.5
   最大经度 maxLng: lng + 0.5
   最小纬度 minLat: lat - 0.5
   最大纬度 maxLat: lat + 0.5
   ```

2. 可以组成4个火星坐标：**注：方位只在东北半球适用，火星坐标也只中国用 : ( **

   ```
   左下角：a (minLng, minLat)
   右下角：b (maxLng, minLat)
   左上角：c (minLng, maxLat)
   右上角：d (maxLng, maxLat)
   ```

3. 这4个坐标点得到一个正方形区域 M，A 是中心点，它们都属于火星坐标系

4. **重点：将M区域 a，b，c，d 4个坐标看作是GPS坐标！获取中心点坐标 P （P是GPS坐标，A是火星坐标，数值相等，位置不同）**

5. 计算 P 与 A 的经纬度差，坐标系不同不能直接比较。将 P 转为火星坐标P1（**有官方转换方法**）

6. 计算 P1 与 A 的经纬度相差之和 ( | P1lng - Alng | + | P1lat - Alat | ) < 0.00001，则认为P1和A重叠，那 P 就是 A 对应的GPS坐标。

7. 如果 > 0.00001，二分法缩小区域，将a，b，c，d 4点转为火星坐标 a1，b1，c1，d1，以P1为准，计算A在P1的什么方位。看A在 P1-a1，P1-b1，P1-c1，P1-d1，哪个之间。

8. 用二分法排除以外区域，比如：A在P1的左下角，就是A在P1-a1之间，得到新的区域：

   ```
   左下角：a（minLng, minLat)
   右下角：b ((minLng + maxLng) / 2, minLat)
   左上角：c (minLng, (minLat + maxLat) / 2)
   右上角：d ((minLng + maxLng) / 2, (minLat + maxLat) / 2)
   ```

9. 6 - 8 步骤循环30次，一般情况下能得到差距 < 0.00001 的GPS坐标，即时达不到标准，结果也很接近。

这个转换一句话解释：**用已知的GPS坐标，逐步逼近要转换的火星坐标**

## 算法漏洞

在测试过程中，有一个坐标点出现了问题：上海虹桥机场2号航站楼 (latitude : 31.194248175984768  longitude : 121.32863948433301)

下面是算法输出的经纬度相差之和：

```
1. 左上，0.49360417451567073
2. 右下，0.24358081099306617
3. 右下，0.11855877737957243
4. 右下，0.05603884882789245
5. 右下，0.024777349920977798
6. 右下，0.00914639780950921
7. 右下，0.0025836297689565413
8. 左下，0.0013319084599991982
9. 右上，0.001330858958493053
10. 右上，0.001604743230569028
```

坐标点在循环第10次，四角坐标点从GPS转为火星坐标系时，逃出了限定区域，这时已没法继续二分法逼近了，只能返回差距`0.001604743230569028`的坐标转换。

问题成因：GPS坐标在转换为火星坐标系时，四个点并非同步平移，正方形区域的GPS坐标，转为火星坐标时，就成了不规则的四边形，有时会把坐标点闪到外面。

正在想办法解决这个问题……

## 代码实现

```swift
/// 火星坐标 -> GPS
func transformFromGCJToWGS(p:CLLocationCoordinate2D) -> CLLocationCoordinate2D {
    let threshold = 0.00001;
    var minLat = p.latitude - 0.5;
    var maxLat = p.latitude + 0.5;
    var minLng = p.longitude - 0.5;
    var maxLng = p.longitude + 0.5;
    var delta = 1.0;
    var maxIteration = 30;
    while(true) {
        let leftBottom = AMapCoordinateConvert(CLLocationCoordinate2D(latitude: minLat, longitude: minLng), .GPS)
        let rightBottom = AMapCoordinateConvert(CLLocationCoordinate2D(latitude: minLat, longitude: maxLng), .GPS)
        let leftUp = AMapCoordinateConvert(CLLocationCoordinate2D(latitude: maxLat, longitude: minLng), .GPS)
        let rightUp = AMapCoordinateConvert(CLLocationCoordinate2D(latitude: maxLat, longitude: maxLng), .GPS)
        let midPoint = AMapCoordinateConvert(CLLocationCoordinate2D(latitude : ((minLat + maxLat) / 2),longitude : ((minLng + maxLng) / 2)), .GPS)
        delta = fabs(midPoint.latitude - p.latitude) + fabs(midPoint.longitude - p.longitude);
        maxIteration = maxIteration - 1
        if(maxIteration <= 0 || delta <= threshold) {
            return CLLocationCoordinate2D(latitude: (minLat + maxLat) / 2, longitude: (minLng + maxLng) / 2);
        }
        if(isContains(point: p, p1: leftBottom, p2: midPoint)) {
            maxLat = (minLat + maxLat) / 2;
            maxLng = (minLng + maxLng) / 2;
        } else if(isContains(point: p, p1: rightBottom, p2: midPoint)) {
            maxLat = (minLat + maxLat) / 2;
            minLng = (minLng + maxLng) / 2;
        } else if(isContains(point: p, p1: leftUp, p2: midPoint)) {
            minLat = (minLat + maxLat) / 2;
            maxLng = (minLng + maxLng) / 2;
        } else if(isContains(point: p, p1: rightUp, p2: midPoint)){
            minLat = (minLat + maxLat) / 2;
            minLng = (minLng + maxLng) / 2;
        } else {
            return CLLocationCoordinate2D(latitude: (minLat + maxLat) / 2, longitude: (minLng + maxLng) / 2);
        }
    }
}
func isContains(point:CLLocationCoordinate2D , p1:CLLocationCoordinate2D, p2:CLLocationCoordinate2D)->Bool {
    return (point.latitude >= min(p1.latitude, p2.latitude) &&
                point.latitude <= max(p1.latitude, p2.latitude)) &&
        (point.longitude >= min(p1.longitude,p2.longitude) &&
            point.longitude <= max(p1.longitude, p2.longitude));
}
```



参考链接：[iOS地图开发2-坐标系的转换(swift)](https://cloud.tencent.com/developer/article/1524369) 

