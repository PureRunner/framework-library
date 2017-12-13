

## 打包 framework文件

Framework是资源的集合，包含静态库及其头文件，在OS X上，可能会创建一个动态连接（Dynamically Linked）的framework。通过动态连接，framework就可以更新（应用不需要重新连接）。在运行时，库中代码的一份拷贝被分享出来，整个工程都可以使用它，因此，这样减少了内存消耗，提高了系统的性能，这是一个功能强大的特性。

但在iOS上，你不能用这种方式添加为系统添加自定义的framework，因此仅有的动态链接的framework只能是Apple提供的那些。（在iOS 8中已加入此特性，开发者可以使用第三方的动态框架）
而静态连接的framework依然可以打包代码，使其在不同的应用中得以复用。由于framework本质上是静态库的“一站式采购点”。

#### 新建framework 
>1. 新建一个工程,选择```(library & FrameWork)```中的 ``` Cocoa Touch Framework```。
>2. 在工程中TARGETS，选择要打包的FrameWork: 
>
>>* 暴露出所要提供使用的头文件，设置：```（Build Phases --> Headers --> Public -->添加xxx.h）```
>>
>>* Build Active Architecture Only 设置为NO
>>
>>*  Shared Scheme：```(Manage Schemes --> HexColors --> 勾选Shared)```
>>
>>* 如果在iPhone5c上使用framework,需要在 ```Build Setting --> Architectures -->Architectures``` 中添加armv7s.

#### 自动编译
>1. 在工程中新建一个Target,选择Other -->Aggregate,重命名MyAggregate。
>
>2. 选中MyAggregate,点击Build Pharas ,选中左上角的 + , New Run Script Phase。
>
>3. 点开Run Script,添加如下的脚本。

```
# Sets the target folders and the final framework product.
# 如果工程名称和Framework的Target名称不一样的话，要自定义FMKNAME
# 例如: FMK_NAME = "MyFramework"
FMK_NAME=${PROJECT_NAME}
# Install dir will be the final output to the framework.
# The following line create it in the root folder of the current project.
INSTALL_DIR=${SRCROOT}/Products/${FMK_NAME}.framework
# Working dir will be deleted after the framework creation.
WRK_DIR=build
DEVICE_DIR=${WRK_DIR}/Release-iphoneos/${FMK_NAME}.framework
SIMULATOR_DIR=${WRK_DIR}/Release-iphonesimulator/${FMK_NAME}.framework
# -configuration ${CONFIGURATION}
# Clean and Building both architectures.
xcodebuild -configuration "Release" -target "${FMK_NAME}" -sdk iphoneos clean build
xcodebuild -configuration "Release" -target "${FMK_NAME}" -sdk iphonesimulator clean build
# Cleaning the oldest.
if [ -d "${INSTALL_DIR}" ]
then
rm -rf "${INSTALL_DIR}"
fi
mkdir -p "${INSTALL_DIR}"
cp -R "${DEVICE_DIR}/" "${INSTALL_DIR}/"
# Uses the Lipo Tool to merge both binary files (i386 + armv6/armv7) into one Universal final product.
lipo -create "${DEVICE_DIR}/${FMK_NAME}" "${SIMULATOR_DIR}/${FMK_NAME}" -output "${INSTALL_DIR}/${FMK_NAME}"
rm -r "${WRK_DIR}"
open "${INSTALL_DIR}"
```

#### 运行打包
选择MyAggregate， Command + B编译,编译完成后会自动弹出已经打包完成的framework。

#### 测试

如果在```framework ``` 中使用了 ```Category``` ,在测试中程序出现了 ```Unrecognized selector sent to instance``` 的错误,需要工程中设置 ```Build settings -> Other Linker Flags 设置-all_load``` .


#### 发布 .framework
[Carthage](http://www.jianshu.com/p/bf263c596538)
>* $ carthage build --no-skip-current
>* $ git tag 1.0.0
>* $ git push --tags



## 注意

>* 在静态库中使用类别来扩展已有类的时候，编辑器不能把类原有的方法和类别中的方法整合起来，就会导致你调用类别中的方法时，出现"selector not recognized"，也就是找不到方法定义的错误。为了解决这个问题，引入了 ```(Build Settings --> Other Linker Flags --> -ObjC)```标志，它的作用就是将静态库中所有的和对象相关的文件都加载进来。不过在64位的Mac系统或者iOS系统下，编辑器有一个bug，会导致只包含有类别的静态库无法使用 ```-ObjC``` 标志来加载文件。那么就要使用 ```-all_load``` 或者 ```-force_load``` 标志，它们的作用都是加载静态库中所有文件，不过 ```all_load``` 作用于所有的库，而 ```-force_load``` 后面必须要指定具体的文件。
>
>
>* Build Active Architecture Only设置为NO的时候，会编译支持的所有的版本设置为YES的时候，是为Debug的时候速度更快，它只编译当前的architecture 版本
>* iOS APP 在许多不同的CPU架构下运行：
>
>>* arm7: 在最老的支持iOS7的设备上使用
>>* arm7s: 在iPhone5和5C上使用
>>* arm64: 运行于iPhone5S的64位 ARM 处理器 上
>>* i386: 32位模拟器上使用
>>* x86_64: 64为模拟器上使用




## 打包 .a文件
#### 新建  Static Library
>1. 新建一个工程,选择```(library & FrameWork)```中的 ``` Cocoa Touch Static Library```。
>2. 在工程中TARGETS，选择要打包的libxxx.a: 
>
>>* 暴露出所要提供使用的头文件，设置：```（Build Phases --> new Headers phase--> Headers --> Public -->添加xxx.h）```
>>
>>* Edit Scheme --> Run --> info --> Build configuration Debug
>>
>>* Build Active Architecture Only 设置为NO
>>
>>* 如果在iPhone5c上使用framework,需要在 ```Build Setting --> Architectures -->Architectures``` 中添加armv7s.

#### Build 

分别在真机及模拟器下编译libxxx.a文件。
lipo -create ".../Debug-iphoneos/libxxx.a" ".../Debug-iphonesimulator/libxxx.a" -output "合并后的libxxx.a路径"



#### 测试

如果在```libxxx.a ``` 中使用了 ```Category``` ,在测试中程序出现了 ```Unrecognized selector sent to class ``` 的错误,检查 ```Build Pharse -> Other Linker Flags ```是否为 ```-all_load``` ,或者检查libxxx.a是否同时支持真机及模拟器在运行。









 
 
 
