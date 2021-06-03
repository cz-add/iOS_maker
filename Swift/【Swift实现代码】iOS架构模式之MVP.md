1.什么是MVP？

MVP是模型（Model）、视图（View）、主持人（Presenter）的缩写，分别代表项目中3个不同的模块。　

1.1 模型 (Model):负责处理数据的加载或存储

1.2 视图 (View):负责界面数据的展示与用户交互

1.3 主持人(Presenter):是Model和View之间的桥梁,将两者进行链接。

![输入图片说明](https://images.gitee.com/uploads/images/2021/0601/152852_2a841d05_9027123.jpeg "mbp.jpg")


整个交互流程看起来大致是这样的:

用户交互->View获得交互事件->View将事件转发给Presenter->Presenter调用Model获取新数据->Presenter将数据推送给View进行展示

案例1：

这里我们用app开发中常用的登录功能为例,用mvp来实现一个登录逻辑(功能)。既然用MVP 那么我们得新建三个类即:LoginModel,LoginPresenter,LoginView

```
class loginPresenter: NSObject {
    //声明V和M2个属性，其中的V中写了代理，待优化
    private var loginViewDelegate:LoginViewDelegate?
    private var loginModel:LoginModel?

    //实例化
    override init() {
        //model实例化
        self.loginModel = LoginModel()
    }

    //V层调用这个login方法，这个方法再调用M层的login方法
    func login(usrName: String, pwd: String)   {
        self.loginModel?.login(usrName: usrName, pwd: pwd, callback: { (result) in
            //从m层的的回调，回调到v层去，同样还是通过一个代理实现
            self.loginViewDelegate?.onLoginResult(result: result)
        })
    }

    //绑定V和P
    func attachView(viewDelegate:LoginViewDelegate)  {
        self.loginViewDelegate = viewDelegate
    }

    //解除绑定，假如网络请求是，viewController已经释放，则无需再回调更新UI
    func detachView()  {
        self.loginViewDelegate = nil
    }

}
```

Presenter

```
import Foundation
//M层
class LoginModel: NSObject {
    //登陆的方法，P层调用这个方法来发起登陆请求
    func login(usrName:String,pwd:String,callback:((String)->Void))  {
        //发起网络请求 处理方法要封装，不能耦合
        print("进入model")
        //调用网络模块方法
        HttpUtils.post(usrName: usrName, pwd: pwd) { (result) in
            //1，处理网络返回的情况，如：登录成功要缓存个人信息
               //......
            //2，完成登录数据处理，回调给P层，这里不与UI部分耦合
            callback(result)
        }

    }

}
```

Model

```
import UIKit
//遵循LoginViewDelegate协议
class ViewController: UIViewController,LoginViewDelegate {

    //定义一个presenter，实例化
    private let presenter = loginPresenter()

    override func viewDidLoad() {
        super.viewDidLoad()
        //添加v和p2层的绑定
        self.presenter.attachView(viewDelegate: self)
        //UI层交互操作，发起登录请求
        self.presenter.login(usrName: "ZZB", pwd: "123456")

    }

    //依据P层的回调数据进行V层UI更新
    func onLoginResult(result: String) {
        print("处理P层返回的数据: \(result)，更新UI")
    }

    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
        //页面注销的时候解除绑定
        self.presenter.detachView()
    }

}
```

ViewController

案例2：

```
class ViewController: UIViewController {

    fileprivate lazy var presenter : ViewPresenster = {
        return ViewPresenster(presenter: self)
    }()

    override func viewDidLoad() {
        super.viewDidLoad()
     }

    override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
        presenter.getData()
    }
}

// MARK:- 获取数据//
extension ViewController:ViewPresensterProtocol{

    func showPost(_ resulet: [DCModel]) {
        print(resulet)
    }
}
```

ViewController

```
import UIKit

protocol ViewPresensterProtocol {
    func showPost(_ resulet: [DCModel])
}

class ViewPresenster: NSObject {
    var presenter: ViewPresensterProtocol!
    lazy var model:[DCModel] = [DCModel]()
    init(presenter:ViewPresensterProtocol) {
        self.presenter = presenter;
    }

    func getData(){
        let dict = [
            ["user_id":"1","user_name":"zhaodacai1"],
            ["user_id":"2","user_name":"zhaodacai2"],
            ["user_id":"3","user_name":"zhaodacai3"],
            ["user_id":"4","user_name":"zhaodacai4"],
            ["user_id":"5","user_name":"zhaodacai5"],
            ["user_id":"6","user_name":"zhaodacai6"],
            ["user_id":"7","user_name":"zhaodacai7"]
        ]

        for item in dict {
            model.append(DCModel(dict: item))
        }

        self.presenter.showPost(model)
    }

}
```

Presenter

```
import UIKit

class DCModel: NSObject {

    // 用户ID
    var user_id : String = ""

    // 用户名字
    var user_name : String = ""

    init(dict : [String : Any]) {
        super.init()
        setValuesForKeys(dict)
    }

    override func setValue(_ value: Any?, forUndefinedKey key: String) {}
}
```

