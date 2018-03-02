---
title: iOS开发笔记三
date: 2018-03-02 14:31:49
tags: [iOS]
categories: [iOS开发]
toc: true
---


## NSInterger to NSData

NSInterger to NSData，数字转化为Data，打印出来以十六进制形式展示

<!--more-->

```Objective-C
- (NSData *)getDataWithInt:(NSInteger)interger {
    Byte b1=interger & 0xff;
    Byte b2=(interger>>8) & 0xff;
    Byte b3=(interger>>16) & 0xff;
    Byte b4=(interger>>24) & 0xff;
    if (interger <= 255) {
        Byte byte[] = {b1};
        return [NSData dataWithBytes:byte length:sizeof(byte)];
    } else if (interger <= 65535) {
        Byte byte[] = {b2,b1};
        return [NSData dataWithBytes:byte length:sizeof(byte)];
    } else if (interger <= 16777215) {
        Byte byte[] = {b3,b2,b1};
        return [NSData dataWithBytes:byte length:sizeof(byte)];
    } else {
        Byte byte[] = {b4,b3,b2,b1};
        return [NSData dataWithBytes:byte length:sizeof(byte)];
    }
}
```

## SHA1加密

```Objective-C
- (NSString *)SHA1Encrypt:(NSString *)string{
    const char *cstr = [string UTF8String];
    NSData *data = [NSData dataWithBytes:cstr length:string.length];
    uint8_t digest[CC_SHA1_DIGEST_LENGTH];
    CC_SHA1(data.bytes, (CC_LONG)data.length, digest);
    NSMutableString *output = [NSMutableString stringWithCapacity:CC_SHA1_DIGEST_LENGTH *2];
    for (int i = 0; i<CC_SHA1_DIGEST_LENGTH; i++) {
        [output appendFormat:@"%02x",digest[i]];
    }
    return output;
}
```

## iPhone屏幕单位

ios开发中设置的size为pt，pt是绝对长度，等于1/72英寸，等于1/72*25.4毫米。

px是像素，像素点的密度代表着屏幕清晰度。这就是开发中@1x、@2x、@3x的区别。

iPhone 3GS是@1x像素级，分辨率为480px * 320px，iPhone4是@2x像素级，分辨率为960px * 640px。这两者尺寸是一样的，所以pt是一样的，在同样大的范围内，iPhone 4的像素点比iPhone 3GS的多一倍。

## QR码的一些知识点

QR码有40个版本，版本1是21 x 21个小方块组成，版本2是25 x 25个小方块，每增加1版本，二维码长宽各增加4个方块。所以最高版本40，方块数为177 * 177。计算公式是：(V-1) * 4 + 21

使用CIFilter生成QR码

```
//生码Objective-C
- (UIImage *)markCode:(NSString *)code {
    CIFilter *filter = [CIFilter filterWithName:@"CIQRCodeGenerator"];
    [filter setDefaults];
    NSData *data = [code dataUsingEncoding:NSUTF8StringEncoding];
    [filter setValue:data forKey:@"inputMessage"];
    [filter setValue:@"L" forKey:@"inputCorrectionLevel"]; //二维码的纠错级别 L < H < Q < M
    
    CIColor *color1 = [CIColor colorWithCGColor:[UIColor blackColor].CGColor];//二维码颜色
    CIColor *color2 = [CIColor colorWithCGColor:[UIColor whiteColor].CGColor];//背景色
    NSDictionary *parameters = [NSDictionary dictionaryWithObjectsAndKeys: filter.outputImage ,@"inputImage", color1,@"inputColor0", color2,@"inputColor1",nil];
    CIFilter *newFilter = [CIFilter filterWithName:@"CIFalseColor" withInputParameters:parameters];
    CIImage *outPutImage = [newFilter outputImage];
    int version = (int)((outPutImage.extent.size.width - 21) / 4.0 + 1); //获取该二维码的版本号
    return [self createNonInterpolatedUIImageFormCIImage:outPutImage withSize:600];
}
//让二维码变的清楚
- (UIImage *)createNonInterpolatedUIImageFormCIImage:(CIImage *)image withSize:(CGFloat) size {
    CGRect extent = CGRectIntegral(image.extent);
    CGFloat scale = MIN(size/CGRectGetWidth(extent), size/CGRectGetHeight(extent));
    size_t width = CGRectGetWidth(extent) * scale;
    size_t height = CGRectGetHeight(extent) * scale;
    CGColorSpaceRef cs = CGColorSpaceCreateDeviceRGB();
    CGContextRef bitmapRef = CGBitmapContextCreate(nil, width, height, 8, 0, cs, (CGBitmapInfo)kCGImageAlphaPremultipliedLast);
    CIContext *context = [CIContext contextWithOptions:nil];
    CGImageRef bitmapImage = [context createCGImage:image fromRect:extent];
    
    CGContextSetInterpolationQuality(bitmapRef, kCGInterpolationNone);
    CGContextScaleCTM(bitmapRef, scale, scale);
    CGContextDrawImage(bitmapRef, extent, bitmapImage);
    
    CGImageRef scaledImage = CGBitmapContextCreateImage(bitmapRef);
    CGContextRelease(bitmapRef); CGImageRelease(bitmapImage);
    
    UIImage *outputImage = [UIImage imageWithCGImage:scaledImage];
    return outputImage;
}
```


