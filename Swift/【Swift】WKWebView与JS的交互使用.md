## 一、前言　　

　　近日，有朋友问我关于WKWebView与JS的交互问题，可我之前一直使用的是UIWebView，也不曾做过WKWebView的交互啊！接下来大家一块学习下WKWebView是怎么实现原生代码和JS交互的。

## 二、WKWebView

*   支持更多的HTML5的特性

*   高达60fps滚动刷新频率与内置手势

*   与Safari相容的JavaScript引擎

*   在性能、稳定性方面有很大提升占用内存更少 协议方法及功能都更细致

*   可获取加载进度等。

## 三、WKWebView的代理方法
```
/*! @abstract The web view's navigation delegate. */
weak open var navigationDelegate: WKNavigationDelegate?
 
/*! @abstract The web view's user interface delegate. */
weak open var uiDelegate: WKUIDelegate?
```
## 三、WKNavigationDelegate的代理方法
```
`//判断链接是否允许跳转`

`optional func webView(_ webView: WKWebView, decidePolicyFor navigationAction: WKNavigationAction, decisionHandler: ``@escaping` `(WKNavigationActionPolicy) -> Void)`

`//拿到响应后决定是否允许跳转`

`optional func webView(_ webView: WKWebView, decidePolicyFor navigationResponse: WKNavigationResponse, decisionHandler: ``@escaping` `(WKNavigationResponsePolicy) -> Void)`

`//链接开始加载时调用`

`optional func webView(_ webView: WKWebView, didStartProvisionalNavigation navigation: WKNavigation!)`

`//收到服务器重定向时调用`

`optional func webView(_ webView: WKWebView, didReceiveServerRedirectForProvisionalNavigation navigation: WKNavigation!)`

`//加载错误时调用`

`optional func webView(_ webView: WKWebView, didFailProvisionalNavigation navigation: WKNavigation!, withError error: Error)`

`//当内容开始到达主帧时被调用（即将完成）`

`optional func webView(_ webView: WKWebView, didCommit navigation: WKNavigation!)`

`//加载完成`

`optional func webView(_ webView: WKWebView, didFinish navigation: WKNavigation!)`

`//在提交的主帧中发生错误时调用`

`optional func webView(_ webView: WKWebView, didFail navigation: WKNavigation!, withError error: Error)`

`//当webView需要响应身份验证时调用(如需验证服务器证书)`

`optional func webView(_ webView: WKWebView, didReceive challenge: URLAuthenticationChallenge, completionHandler: ``@escaping` `(URLSession.AuthChallengeDisposition, URLCredential?) -> Void)`

`//当webView的web内容进程被终止时调用。(iOS 9.0之后)`

`optional func webViewWebContentProcessDidTerminate(_ webView: WKWebView)`

```
>作为一个开发者，有一个学习的氛围跟一个交流圈子特别重要，这是一个我的iOS交流群：[642363427](https://jq.qq.com/?_wv=1027&k=bESXgU0E)，不管你是小白还是大牛欢迎入驻 ，分享BAT,阿里面试题、面试经验，讨论技术， iOS开发者一起交流学习成长！

## 四、WKUIDelegate的代理方法

　　用来做一些页面上的事件，弹窗警告，提醒等。

```

//接收到警告面板
 optional func webView(_ webView: WKWebView, runJavaScriptAlertPanelWithMessage message: String, initiatedByFrame frame: WKFrameInfo, completionHandler: @escaping () -> Void)
   
 //接收到确认面板
optional func webView(_ webView: WKWebView, runJavaScriptConfirmPanelWithMessage message: String, initiatedByFrame frame: WKFrameInfo, completionHandler: @escaping (Bool) -> Void)
   
 //接收到输入框
optional func webView(_ webView: WKWebView, runJavaScriptTextInputPanelWithPrompt prompt: String, defaultText: String?, initiatedByFrame frame: WKFrameInfo, completionHandler: @escaping (String?) -> Void)

```

## 五、WKWebView与JS的交互使用

#### 　　首页创建html文件，代码如下：
```
<html lang="en">
     
    <head>
        <meta charset="utf-8" />
        <title>JS交互</title>
         
        <style>
             
            body {
                font-size:30px;
                text-align:center;
            }
         
        * {
            margin: 30px;
            padding: 0;
        }
         
        h1{
            color: red;
        }
         
        button{
            width: 300px;
            height: 50px;
            font-size: 30px;
        }
         
            </style>
         
    </head>
    <body>
        <h1>WKWebview与iOS交互</h1>
        <h2></h2>
        <button onclick="testA()">点击alert弹框</button>
        <button onclick="testB('我是弹窗内容')">点击alert有参弹窗</button>
        <button onclick="testConfrim()">点击confrim弹窗</button>
        <button onclick="buttonAction()">向iOS端传递数据</button>
        <script type="text/javascript">
             
            //无参数函数
            function testA() {
                alert("我是JS中的弹窗消息");
            }
         
            //有参数函数
            function testB(value) {
                alert(value);
            }
         
            function testC(value) {
                return value + "value";
            }
         
            //接受iOS端传过来的参数，
            function testObject(name,age) {
                var object = {name:name,age:age};
                return object;
            }
         
            function testConfrim() {
                comfirm("确定修改数据吗？")
            }
         
            function buttonAction(){
                try {
                    <!-- js 向iOS 传递数据-->
                    window.webkit.messageHandlers.getMessage.postMessage("我是js传递过来的数据")
                }catch (e) {
                    console.log(e)
                }
            }
        </script>
         
    
    </body>
</html>

```

####  　　1、iOS调用js中的方法进行并传参

```

//案例1
self.webView?.evaluateJavaScript("testInput('123')", completionHandler: { (data
            , error) in
            print(data as Any)
        })
//案例2
self.webView?.evaluateJavaScript("testObject('xjf',26)", completionHandler: { (data, err) in
            print("\(String(describing: data)),\(String(describing: err))")
        })
```

#### 　　2、js向iOS传递数据

```



 func userContentController(_ userContentController: WKUserContentController, didReceive message: WKScriptMessage) {
        print("\(message.name)" + "\(message.body)")
//        message.name 方法名
//        message.body 传递的数据
    }

```

#### 　　3、在js中点击按钮，进行弹窗实现

```

//MARK:WKUIDelegate
//此方法作为js的alert方法接口的实现，默认弹出窗口应该只有提示消息，及一个确认按钮，当然可以添加更多按钮以及其他内容，但是并不会起到什么作用
//点击确认按钮的相应事件，需要执行completionHandler，这样js才能继续执行
////参数 message为  js 方法 alert(<message>) 中的<message>
func webView(_ webView: WKWebView, runJavaScriptAlertPanelWithMessage message: String, initiatedByFrame frame: WKFrameInfo, completionHandler: @escaping () -> Void) {
    let alertViewController = UIAlertController(title: "提示", message:message, preferredStyle: UIAlertController.Style.alert)
    alertViewController.addAction(UIAlertAction(title: "确认", style: UIAlertAction.Style.default, handler: { (action) in
        completionHandler()
    }))
    self.present(alertViewController, animated: true, completion: nil)
}
 
// confirm
//作为js中confirm接口的实现，需要有提示信息以及两个相应事件， 确认及取消，并且在completionHandler中回传相应结果，确认返回YES， 取消返回NO
//参数 message为  js 方法 confirm(<message>) 中的<message>
func webView(_ webView: WKWebView, runJavaScriptConfirmPanelWithMessage message: String, initiatedByFrame frame: WKFrameInfo, completionHandler: @escaping (Bool) -> Void) {
    let alertVicwController = UIAlertController(title: "提示", message: message, preferredStyle: UIAlertController.Style.alert)
    alertVicwController.addAction(UIAlertAction(title: "取消", style: UIAlertAction.Style.cancel, handler: { (alertAction) in
        completionHandler(false)
    }))
    alertVicwController.addAction(UIAlertAction(title: "确定", style: UIAlertAction.Style.default, handler: { (alertAction) in
        completionHandler(true)
    }))
    self.present(alertVicwController, animated: true, completion: nil)
}
 
// prompt
//作为js中prompt接口的实现，默认需要有一个输入框一个按钮，点击确认按钮回传输入值
//当然可以添加多个按钮以及多个输入框，不过completionHandler只有一个参数，如果有多个输入框，需要将多个输入框中的值通过某种方式拼接成一个字符串回传，js接收到之后再做处理
//参数 prompt 为 prompt(<message>, <defaultValue>);中的<message>
//参数defaultText 为 prompt(<message>, <defaultValue>);中的 <defaultValue>
func webView(_ webView: WKWebView, runJavaScriptTextInputPanelWithPrompt prompt: String, defaultText: String?, initiatedByFrame frame: WKFrameInfo, completionHandler: @escaping (String?) -> Void) {
    let alertViewController = UIAlertController(title: prompt, message: "", preferredStyle: UIAlertController.Style.alert)
    alertViewController.addTextField { (textField) in
        textField.text = defaultText
    }
    alertViewController.addAction(UIAlertAction(title: "完成", style: UIAlertAction.Style.default, handler: { (alertAction) in
        completionHandler(alertViewController.textFields![0].text)
    }))
    self.present(alertViewController, animated: true, completion: nil)
}

```

#### 　　4、获取网页中节点的数据 

```

//网页加载完成
-(void)webView:(WKWebView *)webView didFinishNavigation:(WKNavigation *)navigation{
    //设置JS
    NSString *js = @"document.getElementsByTagName('h1')[0].innerText";
    //执行JS
    [webView evaluateJavaScript:js completionHandler:^(id _Nullable response, NSError * _Nullable error) {
        NSLog(@"value: %@ error: %@", response, error);
         
    }];
}

```

#### 　　5、通过注入JS修改节点的内容

```
let js = "document.getElementsByTagName('h2')[0].innerText = '这是一个iOS写入的方法'";
//将js注入到网页中

```

#### 　　6、js获取DOM节点的几种方式

```
document.getElementById();//id名，
document.getElementsByTagName();//标签名
document.getElementsByClassName();//类名
document.getElementsByName();//name属性值，一般不用
document.querySelector();//css选择符模式，返回与该模式匹配的第一个元素，结果为一个元素；如果没找到匹配的元素，则返回null
document.querySelectorAll()//css选择符模式，返回与该模式匹配的所有元素，结果为一个类数组

```

##  六、**JavaScriptCore**

　　JavaScriptCore 这个库是 Apple 在 iOS 7 之后加入到标准库的，它对 iOS Native 与 JS 做交互调用产生了划时代的影响。

 　　JavaScriptCore 大体是由 4 个类以及 1 个协议组成的：

![输入图片说明](https://images.gitee.com/uploads/images/2021/0601/153233_754d46aa_9027123.png "js1.png")

*   JSContext 是 JS 执行上下文，你可以把它理解为 JS 运行的环境。

*   JSValue 是对 JavaScript 值的引用，任何 JS 中的值都可以被包装为一个 JSValue。

*   JSManagedValue 是对 JSValue 的包装，加入了“conditional retain”。

*   JSVirtualMachine 表示 JavaScript 执行的独立环境。

　　还有 JSExport 协议：

```

实现将原生类及其实例方法，类方法和属性导出为 JavaScript 代码的协议。

```

　　这里的 JSContext，JSValue，JSManagedValue 相对比较好理解，下面我们把 JSVirtualMachine 单拎出来说明一下：

　　**JSVirtualMachine 的用法和其与 JSContext 的关系**

![输入图片说明](https://images.gitee.com/uploads/images/2021/0601/153249_60eb9141_9027123.png "js2.png")

　　JSVirtualMachine 实例表示用于 JavaScript 执行的独立环境。 您使用此类有两个主要目的：支持并发 JavaScript 执行，并管理 JavaScript 和 Objective-C 或 Swift 之间桥接的对象的内存。

　　关于 JSVirtualMachine 的使用，一般情况下我们不用手动去创建 JSVirtualMachine。因为当我们获取 JSContext 时，获取到的 JSContext 从属于一个 JSVirtualMachine。

　　每个 JavaScript 上下文（JSContext 对象）都属于一个 JSVirtualMachine。 每个 JSVirtualMachine 可以包含多个上下文，允许在上下文之间传递值（JSValue 对象）。 但是，每个 JSVirtualMachine 是不同的，即我们不能将一个 JSVirtualMachine 中创建的值传递到另一个 JSVirtualMachine 中的上下文。

　　JavaScriptCore API 是线程安全的 —— 例如，我们可以从任何线程创建 JSValue 对象或运行 JS 脚本 - 但是，尝试使用相同 JSVirtualMachine 的所有其他线程将被阻塞。 要在多个线程上同时（并发）运行 JavaScript 脚本，请为每个线程使用单独的 JSVirtualMachine 实例。

七、案例源码：

```
class NAHomeViewController : UIViewController,WKNavigationDelegate,WKScriptMessageHandler,WKUIDelegate,UINavigationControllerDelegate {
     
    var webView : WKWebView?
    var content : JSContext?
    var userContentController : WKUserContentController?
    override func viewDidLoad() {
        super.viewDidLoad()
        self.navigationItem.title = "首页"
        
        //创建配置对象
        let configuration = WKWebViewConfiguration()
        //为WKWebViewController设置偏好设置
        let preference = WKPreferences()
        configuration.preferences = preference
         
        //允许native与js交互
        preference.javaScriptEnabled = true
 
        //初识化webView
        let webView = WKWebView.init(frame: CGRect(x: 0, y: 64, width: self.view.frame.size.width, height: 300))
        let path = Bundle.main.path(forResource: "wKWebView", ofType: "html")
        webView.navigationDelegate = self
        webView.uiDelegate = self
        let request = URLRequest.init(url: URL.init(fileURLWithPath: path!))
        webView.load(request)
        self.view.addSubview(webView)
        self.webView = webView
         
        let userContentController = WKUserContentController()
        configuration.userContentController = userContentController
        userContentController.add(self,name: "getMessage")
        self.userContentController = userContentController
         
        let btn = UIButton.init(frame: CGRect(x: 100, y: 390, width: 100, height: 50))
        btn.setTitleColor(.black, for: .normal)
        btn.setTitle("oc调用js", for: .normal)
        btn.addTarget(self, action: #selector(btnAction), for: .touchUpInside)
        self.view.addSubview(btn)
         
        self.view.backgroundColor = .white
    }
     
    @objc func btnAction() {
        self.webView?.evaluateJavaScript("testInput('123')", completionHandler: { (data
            , error) in
            print(data as Any)
        })
        self.webView?.evaluateJavaScript("testObject('xjf',26)", completionHandler: { (data, err) in
            print("\(String(describing: data)),\(String(describing: err))")
        })
    }
     
    func userContentController(_ userContentController: WKUserContentController, didReceive message: WKScriptMessage) {
        print("\(message.name)" + "\(message.body)")
//        message.name 方法名
//        message.body 传递的数据
    }
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
        // Dispose of any resources that can be recreated.
    }
     
    func webView(_ webView: WKWebView, didFinish navigation: WKNavigation!) {
//        webView.evaluateJavaScript("testA()") { (data, err) in
//            print("\(String(describing: data)),\(String(describing: err))")
//        }
    }
     
    func webView(_ webView: WKWebView, didFail navigation: WKNavigation!, withError error: Error) {
        print("\(error)")
    }
     
     
    //MARK:WKUIDelegate
    //此方法作为js的alert方法接口的实现，默认弹出窗口应该只有提示消息，及一个确认按钮，当然可以添加更多按钮以及其他内容，但是并不会起到什么作用
    //点击确认按钮的相应事件，需要执行completionHandler，这样js才能继续执行
    ////参数 message为  js 方法 alert(<message>) 中的<message>
    func webView(_ webView: WKWebView, runJavaScriptAlertPanelWithMessage message: String, initiatedByFrame frame: WKFrameInfo, completionHandler: @escaping () -> Void) {
        let alertViewController = UIAlertController(title: "提示", message:message, preferredStyle: UIAlertController.Style.alert)
        alertViewController.addAction(UIAlertAction(title: "确认", style: UIAlertAction.Style.default, handler: { (action) in
            completionHandler()
        }))
        self.present(alertViewController, animated: true, completion: nil)
    }
     
    // confirm
    //作为js中confirm接口的实现，需要有提示信息以及两个相应事件， 确认及取消，并且在completionHandler中回传相应结果，确认返回YES， 取消返回NO
    //参数 message为  js 方法 confirm(<message>) 中的<message>
    func webView(_ webView: WKWebView, runJavaScriptConfirmPanelWithMessage message: String, initiatedByFrame frame: WKFrameInfo, completionHandler: @escaping (Bool) -> Void) {
        let alertVicwController = UIAlertController(title: "提示", message: message, preferredStyle: UIAlertController.Style.alert)
        alertVicwController.addAction(UIAlertAction(title: "取消", style: UIAlertAction.Style.cancel, handler: { (alertAction) in
            completionHandler(false)
        }))
        alertVicwController.addAction(UIAlertAction(title: "确定", style: UIAlertAction.Style.default, handler: { (alertAction) in
            completionHandler(true)
        }))
        self.present(alertVicwController, animated: true, completion: nil)
    }
     
    // prompt
    //作为js中prompt接口的实现，默认需要有一个输入框一个按钮，点击确认按钮回传输入值
    //当然可以添加多个按钮以及多个输入框，不过completionHandler只有一个参数，如果有多个输入框，需要将多个输入框中的值通过某种方式拼接成一个字符串回传，js接收到之后再做处理
    //参数 prompt 为 prompt(<message>, <defaultValue>);中的<message>
    //参数defaultText 为 prompt(<message>, <defaultValue>);中的 <defaultValue>
    func webView(_ webView: WKWebView, runJavaScriptTextInputPanelWithPrompt prompt: String, defaultText: String?, initiatedByFrame frame: WKFrameInfo, completionHandler: @escaping (String?) -> Void) {
        let alertViewController = UIAlertController(title: prompt, message: "", preferredStyle: UIAlertController.Style.alert)
        alertViewController.addTextField { (textField) in
            textField.text = defaultText
        }
        alertViewController.addAction(UIAlertAction(title: "完成", style: UIAlertAction.Style.default, handler: { (alertAction) in
            completionHandler(alertViewController.textFields![0].text)
        }))
        self.present(alertViewController, animated: true, completion: nil)
    }
}

```
