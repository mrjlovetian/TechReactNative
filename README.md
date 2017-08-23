# ReactNative基本集成
## 如何创建一个ReactNative项目
### [搭建环境](http://reactnative.cn/docs/0.47/getting-started.html#content)安装Homebrew、Node、React Native的命令行工具、Xcode。
### [项目创建](http://reactnative.cn/docs/0.47/getting-started.html#%E4%BF%AE%E6%94%B9%E9%A1%B9%E7%9B%AE)
* react-native init AwesomeProject创建一个项目工程
* react-native run-ios运行一个项目工程在ios设备上
* lsof -i tcp:port查看一个端口的占用情况
* kill -9 port结束这个进程


## 混合模式开发，手机集成ReactNative
* node_modules文件放在项目根目录下使用pod集成到项目中。
* assets(静态资源)、index.ios.jsbundle(编译代码)、index.ios.jsbundle.meta必要的支持文件放在项目模块中。
* 使用的时候
'''
let path = Bundle.main.path(forResource: "index.ios", ofType: "jsbundle")
        let rootView = RCTRootView(bundleURL: NSURL(string: path!)! as URL, moduleName: "RNforSaaS", initialProperties: nil, launchOptions: nil);
        self.view.addSubview(rootView!);
'''
* [如何打包](https://segmentfault.com/a/1190000004189538)

## 跨平台传值




