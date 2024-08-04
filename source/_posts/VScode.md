---
    title: VScode
    date: 2021-08-29 23:02:22 
---

# VScode


## Mac

vscode开发flutter

安装插件：Flutter Snippets：实现自动补全

- fcol：创建一个Column Widget
- fcont：创建一个Container Widget
- frow：创建一个Row Widget
- ftxt：创建一个Text Widget



- 快速创建Widget：在dart文件中输入stf或stl出现提示后按回车键
- 快速修复：command + .
- 自动生成构造函数：选中final参数，快捷键command + .
- 添加父组件、变成子组件、删除子组件：command + .
- 重新打开 关闭的编辑页面：Command + shift +T
- 通过匹配文本打开文件：command + T
- 代码格式化：shift+option+F
- 打开console：command+3
- 查看源码：长按command 点击方法名类名
- 查看类的子类：选中要查看的类，然后command+F12
- 后退：当跟踪代码的时候经常跳到其它类，后退：ctrl + -
- 导入类的快捷键：把光标放在要导入的类上，然后按command + -
- 全局搜索：command + shift + F
- 当前行代码交换位置：option + 上/下
- 快速复制当前行：option + shift + 下



cmd：

flutter run

flutter run -d 设备id

q:终止运行

c:清除屏幕

r:热重载

command+k：清除终端输出的信息

flutter clean：清理缓存，可用于更改代码后运行有些异常的处理方式

flutter --version：查看flutter版本


## 问题解决

### vscode编译 不允许使用与号(&)

不允许使用与号(&)。& 运算符是为将来使用而保留的；请用双引号将与号引起来("&")，以将其作为字符串的一部分传递。

原因：vscode终端（terminal）默认是powershell，vscode在powershell中输入指令时会出现某种我无法理解的问题

解决方法：将默认终端设置为cmd

- 点击Select Default Profile

- 选择Command Prompt