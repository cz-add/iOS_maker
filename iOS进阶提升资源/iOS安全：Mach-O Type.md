> iOS/maxOS二进制文件是mach-o格式的，mach-o又分为几种不同的类型。本文介绍了常见的mach-o文件类型以及它们的不同之处。

在Xcode->Build Setting -> Mach-O Type中，我们可以选择下面几种类型：

*   Executable
*   Dynamic Library
*   Bundle
*   Static Library
*   Relocatable Object File

新建Framework项目中，默认是Dynamic Library。

实际上，这里的Mach-o Type完整的定义在mach-o/loader.h struct mach_header_64结构的filetype：

*   #define MH_OBJECT 0x1 /* relocatable object file */
*   #define MH_EXECUTE 0x2 /* demand paged executable file */
*   #define MH_FVMLIB 0x3 /* fixed VM shared library file */
*   #define MH_CORE 0x4 /* core file */
*   #define MH_PRELOAD 0x5 /* preloaded executable file */
*   #define MH_DYLIB 0x6 /* dynamically bound shared library */
*   #define MH_DYLINKER 0x7 /* dynamic link editor */
*   #define MH_BUNDLE 0x8 /* dynamically bound bundle file */
*   #define MH_DYLIB_STUB 0x9 /* shared library stub for static */ /* linking only, no section contents */
*   #define MH_DSYM 0xa /* companion file with only debug */ /* sections */
*   #define MH_KEXT_BUNDLE 0xb /* x86_64 kexts */

各种类型说明如下：

*   MH_OBJECT

.m，.c等文件编译出来的目标文件，文件后缀是.o。Static Library类型产出是MH_OBJECT类型文件的archieve。

*   MH_EXECUTE

可单独执行的可执行文件，必须有main方法作为执行入口，app的二进制就是这种类型；对应Executable类型产出；

*   MH_FVMLIB
*   MH_CORE
*   MH_PRELOAD
*   MH_DYLIB

动态库文件，包括.dylib文件，动态framework；对应Dynamic Library类型产出。

*   MH_DYLINKER

动态链接器，在 iOS中就是/usr/lib/dyld；

*   MH_BUNDLE

独立的二进制文件，不支持在项目中添加Link Binary使用。可以在Copy Bundle Resources中作为资源添加。 通过NSBundle load的方式加载；对应Bundle类型产出。典型的例子就是/System/Library/AccessibilityBundles目录的.axbundle后缀的文件。

*   MH_DSYM

存储二进制文件符号信息的文件，用于Debug分析；

/usr/lib/dyld仅会处理MH_BUNDLE, MH_DYLIB，MH_EXECUTE类型文件。
