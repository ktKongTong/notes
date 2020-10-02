# Android项目文件结构简介

<a name="95ed849c"></a>
#### 最外层项目目录结构
![image.png](https://cdn.nlark.com/yuque/0/2020/png/1419739/1599701045320-a9d16282-0711-4fb4-aa2e-4171108d50c3.png#align=left&display=inline&height=215&margin=%5Bobject%20Object%5D&name=image.png&originHeight=429&originWidth=437&size=21874&status=done&style=none&width=218.5)<br />这是新建一个项目的默认目录结构

- .gradle与.idea

Android Studio自动生成的文件，无需修改

- app

放置项目代码、资源等内容的目录，主要的开发工作大多都会在该目录下进行

- gradle

该目录下包含gradle wrapper的配置文件，使用gradle warpper方式不需要提前下载gradle，而是根据本地的缓存情况决定是否需要联网下载gradle。AndroidStudio默认未启用gradle wrapper的方式。<br />在Android Studio导航栏->File->Settings->Build,Execution,Deployment->Gradle，进行配置更改

- .gitignore

用于将指定目录或文件排除在版本控制之外

- build.gradle

项目的全局构建脚本，通常这个文件内容不需要修改，但是因为国内的网络原因，可能需要修改镜像源

- gradle.properties

全局的gradle配置文件，在这里的配置的属性会影响项目中所有的gradle编译脚本

- gradlew与gradlew.bat

用于命令行中执行gradle命令的，gradlew用于Linux/Mac，gradlew.bat是在windows系统中使用的

- local.properties

这个文件用于指定本机中的Android SDK路径，通常内容自动生成，若本机AndroidSDK位置变化，应修改

- settings.gradle

用于指定项目中所有引入的模块。通常也是自动引入，需要修改的场景较少<br />

<a name="o88wQ"></a>
#### app目录下的文件结构
![image.png](https://cdn.nlark.com/yuque/0/2020/png/1419739/1599713160026-8043ba41-d937-4e1e-ad8f-8b00a90b8114.png#align=left&display=inline&height=194&margin=%5Bobject%20Object%5D&name=image.png&originHeight=387&originWidth=418&size=15582&status=done&style=none&width=209)

- build

编译时自动产生的文件

- libs

项目中使用的第三方jar包，就需要把这些jar包都放在libs目录下，会自动加入到构建路径

- src/androidTest

用于编写AndroidTest测试用例，对项目进行自动化测试

- src/main/java

存放主要的项目代码

- src/main/res

用于存放资源文件，如布局，图标等

- src/main/AndroidManifest.xml

Android项目的配置文件，程序中所定义的四大组件都需要在此处注册，权限申请也在此处声明

- test

用于编写UnitTest测试用例，同样可以对项目进行自动化测试

- .gitignore

将app模块内指定的目录或文件排除在版本控制之外

- build.gradle

app模块的gradle构建脚本，指定项目构建相关的配置文件

- proguard-rules.pro

用于指定项目代码的混淆规则，代码开发完成打包成apk，可通过代码混淆增加代码的阅读难度，防止恶意破解
<a name="eiQFR"></a>
#### src/main/res文件夹目录结构
![image.png](https://cdn.nlark.com/yuque/0/2020/png/1419739/1599714242731-ecd2de99-5f1f-4eb4-9dd1-602897adbba1.png#align=left&display=inline&height=169&margin=%5Bobject%20Object%5D&name=image.png&originHeight=337&originWidth=306&size=13158&status=done&style=none&width=153)

- drawable

以drawable开头的文件均用于存放图片，不同的drawable用于提供不同的图片大小以适配不同尺寸的设备

- mipmap

一般用于存放图标，同样为了设备兼容性等提供不同尺寸的图标等

- value

一般用于存放字符串，颜色，样式等配置

- layout

存放活动的布局文件，配合Activity就是app中所见到的内容
