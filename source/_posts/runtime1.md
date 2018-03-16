---
title: Runtime(1)：消息传递
date: 2018-03-16 17:35:50
tags: [Runtime]
categories: [iOS开发]
toc: true
---

OC是动态语言，只有在运行时才会根据方法名去调用方法，称为给方法发送消息，是因为当调用`[demoObj setTest]`方法时，编译器会转化为`objc_msgSend(demoObj, @selector(setTest))`。如果携带参数，如`[demoObj setTest:str]`，会转化为`objc_msgSend(demoObj, @selector(setTest), str)`。

<!--more-->

## unrecognized selector sent to instance

`unrecognized selector sent to instance`是开发中经常遇到的异常，诸如点击事件没有实现，调用了只声明未实现的方法，向NSArray调用了NSMutableArray的方法等等。

我们定义类`DemoObject`，在`.h`中声明方法`- (void)setTest;`，`.m`中不写方法实现。然后调用：

```Objective-C
DemoObject *demoObj = [[DemoObject alloc] init];
[demoObj setTest];
```

程序会很听话的崩掉，并抛出异常：

```Objective-C
-[DemoObject setTest]: unrecognized selector sent to instance 0x60000000bda0
```

在程序崩溃之前，消息会经过下面几个方法转发：

```Objective-C
+ (BOOL)resolveInstanceMethod:(SEL)sel {
    NSLog(@"解析实例方法");
    return [super resolveInstanceMethod:sel];
}
+ (BOOL)resolveClassMethod:(SEL)sel {
    NSLog(@"解析类方法");
    return [super resolveClassMethod:sel];
}
- (id)forwardingTargetForSelector:(SEL)aSelector {
    NSLog(@"转发目标选择器");
    return [super forwardingTargetForSelector:aSelector];
}
- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector {
    NSLog(@"选择器方法签名");
    return [super methodSignatureForSelector:aSelector];
}
- (void)forwardInvocation:(NSInvocation *)anInvocation {
    NSLog(@"转发调用");
}
- (void)doesNotRecognizeSelector:(SEL)aSelector {
    NSLog(@"不识别选择器，崩溃");
}
```

消息会经过上述方法传递，最终未果才会崩溃。在上述方法中我们有三次机会操作消息传递，防止崩溃。

1. 动态添加方法：`+ (BOOL)resolveInstanceMethod:(SEL)sel`或`+ (BOOL)resolveClassMethod:(SEL)sel`
2. 方法重定向：`- (id)forwardingTargetForSelector:(SEL)aSelector`
3. 消息转发：`- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector`和`- (void)forwardInvocation:(NSInvocation *)anInvocation`

## 动态添加方法

### 实例方法

在类DemoObject中引入runtime：`#import <objc/runtime.h>`，使用`class_addMethod`动态添加方法实现。

```Objective-C
void resolveTest(id self, SEL _cmd) {
    NSLog(@"动态添加方法调用 = %@", NSStringFromSelector(_cmd));
}
+ (BOOL)resolveInstanceMethod:(SEL)sel {
    if (sel == @selector(setTest)) {
        class_addMethod([self class], sel, (IMP)resolveTest, "v@:");
        return YES;
    }
    return [super resolveInstanceMethod:sel];
}
```

方法`class_addMethod`：

```Objective-C
OBJC_EXPORT BOOL
class_addMethod(Class _Nullable cls, SEL _Nonnull name, IMP _Nonnull imp, 
                const char * _Nullable types) 
    OBJC_AVAILABLE(10.5, 2.0, 9.0, 1.0, 2.0);
```

参数意义如下：

- cls：消息接收者
- name：SEL方法名
- imp：要动态添加方法的IMP指针
- types：参数和返回值的符号字符串，[查看格式文档](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtTypeEncodings.html)

### 类方法

解析类方法如下：

```Objective-C
+ (BOOL)resolveClassMethod:(SEL)sel {
    if (sel == @selector(setTestClass)) {
        class_addMethod(object_getClass(self), sel, (IMP)resolveTest, "v@:");
        return YES;
    }
    return [super resolveClassMethod:sel];
}
```

这里的区别在于cls：消息接收者，解析实例方法使用的`[self class]`，解析类方法使用的`object_getClass(self)`。

当self为实例对象时，`[self class]`与`object_getClass(self)`等价，因为前者会调用后者。`object_getClass([self class])`得到元类。

当self为类对象时，`[self class]`返回值为自身，还是self，所以上面解析实例方法将`[self class]`换成`self`也可以。`object_getClass(self)`与`object_getClass([self class])`等价。

## 方法重定向

### 实例方法

重写`- (id)forwardingTargetForSelector:(SEL)aSelector`方法，可以将消息的接收者替换成其他对象。

新创建一个类`DemoNewObject`，将类`DemoObject`未实现的方法`- (void)setTest`，在`.m`中实现，无需在`.h`中暴露方法：

```Objective-C
- (void)setTest {
    NSLog(@"实例方法重定向");
}
```

方法重定向操作：

```Objective-C
- (id)forwardingTargetForSelector:(SEL)aSelector {
    if (aSelector == @selector(setTest)) {
        DemoNewObject *obj = [[DemoNewObject alloc] init];
        return obj;
    }
    return [super forwardingTargetForSelector:aSelector];
}
```

### 类方法

类方法重定向需要重写`+ (id)forwardingTargetForSelector:(SEL)aSelector`方法，注意是`+`开头的类方法。

同样在新类`DemoNewObject`中实现类方法：

```Objective-C
+ (void)setTestClass {
    NSLog(@"类方法重定向");
}
```

重写重定向方法：

```Objective-C
+ (id)forwardingTargetForSelector:(SEL)aSelector {
    if (aSelector == @selector(setTestClass)) {
        return NSClassFromString(@"DemoNewObject");
    }
    return [super forwardingTargetForSelector:aSelector];
}
```

方法重定向就是将当前类未实现的方法，重定向到一个实现该方法的新类中，调用新类中的方法实现。实例方法中返回实例对象，类方法中返回类对象。

## 消息转发

消息转发是通过方法`- (void)forwardInvocation:(NSInvocation *)anInvocation`实现的，它可以将不能处理的消息转发给其他对象处理，参数`anInvocation`是通过方法`- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector`产生的。

所以需要重写两个方法，`- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector`和`- (void)forwardInvocation:(NSInvocation *)anInvocation`：

```Objective-C
- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector {
    NSMethodSignature *signature = [super methodSignatureForSelector:aSelector];
    if (!signature) {
        if ([DemoNewObject instancesRespondToSelector:aSelector]) {
            signature = [DemoNewObject instanceMethodSignatureForSelector:aSelector];
        }
    }
    return signature;
}
- (void)forwardInvocation:(NSInvocation *)anInvocation {
    if ([DemoNewObject instancesRespondToSelector:anInvocation.selector]) {
        [anInvocation invokeWithTarget:[[DemoNewObject alloc] init]];
    } else {
        [super forwardInvocation:anInvocation];
    }
}
```

**参考资料**

[Objective-C Runtime](http://yulingtianxia.com/blog/2014/11/05/objective-c-runtime/)

[iOS中的unrecognized selector sent to instance..](https://www.jianshu.com/p/60c251712df7)

