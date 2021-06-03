> Appium是目前比较好用的跨平台自动化测试框架，在iOS端采用WebDriverAgent作为webdriver驱动，实现了自动化脚本编写到运行的全流程覆盖。

在Xcode 8之前，基于UI Automation的自动化测试方案是比较好用且非常流行的。但在Xcode 8之后，苹果在instruments工具集中直接废除了Automation组件，转而支持使用UI Testing。

### UI Testing

从Xcode 7开始，苹果提供了UI Testing框架，也就是我们在APP test工程中使用的XCTest的那一套东西。UI Testing包含几个重要的类，分别是XCUIApplication、XCUIElement、XCUIElementQuery。

*   XCUIApplication

代表正在测试的应用程序的实例，可以对APP进行启动、终止、传入参数等操作。

```
- (void)launch;
- (void)activate;
- (void)terminate;
@property (nonatomic, copy) NSArray <NSString *> *launchArguments;
@property (nonatomic, copy) NSDictionary <NSString *, NSString *> *launchEnvironment;
```

XCUIApplication在iOS上提供了两个初始化接口：

```
//Returns a proxy for the application specified by the "Target Application" target setting.
- (instancetype)init NS_DESIGNATED_INITIALIZER;

//Returns a proxy for an application associated with the specified bundle identifier.
- (instancetype)initWithBundleIdentifier:(NSString *)bundleIdentifier NS_DESIGNATED_INITIALIZER;

```

其中initWithBundleIdentifier接口允许传入一个bundle id来操作指定APP。这个技术点是iOS APP能够自动化测试的关键所在。

*   XCUIElement

表示界面上显示的UI元素。

*   XCUIElementQuery

用于定位UI元素的查询对象。

上述几个模块就是一个UI测试框架的核心能力，后面在写Appium的自动化脚本时也是一样的套路：启动APP->定位UI元素->触发操作。

### WebDriverAgent

[WebDriverAgent](https://github.com/facebookarchive/WebDriverAgent)是Facebook开发的基于XCTest.framework的开源项目，实现了在[iOS](https://easeapi.com/blog/tags/iOS.html)上支持WebDriver协议的服务，可以用来启动/终止APP，点击/滑动页面。

[webdriver协议](https://w3c.github.io/webdriver/)是一套基于HTTP协议的JSON格式规范，协议规定了不同操作对应的格式。之所以需要这层协议，是因为iOS、Android、浏览器等都有自己的UI交互方式，通过这层”驱动层“屏蔽各平台的差异，就可以通过相同的方式进行自动化的UI操作，做网络爬虫常用的selenium是浏览器上实现webdriver的驱动，而WebDriverAgent则是iOS上实现webdriver的驱动。

使用Xcode打开WebDriverAgent项目，连接上iPhone设备之后，选中WebDriverAgentRunner->Product->Test，则会在iPhone上安装一个名为WebDriverAgentRunner的APP，这个APP实际上是一个后台应用，直接点击ICON打开的话会退出。

具体到代码层面，WebDriverAgentRunner的入口在UITestingUITests.m文件

```
- (void)testRunner
{
  FBWebServer *webServer = [[FBWebServer alloc] init];
  webServer.delegate = self;
  [webServer startServing];
}
```

会在手机上6100端口启动一个HTTP server，startServing方法内部就是一个死循环，监听网络传输过来的webdriver协议的数据，解析并处理点击事件。

有意思的是，WebDriverAgent并没有使用XCUIApplication的initWithBundleIdentifier方法，而是使用了initPrivateWithPath的私有方法。测试发现两者效果上没什么区别，这里暂时不清楚为什么不直接使用initWithBundleIdentifier。

#### WebDriverAgent如何处理点击事件的？

从WebDriverAgent的源码可以清晰的看到，在Commands目录，是支持的操作类集合。每一个操作都通过routes类方法注册对应的路由和处理该路由的函数。手势处理在FBTouchActionCommands.m中，

```
+ (NSArray *)routes
{
  return
  @[
    [[FBRoute POST:@"/wda/touch/perform"] respondWithTarget:self action:@selector(handlePerformAppiumTouchActions:)],
    [[FBRoute POST:@"/wda/touch/multi/perform"] respondWithTarget:self action:@selector(handlePerformAppiumTouchActions:)],
    [[FBRoute POST:@"/actions"] respondWithTarget:self action:@selector(handlePerformW3CTouchActions:)],
  ];
}
```

可以看到/wda/touch/perform、/wda/touch/multi/perform、/actions路由负责处理不同的点击事件。那么当一个点击的url请求过来时，如何转化为iOS的UIEvent事件呢？跟踪代码发现核心代码是：

```
[[XCUIDevice.sharedDevice eventSynthesizer] synthesizeEvent:record completion:(id)^(BOOL result, NSError *invokeError) {
    handlerBlock(record, invokeError);
}];
```

XCUIDevice的eventSynthesizer是私有方法，通过synthesizeEvent发送XCSynthesizedEventRecord（也是私有类）事件。到这里WebDriverAgent的流程就很清除了。实际上由于使用了很多私有方法，WebDriverAgent并非仅能自动化当前APP，也是可以操作手机屏幕以及任意APP的。

### Appium

上面介绍的UI Testing和WebDriverAgent都是iOS上的内容。实际上，在更多的时候我们需要考虑跨平台，从笔者调研来看，现阶段比较好的方案是Appium。Appium是一个开源测试自动化框架，可用于iOS、Android和web应用程序测试，支持python、JAVA、[php](https://easeapi.com/blog/tags/PHP.html)等多种语言编写测试用例。Appium包含客户端和服务端等模块。

#### Appium客户端

在iOS上的客户端实际上就是使用了WebDriverAgent，作为实现webdriver协议的驱动层。

#### Appium服务端

Appium的服务端是一个桌面应用，用于和客户端通信，启动Appium的服务端之后，会在电脑上启动一个默认端口号是4723的HTTP服务。当我们编写完脚本执行时，脚本代码会被转换为webdriver协议的JSON数据，通过HTTP请求发送到电脑的4723端口。Appium服务端将脚本的执行请求下发给客户端（请求客户端的6100端口），客户端同样使用webdriver协议响应。

### Appium的部署

#### 安装各种驱动

```
brew install libimobiledevice  
npm install -g ios-deploy   
#appium-doctor可检查依赖是否安装成功  
npm install -g appium-doctor   
#检查依赖项  
appium-doctor -ios  
```

#### 安装[appium-desktop](https://github.com/appium/appium-desktop/releases)

下载安装之后，需要对WebDriverAgent修改下证书信息，使其可以真机调试。

```
cd /Applications/Appium.app/Contents/Resources/app/node_modules/appium/node_modules/appium-youiengine-driver/node_modules/appium-webdriveragent  
./Scripts/bootstrap.sh -d  
```

连接真机后，打开WebDriverAgent.xcodeproj，Product->Test运行WebDriverAgentRunner（需要配置证书）。终端会显示一个地址 `http://169.254.138.98:8100` 访问 `http://169.254.138.98:8100/status`打开有json信息则表示服务连接成功。

需要注意的是，Facebook原始的WebDriverAgent功能上似乎有些问题，appium-desktop自带的WebDriverAgent版本据说是经过优化修改过的，运行正常。

部署完成之后打开Appium，Start Server后即可编写脚本了。一个典型的monkey脚本如下：

```
#!/usr/bin/python
# coding=utf-8
import unittest
import os
from appium import webdriver
from time  import sleep
from appium.webdriver.common.touch_action import TouchAction
import random

class AppTest(unittest.TestCase):

    def setUp(self):
        print("setUp")
        udid = "c9d719e41f6a9ae00353db126a6561fdf032cc23"
        bundleId = "com.easeapi"

        if False:
            #new app
            app = os.path.abspath('/Users/easeapi.app')
            self.driver = webdriver.Remote(command_executor = 'http://127.0.0.1:4723/wd/hub', desired_capabilities = {'app':app,'platformName': 'iOS', 'platformVersion': '12.4.3', 'deviceName': 'EaseapiPhone', 'bundleId': bundleId, 'udid': udid})
        else:
            #exist app
            self.driver = webdriver.Remote(command_executor = 'http://127.0.0.1:4723/wd/hub', desired_capabilities = {'platformName': 'iOS', 'platformVersion': '12.4.3', 'deviceName': 'EaseapiPhone', 'bundleId': bundleId, 'udid': udid})

    def tearDown(self):
        #self.driver.quit()

    def test_monkey(self):
        __size = self.driver.find_element_by_xpath('//UIAApplication[1]').size
        window_width = __size['width']
        window_height = __size['height']
        while True: 
            sleep(1.0)
            _x = random.uniform(0, window_width)
            _y = random.uniform(0, window_height)
            print('x：%f y：%f') % (_x, _y)
            TouchAction(self.driver).tap(x=_x, y=_y).perform()

if __name__ == '__main__':
    suite = unittest.TestLoader().loadTestsFromTestCase(AppTest)
    unittest.TextTestRunner(verbosity=2).run(suite)
```

执行该脚本，即可对bundle id为com.easeapi的app进行monkey测试。如果想自定义事件也比较简单：

```
button = self.driver.find_element_by_accessibility_id("按钮")
button.click()
```

很多时候，编写脚本是一件很繁琐无趣的工作，好在Appium提供了录制的功能。我们上面仅仅启动了Appium的服务。实际上通过Appium也是可以直接在电脑上操作手机的。打开Appium，点击搜索按钮，在JSON Representation区域填写如下模板内容：

```
{
  "platformName": "ios",
  "platformVersion": "12.4.3",
  "udid": "c9d719e41f6a9ae00353db126a6561fdf032cc23",
  "deviceName": "EaseapiPhone",
  "automationName": "XCUITest",
  "bundleId": "com.easeapi",
  "xcodeSigningId": "iPhone Developer"
}

```

完成后点击Start Session，即可在电脑上直接操作iPhone手机，使用录制功能可以将每一步的点击操作转化为脚本文件，大大较少了开发工作量。

### 最后

以上就是Appium的实现原理和简单使用介绍，总体来看Appium是一个比较优秀的自动化测试方案，值得推广使用。最后简单说下其他方案的调研结果：

*   fastmonkey：在Xcode11上已无法运行。
*   swiftmonkey：使用Swift3.4开发，最新Xcode已经不支持这个版本的Swift。


