# AppflowSDK
Platform：iOS
Version：v1.0.0

## 1. SDK integration
##### AppflowSDK provides two integration methods for iOS developers to choose:
- Pass through CocoaPods integrated
- Pass through Swift package manager integrated
### Pass through CocoaPods integrated

```
target 'MyApp' do
    use_frameworks!
    pod 'AppflowSDK', '~> 1.0.0'
end
```
Save and execute pod install, then open the project with a file suffixed with .xcworkspace.
```
pod install
```
> Notice:
> Execute pod search AppflowSDK under the command line. If the displayed version of **AppflowSDK** is not the latest, execute the pod repo update operation to update the content of the local repo
> For more information on CocoaPods please see [ CocoaPods](https://cocoapods.org/)。

### Pass through Swift package manager integrated
To add the SDK using Swift's package manager:
1. Click File.
1. Select Swift Packages.
1. Select Add Package Dependency.
1. In the box that appears, enter the SDK's GitHub address.
1. Select the version of the AppflowSDK you want to use in the Version dropdown. Check the releases page for the latest stable version.
```
https://github.com/appflow-ai/AppflowSDK
```


## 2. Initialize SDK
### Add configuration file

> Download the **appflow-app-token.json** file, Add to targets to the project. 
> Select Copy Items if needed.
> Create groups.
> Add to targets.

### import header file
Import the header file in the project's AppDelegate file
```
import AppflowSDK
```
## Setup
```
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        //AppflowSDK initialization
        Appflow.shared.configure()
        return true
}
```
### Enable SDK log (optional)
#### You can enable SDK log output before initializing the SDK

```
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
    // Override point for customization after application launch.
    
    //AppflowSDK initialization
    Appflow.shared.configure()
    //Whether to enable in-app purchase related logs, the default is not
    Appflow.shared.setPurchasesLogs(enabled: true)
    //Whether to enable the log related to the buried point, the default is not
    Appflow.shared.setAnalyticsLogs(enabled: true)
    return true
}
```


## 3. Purchases
> Ready to work
> a. Configure product information in apple store
> 
> b. Add product information in appflow platform

### Get product id of in-app purchase

```
 Appflow.shared.getPurchaseProductIds { productIdArray, error in 
 
 }
```
### Get the sku information of in-app purchases
Displaying Products
To fetch the products sku information, you have to call method:

According to the interface `getSkuDetails`, you need to pass in the Set <product_Ids> of in-app purchases, the return value is a dictionary, `product_id` is key, value: `SKProduct` object
```
Appflow.shared.getSkuDetails(productIds: Set(productIDs)) { skuDetailInfo, error in
    
};
```
After obtaining `SKProduct`, you can obtain information related to in-app purchases, and developers can save them in their own projects. Subsequent payment needs to be used.

### Check if payment available
```
 let canMakePurchases = Appflow.shared.canMakePayment()
```
### Making purchase
To start the purchase process call function purchaseProduct it will take `SKProduct` object as a parameter.
```
Appflow.shared.purchaseSKProduct(product) { (transaction, entitlement, error, isCanceled) in {
}
```
Closure will return `SKPaymentTransaction`, entitlement dictionary, error if it occurred and bool to indicate if a user canceled the purchase process.

### Checking user status
To check if user have any active entitlements call function `hasActiveSubscription`
```
Appflow.shared.hasActiveSubscription({ [weak self] entitlements, error in
    guard let `self` = self else { return }
    var status = "pro not active"
    if error == nil {
        if entitlements.count > 0 , let entilement:IMEntitlement = entitlements.values.first {
            if entilement.isActive() {
                status = "pro active"
            }
        }
        self.statusLabel.text = "Subcription status: \(status)"
    }
})

```
It will return the entitlements dictionary where key entitlement id and value `IMEntitlment` object. 
To check if entilement active call `isActive()` method of `IMEntitlement` object.

### Restoring purchases
To restore user purchases call restorePurchases function.

```
Appflow.shared.restorePurchases { (entitlement, error) in
    if error == nil {
        if entitlements.count > 0 , let entilement:IMEntitlement = entitlements.values.first {
            if entilement.isActive() {
                // Purchase restored and have active entitlment
            }else {
                // Purchase restore finished but user don't have active entitlment
            }
        }else {
            // Purchase restore finished but user don't have active entitlment
        }
    }
}
```

### UploadUserInfo

To UploadUserInfo call uploadUserInfo function. This method needs to pass in the parameter userId
> NOTE: Adds an interface uploadUserInfo, which is used to upload subscribed user data. Currently, this data is used for iap attribution. When in-app purchases are completed, uploadUserInfo, userId needs to be called (optional)

```
Appflow.shared.uploadUserInfo(userId: "xxxx") { (result, error) in
    
}
//
```
#### expand
##### Add a json string field in the UploadUserInfo API to support developers uploading map<string, string> type data. It supports fixed fields: username, email, phone, gender, age.
> - Field:
> - All key , value length limit is 128 , calculated according to toString characters
> - The fixed fields are username, email, phone, gender, age, please refer to the table below, gender is a number, and the meaning is shown in the table
> - There are no restrictions on the type of additional fields added by the user, both numbers and strings can be used
> - If you need to report user-related data, report it through the following methods

field | type |
---|---
username | string
email | string
phone | string
gender | number(FEMALE = 1, MALE = 2, OTHER = 3)
age | number



```
//uploadUserInfo to extraAttribute
Appflow.shared.uploadUserInfo(userId: "app_user_idxxxxx", extraAttribute: extraAttribute) {[weak self] _, _ in
}
```


## 4. Event tracking
### First installation
```
Appflow.shared.analytics.uploadBigDataWithType(eventType: .firstOpen, params: nil)
```
### The App launched
```
Appflow.shared.analytics.uploadBigDataWithType(eventType: .appStart, params: nil)
```
### The App current uv
```
Appflow.shared.analytics.uploadBigDataWithType(eventType: .screenView, params: [
    kScreenViewName: kHomepage
])
or
let model = ParamModel()
model.screenType = ParamScreenHome
Appflow.shared.analytics.uploadBigDataWithType(eventType: .screenView, paramModel: model)
```
### Ads click
```
Appflow.shared.analytics.uploadBigDataWithType(eventType: .adsClick, params: [
    kPlacement: kQuickClean,
    kFormat: kBanner
])
or 
let model = ParamModel()
model.placeType = ParamPlaceClean
model.formatType = ParamFormatBanner
Appflow.shared.analytics.uploadBigDataWithType(eventType: .adsClick, paramModel: model)
```
### Ads show
```
Appflow.shared.analytics.uploadBigDataWithType(eventType: .adsShow, params: [
    kPlacement: kQuickClean,
    kFormat: kBanner
])

let model = ParamModel()
model.placeType = ParamPlaceClean
model.formatType = ParamFormatBanner
Appflow.shared.analytics.uploadBigDataWithType(eventType: .adsShow, paramModel: model)
```
### ev_User_Engagement
```
let tMSimestamp = "\(IMDeviceInfoManager.shared().getCurrentMSimestamp())"
Appflow.shared.analytics.uploadBigDataWithType(eventType: .userEngagement, params: [
    kEngagementTimeMsec: tMSimestamp,
    kScreenViewName: kHomepage
])
or
let tMSimestamp = "\(IMDeviceInfoManager.shared().getCurrentMSimestamp())"
let model = ParamModel()
model.screenType = ParamScreenHome
model.otherData = [
    kEngagementTimeMsec: tMSimestamp
]
Appflow.shared.analytics.uploadBigData(eventType: .userEngagement, paramModel: model)
```
### If you need a callback in the above method, do something like this

```
Appflow.shared.analytics.uploadBigDataWithType(eventType: .userEngagement, params: [
    kEngagementTimeMsec: tMSimestamp,
    kScreenViewName: kHomepage
], successBlock: { obj in

}) { obj in

}
```
### If the above enumeration does not meet the requirements, extend it as follows
```
Appflow.shared.analytics.uploadBigDataWithEvent(
    eventName: "event_name",
    params: [
        "name": "123",
        "time": "456"
    ])
```


## 5. Push notification
### a. Capabilities
Please enable the Capabilities->Push Notifications option of Application Target
### b. Request PushNotifications Permissions in AppDelegate

```
private func requestPushNotificationsPermissions() {
    let userNotificationCenter = UNUserNotificationCenter.current()
    userNotificationCenter.requestAuthorization(options: [.alert, .sound, .badge]) { [weak self] granted, error in
        print("Permission granted: \(granted)")
        if granted {
            self?.getNotificationSettings()
        }
    }
}
```
### c. Permission request completed, get notification settings

```
private func getNotificationSettings() {
    UNUserNotificationCenter.current().getNotificationSettings { settings in
        print("Notification settings: \(settings)")
        guard settings.authorizationStatus == .authorized else { return }
        DispatchQueue.main.async {
            UIApplication.shared.registerForRemoteNotifications()
        }
    }
}
```
### d. Successfully register APNs and report DeviceToken
```
func application(_ application: UIApplication, didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
    let deviceToken = deviceToken.reduce("", { $0 + String(format: "%02X", $1) })
    Appflow.shared.purchases.uploadDeviceInfo(deviceToken: deviceToken) { _, _ in
    }
}
```

## 5. Paywall
The SDK provides a shortcut for displaying paid products. After setting the style in Appflow plaltform, you can display it by call method:

```
Appflow.shared.loadPaywallToPurachase {[weak self] result, msg in
    guard let `self` = self else { return }
    if result == false {
        //if false , Developers can jump to the native payment page
    }
}
```



## 5. Attribution
Appflow supports Appsflyer, Adjust, Branch, Apple search ads, FacebookAds and custom attribution data upload，you have to call method:
### Appsflyer
To upload Appsflyer attribution data, the developer accesses the appsflyer SDK and reports the data in the AppsFlyerLibDelegate method, refer to the example below,which **networkUserId **cannot be empty

```
extension AppDelegate: AppsFlyerLibDelegate {
    func onConversionDataFail(_ error: Error) {
        //appsflyer is fail
    }
    
    func onConversionDataSuccess(_ installData: [AnyHashable : Any]) {
        // It's important to include the network user ID
        Appflow.shared.updateAttribution(installData, source: .appsflyer, networkUserId: AppsFlyerLib.shared().getAppsFlyerUID())
        
    }
}
```
### Adjust
To upload Adjust attribution data, the developer accesses the Adjust SDK and reports the data in the AdjustDelegate method, refer to the example below

```
extension AppDelegate: AdjustDelegate {
    //Callback for the first app installation
    func adjustAttributionChanged(_ attribution: ADJAttribution?) {
        // Just pass Adjust attribution to Adapty SDK
        if let attribution = attribution?.dictionary() {
            Appflow.shared.updateAttribution(attribution, source: .adjust)
        }
    }
    
}
```

### Branch
To upload Adjust attribution data, the developer accesses the Adjust SDK and reports the data in the andRegisterDeepLinkHandler block, refer to the example below

```
//  MARK: branch init
func branchInit(launchOptions: [UIApplication.LaunchOptionsKey: Any]?) {
    Branch.getInstance().setIdentity("YOUR_USER_ID")
    Branch.getInstance().initSession(launchOptions: launchOptions) { (data, error) in
        if let data = data {
                Appflow.shared.updateAttribution(data, source: .branch)
        }
    }

}
```
### FacebookAds
To upload FacebookAds attribution data, refer to the following example, which **networkUserId **cannot be empty

```
Appflow.shared.updateAttribution([:], source: .facebook,networkUserId:FBSDKCoreKit.AppEvents.shared.anonymousID)
```


### Apple Search ads
To upload Apple Search ads attribution data, refer to the following example,

```
Appflow.shared.updateAttribution(attribution, source: .appleSearchAds)
```

### Custom
To upload custom attribution data, refer to the following example,The framework AdServices and iAd are required for use, please refer to Apple's documentation for details.[setting_up_apple_search_ads_attribution](https://developer.apple.com/documentation/iad/setting_up_apple_search_ads_attribution)

```
let attribution = [
    "status" : "non_organic",
    "channel" : "Google Ads",
    "campaign" : "Christmas Sale"
]
Appflow.shared.updateAttribution(attribution, source: .custom)
```

## 5. WelcomPage
The SDK adds the View of AppflowWelcomePage. If the user needs to start the welcome page, the configuration of the page needs to be configured on the Appflow platform. The welcomePage loading method can be referred to as follows:

```
// loading WelcomPage
func loadLaunchScreenView() {
    // customized launch screen
    if let window = self.window {
        window.backgroundColor = .white
        let welcomPageView = AppflowWelcomePage(frame: window.bounds)
        welcomPageView.waitingTime = 6
        self.window?.makeKeyAndVisible()
        self.window?.addSubview(welcomPageView)
        self.window?.bringSubviewToFront(welcomPageView)
    }
}

```
> It may take time to load welcome page for the first time. The default loading waiting time is 30s. Users can set **waitingTime** according to their needs.
