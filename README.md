# 什么是ReactNative?
移动端平台能够运行 JavaScript 代码，JavaScript 不仅仅可以传递配置信息，还可以表达逻辑信息的优点。[相关学习链接](http://blog.cnbang.net/tech/2698/)

# ReactNative原理
JavaScript 的形式告诉 Objective-C 该执行什么代码。

<pre><code>
RCTRootView *rootView = [[RCTRootView alloc] initWithBundleURL:jsCodeLocation
                                                    moduleName:@"PropertyFinder"
                                             initialProperties:nil
                                                 launchOptions:launchOptions];
</code></pre>
初始化方法的核心是 setUp 方法，而 setUp 方法的主要任务则是创建 BatchedBridge。
BatchedBridge 的作用是批量读取 JavaScript 对 Objective-C 的方法调用，同时它内部持有一个 JavaScriptExecutor，顾名思义，这个对象用来执行 JavaScript 代码。

## JavaScriptExecutor 关键方法是 start 
* 读取 JavaScript 源码
* 初始化模块信息
* 初始化 JavaScript 代码的执行器，即 RCTJSCExecutor 对象
* 生成模块列表并写入 JavaScript 端
* 执行 JavaScript 源码

## 调用逻辑
![](/ReactNative2.png)

1. JS端调用某个OC模块暴露出来的方法。

2. 把上一步的调用分解为ModuleName,MethodName,arguments，再扔给MessageQueue处理。
在初始化时模块配置表上的每一个模块都生成了对应的remoteModule对象，对象里也生成了跟模块配置表里一一对应的方法，这些方法里可以拿到自身的模块名，方法名，并对callback进行一些处理，再移交给MessageQueue。具体实现在BatchedBridgeFactory.js的_createBridgedModule里，整个实现区区24行代码，感受下JS的魔力吧。

3. 在这一步把JS的callback函数缓存在MessageQueue的一个成员变量里，用CallbackID代表callback。在通过保存在MessageQueue的模块配置表把上一步传进来的ModuleName和MethodName转为ModuleID和MethodID。

4. 把上述步骤得到的ModuleID,MethodId,CallbackID和其他参数argus传给OC。至于具体是怎么传的，后面再说。

5. OC接收到消息，通过模块配置表拿到对应的模块和方法。
实际上模块配置表已经经过处理了，跟JS一样，在初始化时OC也对模块配置表上的每一个模块生成了对应的实例并缓存起来，模块上的每一个方法也都生成了对应的RCTModuleMethod对象，这里通过ModuleID和MethodID取到对应的Module实例和RCTModuleMethod实例进行调用。具体实现在_handleRequestNumber:moduleID:methodID:params:。

6. RCTModuleMethod对JS传过来的每一个参数进行处理。
RCTModuleMethod可以拿到OC要调用的目标方法的每个参数类型，处理JS类型到目标类型的转换，所有JS传过来的数字都是NSNumber，这里会转成对应的int/long/double等类型，更重要的是会为block类型参数的生成一个block。

例如-(void)select:(int)index response:(RCTResponseSenderBlock)callback 这个方法，拿到两个参数的类型为int,block，JS传过来的两个参数类型是NSNumber,NSString(CallbackID)，这时会把NSNumber转为int，NSString(CallbackID)转为一个block，block的内容是把回调的值和CallbackID传回给JS。

这些参数组装完毕后，通过NSInvocation动态调用相应的OC模块方法。

7. OC模块方法调用完，执行block回调。

8. 调用到第6步说明的RCTModuleMethod生成的block。

9. block里带着CallbackID和block传过来的参数去调JS里MessageQueue的方法invokeCallbackAndReturnFlushedQueue。

10. MessageQueue通过CallbackID找到相应的JS callback方法。

11. 调用callback方法，并把OC带过来的参数一起传过去，完成回调。

整个流程就是这样，简单概括下，差不多就是：JS函数调用转ModuleID/MethodID -> callback转CallbackID -> OC根据ID拿到方法 -> 处理参数 -> 调用OC方法 -> 回调CallbackID -> JS通过CallbackID拿到callback执行


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
<pre><code>#import "TopsTechSaaS-Swift.h"</code></pre>
* 如果访问Swift的static值或是单列相关数据，可以建一个桥接类，在桥接类里对OC要访问的值对外访问即可。
* Swift访问Object-C需要导入<br>
<pre><code>TopsTechSaaS-Bridging-Header.h</code></pre> 
但是有的第三方库在TopsTechSaaS-Bridging-Header文件中导入相关.h文件也是无法访问，需要导入第三方文件如：
<pre><code>
import RxSwift
import RxCocoa
</code></pre>
*ReactNative访问Object-C需要建立RCT_EXPORT_MODULE通讯，以回调的方式像RN回传，RN使用<pre><code>const UserInfoSync = NativeModules.UserInfoSync;</code></pre>导入原生模块访问相关数据。



