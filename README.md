###简介
统一接口，包括（微信支付，支付宝支付，银联支付） 只需要一行代码即可，让支付变得简单。

---

###集成
####使用CocoaPods安装
1. 在 Podfile 添加
    
    ```
    platform :ios, '8.0'
    use_frameworks!
    
    pod 'CGYPay'
    ```
    
    你可以选择添加需要的支付功能，默认情况下，是不自动添加支付功能的，因此必须选择一下任意一种（或多种）支付功能，目前仅支持 `AliPay`，`WxPay`，`UnionPay`
    例如：
    
    ```
    pod 'CGYPay/AliPay'
    pod 'CGYPay/WxPay'
    pod 'CGYPay/UnionPay'
    ```
    
2. 然后运行以下命令
    
    ```
    $ pod install
    ```
3. 这时候你应该从`{projectName}.xcworkspace`文件打开工程，而不是`{projectName}.xcodeproj`


####手动导入（不建议使用手动导入，强烈建议使用CocoaPods）

***[注意] 已自带SDK, 可以免去1.1, 2.2步骤, 有强迫症的可以自行下载.***

1. 下载SDK
    * [微信SDK下载](https://pay.weixin.qq.com/wiki/doc/api/app/app.php?chapter=11_1)
    * [支付宝SDK下载](https://doc.open.alipay.com/doc2/detail?treeId=59&articleId=103563&docType=1)
    * [银联SDK下载](https://open.unionpay.com/ajweb/help/file/toDetailPage?id=346&flag=2)

2. 导入SDK
    SDK主要包含的文件
    
    | 微信          | 支付宝            | 银联支付          |
    |:---:          |:---:              |:---:              |
    |libWeChatSDK.a |AlipaySDK.framework|libPaymentControl.a|
    |WechatAuthSDK.h|                   |UPPaymentControl.h |
    |WXApi.h        |                   |                   |
    |WXApiObject.h  |                   |                   |

3. 导入依赖库

    |库名|微信/支付宝|
    |:---|:---|
    |libc++.tbd|微信, 支付宝|
    |libz.tbd|微信, 支付宝, 银联支付|
    |CoreTelephony.framework|微信, 支付宝|
    |libsqlite3.0.tbd|微信|
    |SystemConfiguration.framework|微信, 银联支付|
    |CoreMotion.framework|支付宝|
    |CoreGraphics.framework|支付宝|
    |CoreText.framework|支付宝|
    |QuartzCore.framework|支付宝|
    |CFNetwork.framework|支付宝, 银联支付|

4. 创建桥接文件`ProjectName-Bridging-Header.h`
    配置桥接文件的路径
    ![](https://raw.githubusercontent.com/Chakery/images/master/CGYPay/%E6%A1%A5%E6%8E%A5%E6%96%87%E4%BB%B6%E8%B7%AF%E5%BE%84.png)
    桥接文件内容如下:

    ```Swift
    #import <UIKit/UIKit.h>
    #import <Foundation/Foundation.h>
    // 微信支付
    #import "WXApi.h"
    #import "WXApiObject.h"
    // 支付宝支付
    #import <AlipaySDK/AlipaySDK.h>
    // 银联支付
    #import "UPPaymentControl.h"
    ```

5 配置URL Schemes
`URL Schemes`建议使用appid, 或者使用Bundle identifier
![这图片要改](https://raw.githubusercontent.com/Chakery/images/master/CGYPay/%E9%85%8D%E7%BD%AEurl.png)

####1.6 适配iOS 9.0及以上
为了适配iOS 9(及以上)的网络请求, 可以给微信, 支付宝等添加白名单, 也可以直接允许所有http请求, 这里为了方便, 直接允许http请求. 在info.plist文件中添加:
```Swift
<key>NSAppTransportSecurity</key>
<dict>
<key>NSAllowsArbitraryLoads</key>
<true/>
</dict>
```
给URL Schemes添加白名单:
```Swift
<key>LSApplicationQueriesSchemes</key>
 <array>
    <string>alipayauth</string>
	<string>alipay</string>
	<string>alipayshare</string>
	<string>safepay</string>
	<string>weixin</string>
	<string>wechat</string>
	<string>uppaysdk</string>
	<string>uppaywallet</string>
	<string>uppayx1</string>
	<string>uppayx2</string>
	<string>uppayx3</string>
 </array>
```
这里的白名单并不完整, 只是其中一部分, 因此在使用过程中, 可能会发出警告.

---

#2. 使用

#### 2.1 微信支付
关于微信支付的参数, 请参考[官方说明](https://pay.weixin.qq.com/wiki/doc/api/app/app.php?chapter=9_12&index=2).
在AppDelegate中的application: didFinishLaunchingWithOptions: 方法中注册微信.
```Swift
func application(application: UIApplication, didFinishLaunchingWithOptions launchOptions: [NSObject: AnyObject]?) -> Bool {
        
        WXApi.registerApp(appid)
        
        return true
    }
```
调起微信支付.
```Swift
let channel = CGYPayChannel.weixin(
                                 partnerId: "商家id",
                                 prepayid: "订单id",
                                 nonceStr: "随机字符串,防止重发",
                                 timeStamp: 时间戳,防止重发(例: 1459014554),
                                 package: "扩展字段(暂填写固定值Sign=WXpay)",
                                 sign: "签名")

CGYPay.createPayment(channel) { (status) in
    switch status {
    case .PaySuccess(let wxPayResult, _, _):
        print("支付成功: \(wxPayResult)")
    default:
        print("支付失败")
    }
}
```

#### 2.2 支付宝支付
关于`orderString`参数, 请参考[官方说明](https://doc.open.alipay.com/doc2/detail?treeId=59&articleId=103663&docType=1), `appScheme`参数对应URL types里面的URL scheme, 建议使用appid.
```Swift
let orderStr = "partner=\"2088101568358171\"&seller_id=\"xxx@alipay.com\"&out_trade_no=\"0819145412-6177\"&subject=\"我是测试数据标题\"&body=\"我时测试数据商品描述\"&total_fee=\"0.01\"&notify_url=\"http://www.xxx.com\"&service=\"mobile.securitypay.pay\"&payment_type=\"1\"&_input_charset=\"utf-8\"&it_b_pay=\"30m\"&show_url=\"m.alipay.com\"&sign=\"lBBK%2F0w5LOajrMrji7DUgEqNjIhQbidR13GovA5r3TgIbNqv231yC1NksLdw%2Ba3JnfHXoXuet6XNNHtn7VE%2BeCoRO1O%2BR1KugLrQEZMtG5jmJIe2pbjm%2F3kb%2FuGkpG%2BwYQYI51%2BhA3YBbvZHVQBYveBqK%2Bh8mUyb7GM1HxWs9k4%3D\"&sign_type=\"RSA\""
        
let channel = CGYPayChannel.aliPay(orderString: orderStr, appScheme: appid)
        
CGYPay.createPayment(channel) { (status) in
    switch status {
    case .PaySuccess(_, let aliPayResult, _):
        print("支付成功: \(aliPayResult)")
    default:
        print("支付失败")
    }
}
```

####2.3 银联支付
```Swift
// tn:          订单号
// appScheme:   参数对应URL types里面的URL scheme
// mode:        接入模式, "00"生产模式, "01"开发测试模式
let channel = CGYPayChannel.upPay(
                                tn: "201603282300181104808", 
                                appScheme: "com.ccggyy.cgypay", 
                                mode: "01")
        
CGYPay.createPayment(channel) { status in
    switch status {
    case .PaySuccess(_, _, let upPayResult):
        print("银联支付成功: \(upPayResult)")
    default:
        print("银联支付失败")
    }
}
// 支付成功后, 返回一个字典对象 --- upPayResult
// 格式 { "sign":"" , "data":"" }
// sign = 签名后做Base64的数据
// data = "pay_result=支付结果(success，fail，cancel)&tn=订单号&cert_id=证书id"
```

