---
title: flutter
date: 2024-07-30
tags: 
categories: 开发经验
---

## Dart基础

在线练习网址 https://dartpad.dev/

```
程序入口：main(){}

控制台输出：print('hello');

变量：var int

检查null或零：1或非null对象都为true ??为null时设置默认值

functions：fn(){}

异步编程：async函数返回一个Future awit运算用于等待future

_getIPAddress() async{
	final url = 'https://www.baidu.com/';
 	var request = await HttpRequest.request(url);
	String ip = json.decode(request.responseText)['origin'];
	print(ip);
}
```



## 声明式UI

声明式UI要修改UI，Widget本身不可变，只能触发重构修改UI

## Flutter入门基础

```
创建flutter
flutter create <projectname>

运行flutter
flutter run -d 'iPhone X'

导入Widget
import

写一个Hello World
//flutter
import 'package:flutter/material.dart';

void main(){
	runApp(
		Center(
			child: Text(
				'Hello, world!',
				textDirection: TextDirection.ltr,
			),
		),
	);
}


使用Widget并将其嵌套以形成widget树
//flutter
import 'package:flutter/material.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
	@override
	Widget build(BuildContext context){
	return MaterialApp(
			title: 'Werlcome to Flutter',
			home: Scaffold(
				appBar: AppBar{
					title:Text ('Welcome to Flutter'),
				},
				body: Center(
					child: Text('Hello, world!'),
				),
			),
		);
	}
}

创建可重用Widget

// Flutter
class CustomCard extends StatelessWidget {
	CustomCard ({@required this.index, @required
		this.onPress});
		
	final index;
	final Function onPress;
	
	@override
	Widget build(BuildContext context) {
		return Card(
			child: Column (
				children: <Widget>[
					Text ('Card $index'), 
					FlatButton (
						child: const Text( 'Press'), 
						onPressed: this.onPress,
					),
				],
			)
		);
	｝
｝
// Usage
CustomCard (
	index: index,
	onPress: () {
		print( 'Card $index');
	},
)

```



