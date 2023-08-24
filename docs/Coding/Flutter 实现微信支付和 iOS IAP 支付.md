> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/6949949215577473061)

> 公司近期将收费的功能排期了，由于项目做的是线上教育，提供的服务属于虚拟物品。根据 iOS 官方的规定，虚拟物品交易只能使用 iOS 应用内支付，其他类似微信、支付宝官方都是明文规定不允许存在的。(注：虚拟物品才有此规定，且 iOS 官方收税 30%；而有实体物品交易的，官方允许在提供应用内支付的前提下，提供其他支付方式供用户选择。)

结合相关平台规定，我们最终确定支付方式为：**Android 端使用微信支付，iOS 使用 IAP 应用内支付。**

微信支付
----

> 不得不说我们这一代程序员是幸运的，得益于国内移动支付的迅猛发展，微信支付的流程闭环比 iOS 完善了 N 倍（iOS 的槽点一篇文章都写不完，稍后我再来吐）；同时微信官方所提供的服务，至少在国内网络中，可以认定为是百分百可靠的。

*   微信支付的流程相对简单：

1.  客户端向业务后台发起一个购买请求
2.  业务后台**到微信服务端生成一个订单**
3.  将微信订单信息和自身系统所需的业务数据整合后返回给客户端
4.  客户端拿到微信支付信息后，通过 WeChatOpensdk **调起支付**
5.  在页面中**订阅支付回调**，接受支付信息并做业务流程处理 **（如：进入支付结果页等流程）**
6.  最后请求后台，由**后台主动去微信系统中查询最终支付状态**，交回给前端显示结果。**（ps：后端在微信系统中主动查询订单转态是同步的，可以马上拿到支付结果）**

*   接下来讲讲开发，Flutter 使用的是 fluwx 插件，简单易用。在项目中，我对微信支付进行了封装，代码见下：

```
import 'dart:async';

import 'package:flutter/foundation.dart';
import 'package:fluwx/fluwx.dart' as fluwx;

class WechatPayment {
  StreamSubscription _wxPay;

  /// 关闭微信消息订阅
  void wxSubscriptionClose() => _wxPay?.cancel();

  /// 调起微信支付面板 
  /// 这里的WxPayModel是业务层的数据，即后台返回的有关微信支付订单的信息
  void wxPay(WxPayModel wxPayModel, {VoidCallback onWxPaying, VoidCallback onSuccess, Function(String data) onError}) async {
    // 跳转微信支付前，告诉页面进入微信支付，页面层可以做一些关闭加载框等的操作
    onWxPaying?.call();
    // 一些异常情况的处理
    if (!await fluwx.isWeChatInstalled) return onError?.call('请安装微信完成支付或使用苹果手机支付');
    if (wxPayModel.appId != Config.WX_APP_ID) return onError?.call('AppID不一致，请联系管理员');
    // 此方法笔者没有做单例，因此支付前尝试注销监听，避免重复回调
    _wxPay?.cancel();
    // 支付回调
    _wxPay = fluwx.weChatResponseEventHandler.listen((event) {
      _wxPay?.cancel();
      if (event is fluwx.WeChatPaymentResponse) {
        if (event.isSuccessful) {
          return onSuccess?.call();
        } else {
          return onError?.call(event.errCode == -1 ? '系统错误，请联系管理员' : '您取消了支付');
        }
      }
    });

    // 发起支付
    fluwx.payWithWeChat(
      appId: wxPayModel.appId,
      partnerId: wxPayModel.partnerId,
      prepayId: wxPayModel.prepayId,
      packageValue: wxPayModel.packageValue,
      nonceStr: wxPayModel.nonceStr,
      timeStamp: wxPayModel.timeStamp,
      sign: wxPayModel.sign,
      signType: wxPayModel.signType,
      extData: wxPayModel.extData,
    );
  }
}



```

页面端是这样调用的

```
WechatPayment paymentUtils = new WechatPayment();
paymentUtils.wxPay(
    state.model.wxPayModel,
    onError: (String err) {
        if (!mounted) return;
        // 微信支付错误，设置支付状态为false，弹框即可
         _isPaying = false;
         SchedulerBinding.instance.addPostFrameCallback((_) {
           CommonUtils.showToast(err, backgroundColor: Theme.of(context).errorColor);
         });
      }, 
      onSuccess:(){ 
        _isPaying = true;
      },
      onWxPaying: () {
        // 启动微信支付，设置支付状态为true，关闭加载框
        _isPaying = true;
        SchedulerBinding.instance.addPostFrameCallback((_) {
          Navigator.pop(context);
        });
   },
);


```

但是需要注意，微信的回调是异步的，并且有很多种情况是接收不到回调的，以下是确定收不到会调的情况。

```
微信调起支付页面时，其实是跳转到新的应用，对于我们的应用而言是触发了前后台切换的生命周期。
因此在检测到应用返回前台，并且支付状态还在进行中时，可以证明是收不到微信的支付状态回调，需要特殊处理下。
收不到的情况有：
// ① 弹出支付框后使用系统返回键关闭；
// ② 进入微信支付密码框后不输入使用系统导航切回app或者系统返回键返回；
// ③ 进入微信后直接返回桌面再回到应用；
// ④ 弹出支付框后锁屏再开屏；
// ⑤ 弹出支付款后下拉任务栏；
// ⑥ 输入密码成功后，直接返回桌面或者使用系统导航或者使用返回键返回app
// ⑦ 退出微信登录，进行支付后直接登录微信，在登录过程中回到app
// ⑧ 在系统应用管理中双开微信后，调起支付后不点击任一个微信端，而是点击取消



```

现在主流的做法是再支付页面监听 app 的生命周期，即由后台切回前台的时候，检测下状态，若还在支付中，直接进入查询结果页面，由后台去检验订单，拿到结果显示即可。**（后台主动查询理论上还是存在微信服务端延时的问题，因此后台进行查询的时候，建议采取轮询机制，若是没有支付成功的话，延时 5 秒后再确认下更保险）**

```
class _XXXPageState extends State<XXXPage> with WidgetsBindingObserver {
  @override
  void initState() {
    super.initState();
    WidgetsBinding.instance.addObserver(this); //添加观察者
  }

  @override
  void dispose() {
    WidgetsBinding.instance.removeObserver(this); //销毁观察者
    super.dispose();
  }

  /// 应用状态监听
  @override
  void didChangeAppLifecycleState(AppLifecycleState state) {
    switch (state) {
      case AppLifecycleState.resumed:
        {
          if (Platform.isAndroid && _isPaying) {
            _isPaying = false;
          // 监听到时安卓设备并且支付还在进行中，程序员要根据业务做一下处理
            break;
        }
      default:
        break;
    }
    super.didChangeAppLifecycleState(state);
  }
}


```

到此，微信支付很愉快的解决了，以上代码是抽象出来的工具类，可以直接使用；**但是不涉及任何业务流程的开发，这个需要使用者自己去补充。** 综上，微信支付流程主线可简单粗暴总结为：服务端生成订单 → 客户端调起支付 → 客户端通知服务端核验订单 → 客户端拿到最终结果 → 客户端 final 支付。 整个过程形成闭环，有理有据，数据都由后端去操作安全合理。(最重点是前端工作量简直不要太少)。

> _**可是，iOS 就不一样了，简直不要太恶心！**_

iOS IAP 应用内支付
-------------

*   IAP，即 in-app Purchase，苹果推出的 App 内购买虚拟商品的方式，基于 AppStore 账户的支付方式。由于 iOS 整个体系都是基于自己的一套系统的（不像上面的微信支付，是第三方支付平台），因此在开发之前，我们需要到 Apple 开发者中心完成以下步骤： 1. 签署协议和银行业务 2. 在后台创建 App 内购买项目，这里所有的价格都是 Apple 规定好的，我们只有选择的资格，没办法自定价格。**创建完成后，每个项目会有 sku 和 productId** 3. 添加沙盒测试员 Apple [以上步骤参考内容引自站内大神：](https://link.juejin.cn?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2F59c73aa68023 "https://www.jianshu.com/p/59c73aa68023")Geniune
*   支付流程：应用通过 sku 向服务端获取商品列表 → 列表中取出对应产品请求支付 → 进入 appStore 支付 → 页面监听支付回调拿到验证票据 → 业务后台拿到应用接收到的票据后去 Apple 官网进行校验即可。 流程很简单，简单到几乎不用跟业务后台打交代，但是坑却随之而来：

```
① 支付数据完全依赖前端应用，很难跟业务后台的订单系统一一对应；
② 针对①的问题，IAP支付支持传递skPayment对象，里面的applicationUsername经常用来保存系统的OrderId；
但是应用支付成功后收到的回调中，applicationUsername却偶尔会出现为null的情况，没有了对应关系，就没办法核销业务系统中的订单从而为用户充值；
③ iOS支付回调非常不稳定，有时延迟严重；且没有任何注定查询的方法；
④ iOS应用内支付有很多异常情况要处理，最常见的就是没有登录、没有同意最新的iOS支付协议等，都会发送给app支付失败的回调；
但是当用户登录或是同意后，iOS系统又会触发新的支付，导致旧的附带业务订单号的支付无效，莫名又多出一个没有订单号的新支付；
⑤ 国内网上资料极度缺乏，基本都是19年以前的，Flutter的文章更是少的可怜，可参考性不强。
⑥ 测试文档对于中断购买的测试流程有巨坑，后面菜单一定不要错过~


```

> 通过查看文档和不断调试，我们发现： ① 支付错误的回调，基本能马上收到； ② 上面流程说到 IAP 支付需要手动结束支付流程。同时 iOS 规定不能对同一个 skuId 重复发起多次支付的，只要当前 skuId 有没有 final 的支付，再次发起都会失败； ② 无论支付成功或失败，只要 app 没有主动对当前支付进行 final，每次启动 app 后，app 都会收到这个支付信息的通知； ③ 关于 applicationUsername，只有在支付完成马上收到回调的情况下，回调信息才会有这个信息；到②中的情况，肯定不会返回 applicationUsername； ④ 没有 applicationUsername 就意味着订单对不上，因此我们需要进行凑单机制。

> 综上，我们对异常处理有了确定方案： ① app 发起支付后，需要将业务 OrderId 和 skuId 进行持久化存储（即卸载应用都不会删除的数据）； ②只要持久化存储不为空，启动 app 就需要马上启动监听，以接收 iOS 系统的订单推送； ③ 支付出错可以 final 当前支付，但是支付成功必须明确接收到 iOS 推送并且后台核验成功后，才能 final，并删除持久化存储。

最终，结合到业务系统和特殊情况的处理后，支付流程应该如下：

1.  业务后台返回商品列表时，**需要附加返回对应的 skuId**
2.  app 通过 skuId 请 appStore 请求商品信息
3.  app 对商品发起支付，并**将业务订单号存储在 applicationUsername 中，发起成功写入持久化存储，状态为 pending**
4.  接收 iOS 系统回调，**失败马上 final 支付，更改对应持久化存储状态为 cancle**；成功拿到**票据和业务 OrderId** 发送给后台
5.  后台调取 Apple 服务端接口，传入票据（票据其实储存着最新的时间，appStore 用户信息等）
6.  后台获取到 Apple 返回的当前 appStore 用户所有支付的前 100 条记录，拿到 productId 到数据库有中匹配该用户是否有未核销的订单，并对应修改业务订单状态
7.  app 确认核销成功，**final 支付，并且删除持久化存储**

同时还需要做一些特殊处理：

1.  app 刚启动时，若是持久化存储不为空，需要马上启动 iOS 支付订阅监听，以接收 iOS 对未完成订单的推送；
2.  由于 iOS 限制了同一个 skuId 不能重复发起支付，因此持久化存储中，一个 skuId 永远只会有一条记录。因此当 app 接收到的支付推送 applicationUsername 为 null，采取凑单机制，原则是：通过 skuId 找到存储记录，拿到其对应的 OrderId，发给后台核验。

*   接下来进入开发，Futter 采用的是 in_app_purchase 插件，官方提供的，支持 google 和 IAP 支付；而持久化存储用的是 flutter_secure_storage 插件。 依据上面的流程，我同样封装了工具类。而且由于可能会在多个地方调用起监听，所有必须是单例模式，代码如下：

```
import 'dart:async';

import 'package:flutter_secure_storage/flutter_secure_storage.dart';
import 'package:in_app_purchase/in_app_purchase.dart';

// iOS支付单一实例
final iOSPayment = IOSPayment();

class IOSPayment {
  /// 单例模式
  static final IOSPayment _iosPayment = IOSPayment.init();

  factory IOSPayment() {
    return _iosPayment;
  }

  IOSPayment.init();

  // 应用内支付实例
  InAppPurchaseConnection purchaseConnection = InAppPurchaseConnection.instance;
  FlutterSecureStorage storage = new FlutterSecureStorage();

  // iOS订阅监听
  StreamSubscription<List<PurchaseDetails>> subscription;

  /// 判断是否可以使用支付
  Future<bool> isAvailable() async => await purchaseConnection.isAvailable();

  // 开始订阅
  void startSubscription() async {
    if (subscription != null) return;
    print('>>> start subscription');
    // 支付消息订阅
    Stream purchaseUpdates = purchaseConnection.purchaseUpdatedStream;
    subscription = purchaseUpdates.listen(
      (purchaseDetailsList) {
        purchaseDetailsList.forEach((PurchaseDetails purchaseDetails) async {
          if (purchaseDetails.status == PurchaseStatus.pending) {
            print('>>> pending');
            // 业务代码略：有订单开始支付，向外部发出通知，并记录到缓存中；  
          } else {
            if (purchaseDetails.status == PurchaseStatus.error) {
              print('>>> error');
               // 业务代码略：有订单支付错误，向外部发出通知
              // 下面是删除
              String value = await storage.read(key: purchaseDetails.productID);
              String orderId = value.split('￥')[0];
              writeStorage(purchaseDetails.productID, orderId, 'cancel');
              finalTransaction(purchaseDetails);
            } else if (purchaseDetails.status == PurchaseStatus.purchased) {
              print('>>> purchased');
              String orderId = purchaseDetails.skPaymentTransaction.payment.applicationUsername;
              if (orderId == null || orderId.isEmpty) {
                // 如果applicationUsername为空，执行凑单
                orderId = await foundRecentOrder(purchaseDetails.productID);
              }
              if (orderId.isEmpty) {
                  // 凑单失败，找不到业务单号，结束
                  finalTransaction(purchaseDetails);
                  BlocProvider.of<PaymentUtilsBloc>(Application.navigatorState.currentContext).add(IosPayFailureEvent(errorMessage: '支付出错啦，请稍后再试~'));
                  return;
                }
                // 业务代码略：支付成功，向外部发出通知
                // 业务代码略：开始核验订单，核验结果由外部监听
                );
            }
          }
        });
      },
      onDone: () {
        stopListen();
      },
      onError: (error) {
        stopListen();
      },
    );
  }

  /// 检查sku是否有对应商品
  Future<bool> checkProductBySku(String sku, {Function(String err) onError}) async {
    if (!await isAvailable()) {
      onError?.call('无法连接AppStore，请稍后再试');
      return false;
    }
    ProductDetailsResponse appStoreProducts = await purchaseConnection.queryProductDetails([sku].toSet());
    if (appStoreProducts.productDetails.length == 0) {
      onError?.call('没有找到相关产品，请联系管理员');
      return false;
    }
    return true;
  }

  /// 启动支付
  void iosPay(String sku, String orderId, {Function(String err) onError}) async {
    // 获取商品列表
    ProductDetailsResponse appStoreProducts = await purchaseConnection.queryProductDetails([sku].toSet());
    // 发起支付
    purchaseConnection
        .buyNonConsumable(
      purchaseParam: PurchaseParam(
        productDetails: appStoreProducts.productDetails.first,
        applicationUserName: orderId,
      ),
    )
        .then((value) {
      if (value) {
        // 只要能发起，就写入
        writeStorage(sku, orderId, 'pending');
      }
    }).catchError((err) {
      onError?.call('当前商品您有未完成的交易，请等待iOS系统核验后再次发起购买。');
      print(err);
    });
  }

  writeStorage(String key, String value, String status) {
    storage.write(key: key, value: '$value￥$status');
  }

  // 关闭交易
  void finalTransaction(PurchaseDetails purchaseDetails) async {
    await purchaseConnection.completePurchase(purchaseDetails);
    // 每完成一张订单进行缓存的清除
    if (!await checkStorage()) {
      stopListen();
    }
  }

  // 凑单机制
  Future<String> foundRecentOrder(String sku) async {
    String orderId = '';
    String values = await storage.read(key: sku);

    if (values != null) {
      orderId = values.split('￥')[0];
    }
    return orderId;
  }

  // 校验是否还有缓存
  Future<bool> checkStorage() async {
    Map<String, String> remainingValues = await storage.readAll();
    return remainingValues.isNotEmpty;
  }

  // 关闭监听
  stopListen() async {
    subscription?.cancel();
    subscription = null;
  }
}


```

页面调用时，建议启用定时器，**因为 iOS 回调不稳定，所以监听到应用回到前台时开始 30 秒计时；30 秒内没有收到支付回调，需要做对应提示，这一块也是存业务流程，我这里不做代码展示。** 下面代码是如何调用上面工具类的：

```
iOSPayment.startSubscription();
iOSPayment.iosPay(
    state.skuId,
    state.model.orderId,
    onError: (String err) {
      if (!mounted) return;
      // 支付遇到错误，马上停止定时器，并且关掉弹框
    },
 );


```

```
// 应用启动时
if (Platform.isIOS && await iOSPayment.checkStorage()) {
    // 启动订阅：支付缓存未清除完毕、机型可使用应用内支付
    iOSPayment.startSubscription(needDelayed: true);
  }


```

测试 IAP 中断购买的测试
--------------

*   这个测试是模拟用户点击购买协议的操作，当弹出系统协议弹框时，iOS 会发出一个支付错误的消息；这个时候我们的**代码会 final 这个支付，并且将持久化中对应 skuId 的信息状态改为 cancel；**
*   然后用户同意后，iOS 会再发起一个同样的不带 OdrerId(是的，被弄丢了。。。。) 的订单，用户支付成功后，我们的代码就会收到支付成功的没有 OdrerId 的推送，在持久化存储中执行凑单机制后，再发给后台核销。 如何模拟这个流程呢？看看[官方文档描述](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.apple.com%2Fdocumentation%2Fstorekit%2Fin-app_purchase%2Ftesting_in-app_purchases_with_sandbox%3Flanguage%3Dobjc "https://developer.apple.com/documentation/storekit/in-app_purchase/testing_in-app_purchases_with_sandbox?language=objc"), 下面是译文:

```
#### 设置测试

通过[登录App Store Connect](https://help.apple.com/app-store-connect/#/devcd5016d31)启用对Sandbox Apple ID的中断购买，然后：

1.  在“用户和访问”中，单击边栏中沙箱下的“测试器”。在右侧，您可以查看您的Sandbox Apple ID。

2.  选择您要为其启用中断购买的Sandbox Apple ID。如果已启用，则会在“中断购买”列下看到一个复选标记。

3.  在出现的对话框中，选择“此测试仪的中断购买”。

#### 开始测试

1.  在测试设备上，使用已中断购买的沙盒Apple ID登录。
2.  在您的应用中，选择“购买”或“订阅”进行应用内购买。
3.  观察到系统显示付款单。
4.  在您的代码中，验证付款队列在状态下是否收到新交易。
5.  在设备上，验证付款单。
6.  在您的代码中，观察到付款失败。付款队列在状态中接收更新的交易。
7.  检查您的代码调用是否将其从队列中删除。
8.  在设备上，观察到系统显示“条款和条件”，从而中断了购买（因为您已配置了沙盒环境）。
9.  在设备上，点击以同意条款和条件。
10.  在您的代码中，验证付款队列接收到的新交易处于与失败交易相同且数量相同的状态.
11.  在您的代码中，验证收据。检查您的应用是否提供了服务或产品，然后致电。
12.  在设备上，用户应观察到购买成功。


```

也就是说在 Apple 后台把沙盒测试账号设置为中断即可。但是无论我怎么同意，收到的还是支付失败的订阅。其实是因为文档写漏了，**中断后 app 弹出同意协议弹框，也就是上面第 8 步，这个时候必须在后台把中断测试关了，然后再执行第九步。**（就是这么狗血，官方文档不给力，网上也没有任何资料，最后还是在官方论坛，看到某个 QA 的评论才找到的灵感。。。这里也感谢公司大佬花了半天专门找这方面的资料。） ## 写在最后 感谢大家孜孜不倦看到最后，这篇长文希望能帮助开发支付的小伙伴少踩一些坑。 IAP 的支付确实是很坑，但如果站在 iOS 开发者的角度来看。其实也能理解：**他们是做手机系统的，他们能保证系统内部的所有支付流程，根本不 care 开发者的业务逻辑。**

但无论如何，这种方式对于开发者，确实是极度不友善的；另外，还有一种流程，app 发起支付后，只要有回调就马上 final，成功就发给后台，由后台去执行凑单机制，这种对于前端其实更合理，毕竟数据存在客户端永远是不够安全，但是这样 app 就有可能对同一个 skuId 疯狂发起购买，后台凑单时，就做不到一一对应。有利有弊吧~~~

**小弟班门弄斧，希望能一起学习进步！！！** ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/44da156428e948b284b63719cb87e02b~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)