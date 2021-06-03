>开发中我们会使用到第三方的SDK，有的时候也会将整个系统的公用的功能的抽象出来成为FrameWork，我们只需要暴露对外的接口，使用者只需要调用接口，对于内部实现的过程不需要维护，可以以库的形式进行封装，只暴露出头文件。库（FrameWork）是编译好的二进制文件，编译的时候只需要 Link 一下，提高浪费编译时间，库分为静态库和动态库。

## 基础知识

静态库即静态链接库（Windows 下的 .lib，Linux 和 Mac 下的 .a）。之所以叫做静态，是因为静态库在编译的时候会被直接拷贝一份，复制到目标程序里，这段代码在目标程序里就不会再改变了。静态库的好处很明显，编译完成之后，库文件实际上就没有作用了。目标程序没有外部依赖，直接就可以运行。当然其缺点也很明显，就是会使用目标程序的体积增大。

动态库动态库即动态链接库（Windows 下的 .dll，Linux 下的 .so，Mac 下的 .dylib）。与静态库相反，动态库在编译时并不会被拷贝到目标程序中，目标程序中只会存储指向动态库的引用。等到程序运行时，动态库才会被真正加载进来。动态库的优点是，不需要拷贝到目标程序中，不会影响目标程序的体积，而且同一份库可以被多个程序使用（因为这个原因，动态库也被称作**共享库**）。同时，编译时才载入的特性，也可以让我们随时对库进行替换，而不需要重新编译代码。动态库带来的问题主要是，动态载入会带来一部分性能损失，使用动态库也会使得程序依赖于外部环境。如果环境缺少动态库或者库的版本不正确，就会导致程序无法运行（Linux 下喜闻乐见的 lib not found 错误）。

## 动态库

xCode6之后制作动态库相比之前简单很多，xCode7基本上沿袭了xCode6的操作，细节方面有差别。在 iOS 8 之前，iOS 平台不支持使用动态 Framework，开发者可以使用的 Framework 只有苹果基础的 UIKit.Framework，Foundation.Framework 等。iOS 8/Xcode 6 推出之后，iOS 平台添加了动态库的支持，同时 Xcode 6 也原生自带了 Framework 支持（动态和静态都可以），iOS8多了 Extension ，Extension 和 App 是两个分开的可执行文件，同时需要共享代码，这种情况下动态库的支持就是必不可少的了。但是这种动态 Framework 和系统的 UIKit.Framework 还是有很大区别。系统的 Framework 不需要拷贝到目标程序中，我们自己做出来的 Framework 哪怕是动态的，最后也还是要拷贝到 App 中（App 和 Extension 的 Bundle 是共享的），因此苹果又把这种 Framework 称为Embedded FrameWork.



1.新建动态库File→New→Target→Cocoa Touch FrameWork:

![输入图片说明](https://images.gitee.com/uploads/images/2021/0528/135137_c8dc47c0_9027123.png "动1.png")

2.项目名称DynamicLibrary,同时我们新建两个测试文件FEUIImage，TestImage:

![输入图片说明](https://images.gitee.com/uploads/images/2021/0528/135146_2386e84c_9027123.png "动2.png")

3.在Dynamic.h中导入头文件:
```
#import <UIKit/UIKit.h>
 
//! Project version number for DynamicLibrary.
FOUNDATION_EXPORT double DynamicLibraryVersionNumber;
 
//! Project version string for DynamicLibrary.
FOUNDATION_EXPORT const unsigned char DynamicLibraryVersionString[];
 
// In this header, you should import all the public headers of your framework using statements like #import <DynamicLibrary/PublicHeader.h>
#import <DynamicLibrary/FEUIImage.h>

```

4.将FEUIImage和TestImage设置为Public供外部访问:

![输入图片说明](https://images.gitee.com/uploads/images/2021/0528/135156_c9ff3098_9027123.png "动3.png")

5.cmb+b编译项目，编辑成功之后将项目移动到项目中进行测试:

![输入图片说明](https://images.gitee.com/uploads/images/2021/0528/135204_9c72badf_9027123.png "动4.png")

6.通过FEDevice项目找到编译之后的DynamicLibrary.framwork文件

![输入图片说明](https://images.gitee.com/uploads/images/2021/0528/135214_31620106_9027123.png "动5.png")

![输入图片说明](https://images.gitee.com/uploads/images/2021/0528/135224_39232585_9027123.png "动6.png")

## 通用动态库

我们将上面的DynamicLibrary.framework移动的其他项目中是可以直接使用的，但是运行的时候会出错， 错误信息如下:
```
Undefined symbols for architecture x86_64:
  "_OBJC_CLASS_$_FEUIImage", referenced from:
      objc-class-ref in ViewController.o
ld: symbol(s) not found for architecture x86_64
clang: error: linker command failed with exit code 1 (use -v to see invocation)
```
错误信息跟动态的指令集和目标项目的指令集版本有关系，我们可以简单的了解下Achitectures和设备之间的关系，iPhone一直以来都是Arm处理器，Arm是处理器是移动设备上占用率最大的处理器。 在iOS模拟器上运行的是x86指令集，只有在真机上才会执行arm指令集，每个指令集对应不同的机型设置:

都是arm处理器的指令集。通常指令是向下兼容的。在模拟器运行时，iOS模拟器运行的是x86指令集。只有在真机上，才会对执行arm指令集。

armv6：iPhone，iPhone 2G/3G，iPod 1G/2G，xCode4.5已经不支持armv6指令集； 

armv7 ：iPhone 3GS，iPhone4，iPhone 4s，iPad，iPad2，iPad3(The New iPad)，iPad mini，iPod Touch 3G，iPod Touch4，由于iPhone4s的占有率，目前是指令集的最低版本；

armv7s：iPhone5， iPhone5C，iPad4和iPod5；

arm64：iPhone5s，iPhone6，iPhone6 Plus，iPad Air，iPad mini2(iPad mini with Retina Display)，iPhone6s，iPhone6s Plus

我们可以通过lipo命令查看发现i386是mac版指令集:
```
lipo -info DynamicLibrary.framework/DynamicLibrary
Non-fat file: DynamicLibrary.framework/DynamicLibrary is architecture: i386
```
1.如果用真机调试的话，同样会发生程序错误，所以需要将同样的代码同时支持模拟器和真机，两份库聚合一下,回到DynamicLibrary.frameWork项目,通过File→New→Target→Other→Aggregate：

![输入图片说明](https://images.gitee.com/uploads/images/2021/0528/135234_a8c0c8bf_9027123.png "动7.png")

2.执行编译脚本地址，先选中DynamicLibrary-Universal，添加脚本地址:
```
/${PROJECT_DIR}/DynamicLibrary/ios-framework-universal-script.sh
```
![输入图片说明](https://images.gitee.com/uploads/images/2021/0528/135245_f34c36ba_9027123.png "动8.png")

3.设置脚本内容，同时设置DynamicLibrary-Universal的依赖项(Target Dependencies)为DynamicLibrary:
```
#!/bin/sh
 
#  ios-build-framework-script.sh
#  DynamicLibrary 博客园-FlyElephant
#  博客园:http://www.cnblogs.com/xiaofeixiang/
#  Created by keso on 16/1/17.
#  Copyright © 2016年 FlyElephant. All rights reserved.
set -e
set +u
### Avoid recursively calling this script.
if [[ $UF_MASTER_SCRIPT_RUNNING ]]
then
exit 0
fi
set -u
export UF_MASTER_SCRIPT_RUNNING=1
### Constants.
UF_TARGET_NAME=${PROJECT_NAME}
FRAMEWORK_VERSION="A"
UNIVERSAL_OUTPUTFOLDER=${BUILD_DIR}/${CONFIGURATION}-universal
IPHONE_DEVICE_BUILD_DIR=${BUILD_DIR}/${CONFIGURATION}-iphoneos
IPHONE_SIMULATOR_BUILD_DIR=${BUILD_DIR}/${CONFIGURATION}-iphonesimulator
### Functions
## List files in the specified directory, storing to the specified array.
#
# @param $1 The path to list
# @param $2 The name of the array to fill
#
##
list_files ()
{
filelist=$(ls "$1")
while read line
do
eval "$2[\${#$2[*]}]=\"\$line\""
done <<< "$filelist"
}
### Take build target.
if [[ "$SDK_NAME" =~ ([A-Za-z]+) ]]
then
SF_SDK_PLATFORM=${BASH_REMATCH[1]} # "iphoneos" or "iphonesimulator".
else
echo "Could not find platform name from SDK_NAME: $SDK_NAME"
exit 1
fi
### Build simulator platform. (i386, x86_64)
echo "========== Build Simulator Platform =========="
echo "===== Build Simulator Platform: i386 ====="
xcodebuild -project "${PROJECT_FILE_PATH}" -target "${TARGET_NAME}" -configuration "${CONFIGURATION}" -sdk iphonesimulator BUILD_DIR="${BUILD_DIR}" OBJROOT="${OBJROOT}" BUILD_ROOT="${BUILD_ROOT}" CONFIGURATION_BUILD_DIR="${IPHONE_SIMULATOR_BUILD_DIR}/i386" SYMROOT="${SYMROOT}" ARCHS='i386' VALID_ARCHS='i386' $ACTION
echo "===== Build Simulator Platform: x86_64 ====="
xcodebuild -project "${PROJECT_FILE_PATH}" -target "${TARGET_NAME}" -configuration "${CONFIGURATION}" -sdk iphonesimulator BUILD_DIR="${BUILD_DIR}" OBJROOT="${OBJROOT}" BUILD_ROOT="${BUILD_ROOT}" CONFIGURATION_BUILD_DIR="${IPHONE_SIMULATOR_BUILD_DIR}/x86_64" SYMROOT="${SYMROOT}" ARCHS='x86_64' VALID_ARCHS='x86_64' $ACTION
### Build device platform. (armv7, arm64)
echo "========== Build Device Platform =========="
echo "===== Build Device Platform: armv7 ====="
xcodebuild -project "${PROJECT_FILE_PATH}" -target "${TARGET_NAME}" -configuration "${CONFIGURATION}" -sdk iphoneos BUILD_DIR="${BUILD_DIR}" OBJROOT="${OBJROOT}" BUILD_ROOT="${BUILD_ROOT}"  CONFIGURATION_BUILD_DIR="${IPHONE_DEVICE_BUILD_DIR}/armv7" SYMROOT="${SYMROOT}" ARCHS='armv7 armv7s' VALID_ARCHS='armv7 armv7s' $ACTION
echo "===== Build Device Platform: arm64 ====="
xcodebuild -project "${PROJECT_FILE_PATH}" -target "${TARGET_NAME}" -configuration "${CONFIGURATION}" -sdk iphoneos BUILD_DIR="${BUILD_DIR}" OBJROOT="${OBJROOT}" BUILD_ROOT="${BUILD_ROOT}" CONFIGURATION_BUILD_DIR="${IPHONE_DEVICE_BUILD_DIR}/arm64" SYMROOT="${SYMROOT}" ARCHS='arm64' VALID_ARCHS='arm64' $ACTION
### Build device platform. (arm64, armv7)
echo "========== Build Universal Platform =========="
## Copy the framework structure to the universal folder (clean it first).
rm -rf "${UNIVERSAL_OUTPUTFOLDER}"
mkdir -p "${UNIVERSAL_OUTPUTFOLDER}"
## Copy the last product files of xcodebuild command.
cp -R "${IPHONE_DEVICE_BUILD_DIR}/arm64/${PROJECT_NAME}.framework" "${UNIVERSAL_OUTPUTFOLDER}/${PROJECT_NAME}.framework"
### Smash them together to combine all architectures.
lipo -create  "${BUILD_DIR}/${CONFIGURATION}-iphonesimulator/i386/${PROJECT_NAME}.framework/${PROJECT_NAME}" "${BUILD_DIR}/${CONFIGURATION}-iphonesimulator/x86_64/${PROJECT_NAME}.framework/${PROJECT_NAME}" "${BUILD_DIR}/${CONFIGURATION}-iphoneos/armv7/${PROJECT_NAME}.framework/${PROJECT_NAME}" "${BUILD_DIR}/${CONFIGURATION}-iphoneos/arm64/${PROJECT_NAME}.framework/${PROJECT_NAME}" -output "${UNIVERSAL_OUTPUTFOLDER}/${PROJECT_NAME}.framework/${PROJECT_NAME}"
### Create standard structure for framework.
#
# If we don't have "Info.plist -> Versions/Current/Resources/Info.plist", we may get error when use this framework.
#
# MyFramework.framework
# |-- MyFramework -> Versions/Current/MyFramework
# |-- Headers -> Versions/Current/Headers
# |-- Resources -> Versions/Current/Resources
# |-- Info.plist -> Versions/Current/Resources/Info.plist
# `-- Versions
#     |-- A
#     |   |-- MyFramework
#     |   |-- Headers
#     |   |   `-- MyFramework.h
#     |   `-- Resources
#     |       |-- Info.plist
#     |       |-- MyViewController.nib
#     |       `-- en.lproj
#     |           `-- InfoPlist.strings
#     `-- Current -> A
#
echo "========== Create Standard Structure =========="
mkdir -p "${UNIVERSAL_OUTPUTFOLDER}/${PROJECT_NAME}.framework/Versions/${FRAMEWORK_VERSION}/"
mv "${UNIVERSAL_OUTPUTFOLDER}/${PROJECT_NAME}.framework/${PROJECT_NAME}" "${UNIVERSAL_OUTPUTFOLDER}/${PROJECT_NAME}.framework/Versions/${FRAMEWORK_VERSION}/"
mv "${UNIVERSAL_OUTPUTFOLDER}/${PROJECT_NAME}.framework/Headers" "${UNIVERSAL_OUTPUTFOLDER}/${PROJECT_NAME}.framework/Versions/${FRAMEWORK_VERSION}/"
mkdir -p "${UNIVERSAL_OUTPUTFOLDER}/${PROJECT_NAME}.framework/Resources"
declare -a UF_FILE_LIST
list_files "${UNIVERSAL_OUTPUTFOLDER}/${PROJECT_NAME}.framework/" UF_FILE_LIST
for file_name in "${UF_FILE_LIST[@]}"
do
if [[ "${file_name}" == "Info.plist" ]] || [[ "${file_name}" =~ .*\.lproj$ ]]
then
mv "${UNIVERSAL_OUTPUTFOLDER}/${PROJECT_NAME}.framework/${file_name}" "${UNIVERSAL_OUTPUTFOLDER}/${PROJECT_NAME}.framework/Resources/"
fi
done
mv "${UNIVERSAL_OUTPUTFOLDER}/${PROJECT_NAME}.framework/Resources" "${UNIVERSAL_OUTPUTFOLDER}/${PROJECT_NAME}.framework/Versions/${FRAMEWORK_VERSION}/"
ln -sfh "Versions/Current/Resources/Info.plist" "${UNIVERSAL_OUTPUTFOLDER}/${PROJECT_NAME}.framework/Info.plist"
ln -sfh "${FRAMEWORK_VERSION}" "${UNIVERSAL_OUTPUTFOLDER}/${PROJECT_NAME}.framework/Versions/Current"
ln -sfh "Versions/Current/${PROJECT_NAME}" "${UNIVERSAL_OUTPUTFOLDER}/${PROJECT_NAME}.framework/${PROJECT_NAME}"
ln -sfh "Versions/Current/Headers" "${UNIVERSAL_OUTPUTFOLDER}/${PROJECT_NAME}.framework/Headers"
ln -sfh "Versions/Current/Resources" "${UNIVERSAL_OUTPUTFOLDER}/${PROJECT_NAME}.framework/Resources"
### Open the universal folder.
open "${UNIVERSAL_OUTPUTFOLDER}"
```
上面的脚本内容在脚本执行的过程中，会依次编译出支持 i386、x86_64、arm64、armv7、armv7s 的包，然后把各个包中的库文件通过 lipo 工具合并为一个支持各平台的通用库文件，再基于最后一个 xcodebuild 命令打出的包的结构(arm64/DynamicLibrary.framework)和这个通用库文件生成一个支持各个平台的通用 Framwork；最后编译之后会弹出framework的位置:

![输入图片说明](https://images.gitee.com/uploads/images/2021/0528/135300_d4bff1ab_9027123.png "动9.png")

4.设置了通用库之后，还需要在Genral下Embedded Binaries添加一下动态库:

![输入图片说明](https://images.gitee.com/uploads/images/2021/0528/135309_8a4e784f_9027123.png "动10.png")

## 静态库

动态 Framework 是开发中优先考虑的代码打包方式，但是为了兼容一些低版本系统对动态库的限制，我们需要打包静态库来使用，实现起来比较简单，在原有的DynamicLibrary项目 Build Settings 下设置 Mach-O Type 值为 Static Library，可以编译出静态库。

![输入图片说明](https://images.gitee.com/uploads/images/2021/0528/135316_823fb2d6_9027123.png "动11.png")

设置为静态库之后动态库的最后一步Embedded Binaries就不用再添加了，如果已经添加建议删除~

