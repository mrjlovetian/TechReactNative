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
<pre><code>
let path = Bundle.main.path(forResource: "index.ios", ofType: "jsbundle")
let rootView = RCTRootView(bundleURL: NSURL(string: path!)! as URL, moduleName: "RNforSaaS", initialProperties: nil, launchOptions: nil);
self.view.addSubview(rootView!);
</code></pre>
* [如何打包](https://segmentfault.com/a/1190000004189538)

## 跨平台传值
* Object-C访问Swift的值需要导入<br>
<pre><code>#import "TopsTechSaaS-Swift.h"</code><pre>
    * 如果访问Swift的static值或是单列相关数据，可以建一个桥接类，在桥接类里对OC要访问的值对外访问即可。
* Swift访问Object-C需要导入<br>
<pre><code>TopsTechSaaS-Bridging-Header.h</code></pre> 
但是有的第三方库在TopsTechSaaS-Bridging-Header文件中导入相关.h文件也是无法访问，需要导入第三方文件如：
<pre><code>
import RxSwift
import RxCocoa
</code></pre>
*ReactNative访问Object-C需要建立RCT_EXPORT_MODULE通讯，以回调的方式像RN回传，RN使用<pre><code>const UserInfoSync = NativeModules.UserInfoSync;</code></pre>导入原生模块访问相关数据。



