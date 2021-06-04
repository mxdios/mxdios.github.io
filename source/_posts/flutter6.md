---
title: Flutter笔记6-json与model
date: 2021-06-04 14:30:50
tags: [Flutter]
categories: [Flutter]
toc: true

---

开发中json数据是最常见的，在具体使用时，通常会将json转为model。这样能保证类型安全，编译器也能自动补全，提示编译异常，用起来方便又安全。在flutter中如何进行json和model的相互转换呢？

<!--more-->

## 简单的json

```json
var people = {"name": "mark","age": 18};
```

定一个model类，并实现转换方法：

```dart
class PeopleModel {
  String name;
  int age;
  PeopleModel(this.name, this.age);
  /// json -> model
  PeopleModel.fromJson(Map<String, dynamic> json) {
    name = json['name'];
    age = json['age'];
  }
  /// model -> json
  Map<String, dynamic> toJson() {
    final Map<String, dynamic> data = new Map<String, dynamic>();
    dict['name'] = this.name;
    dict['age'] = this.age;
    return dict;
  }
}
```

使用：

```dart
var people = {"name": "mark","age": 18};
var peopleModel = PeopleModel.fromJson(people);
var peopleJson = peopleModel.toJson();
```

## 嵌套json

在开发中嵌套json非常常见，自己写model类、写转换方法不仅麻烦还容易写错。使用[json_serializable](https://pub.dartlang.org/packages/json_serializable)，可以自动生成json序列化代码，非常方便。

嵌套json：

```json
var people = {"name": "mark","age": 18, "school": {"name": "北大", "science": "计算机", "classes": 3}};
```

在`pubspec.yaml`添加依赖，执行`flutter packages get`（vscode文件保存会自动执行）：

```yaml
environment:
  sdk: ">=2.12.0 <3.0.0"
  
dependencies:
  flutter:
    sdk: flutter
  json_annotation: ^4.0.1

dev_dependencies:
  flutter_test:
    sdk: flutter
  json_serializable: ^4.1.3 # json序列化
  build_runner: ^2.0.4 # 自动生成代码
```

我使用的Flutter 2.2.1，Dart 2.13.1，`json_annotation`，4.0以上版本，需要指定sdk ">=2.12.0 <3.0.0"。不然会报错，详情见[issues/832](https://github.com/google/json_serializable.dart/issues/823)

创建model类：

```dart
import 'package:json_annotation/json_annotation.dart';
part 'people_model.g.dart'; // 创建类的文件名

@JsonSerializable()
class PeopleModel { // 类名
  String name;
  int age;
  SchoolMode school;

  PeopleModel(this.name, this.age, this.school);

  factory PeopleModel.fromJson(Map<String, dynamic> json) => _$PeopleModelFromJson(json);
  Map<String, dynamic> toJson() => _$PeopleModelToJson(this);
}


@JsonSerializable()
class SchoolMode {
  String name;
  String science;
  int classes;
  
  SchoolMode(this.name, this.science, this.classes);

  factory SchoolMode.fromJson(Map<String, dynamic> json) => _$SchoolModeFromJson(json);
  Map<String, dynamic> toJson() => _$SchoolModeToJson(this);
}
```

执行命令：`flutter pub run build_runner build --delete-conflicting-outputs`，会生成`people_model.g.dart`文件，这里是model的序列化代码：

```dart
// GENERATED CODE - DO NOT MODIFY BY HAND

part of 'people_model.dart';

// **************************************************************************
// JsonSerializableGenerator
// **************************************************************************

PeopleModel _$PeopleModelFromJson(Map<String, dynamic> json) {
  return PeopleModel(
    json['name'] as String,
    json['age'] as int,
    SchoolMode.fromJson(json['school'] as Map<String, dynamic>),
  );
}

Map<String, dynamic> _$PeopleModelToJson(PeopleModel instance) =>
    <String, dynamic>{
      'name': instance.name,
      'age': instance.age,
      'school': instance.school,
    };

SchoolMode _$SchoolModeFromJson(Map<String, dynamic> json) {
  return SchoolMode(
    json['name'] as String,
    json['science'] as String,
    json['classes'] as int,
  );
}

Map<String, dynamic> _$SchoolModeToJson(SchoolMode instance) =>
    <String, dynamic>{
      'name': instance.name,
      'science': instance.science,
      'classes': instance.classes,
    };

```

使用：

```dart
var people = {"name": "mark","age": 18, "school": {"name": "北大", "science": "计算机", "classes": 3}};
var peopleModel = PeopleModel.fromJson(people);
print(peopleModel.school.name); //输出 北大
var peopleJson = peopleModel.toJson(); //输出 {name: mark, age: 18, school: Instance of 'SchoolMode'}
```

**注：嵌套json的model在toJson()时，转换不彻底，里面的model并没有转换为json。**

Dart2.0之后引入了`空安全`，在定义模型类时，定义空类型需要`String? name;`，这点和swift类似。

