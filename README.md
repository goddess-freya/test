
# Appflow SDK Documentation

Platform：Android

Version：v1.0.4



### 1、Install SDK

Install via Gradle

a、If your project doesn't have `dependencyResolutionManagement` in your `settings.gradle`, add the following to your top-level `build.gradle` at the end of repositories：

**top-level build.gradle**

```java
allprojects {
    repositories {
        ...
        maven { url 'https://maven.appflow.ai/repository/sdk' }
    }
}
```

Otherwise add the following to your `settings.gradle` in `repositories` of `dependencyResolutionManagement` section：

**settings.gradle**

```java
dependencyResolutionManagement {
    ...
    repositories {
        ...
        maven { url 'https://maven.appflow.ai/repository/sdk' }
    }
}
```



b、add the dependency to your module-level `build.gradle` at the end of dependencies：

**module-level build.gradle**

```java
dependencies {
    ...
    implementation 'com.imp:appflow:1.0.4'
}
```



### 2、Initialize SDK

##### Add configuration file

Download the `appflow-app-token.json`  file in the [dash.appflow.ai](https://dash.appflow.ai) and add the file to the `app\src\main\assets` folder of the project



##### Initialize

Add the following to your Application class

```kotlin
override fun onCreate() {
    super.onCreate()
    Appflow.init(this)
}
```

If you want to know if the SDK is initialized, you can do the following

```kotlin
override fun onCreate() {
    super.onCreate()
    Appflow.init(this, AppflowConfig.Builder().build()) {
			//AppflowSDK initialization finished，do something
    }
}
```



##### Set user id (optional)

You can set the user id in the following way

```kotlin
override fun onCreate() {
    super.onCreate()
    Appflow.init(this, AppflowConfig.Builder().setAppUserID("your_user_id").build())
}
```



##### Enable SDK log (optional)

You can enable SDK log output before initializing the SDK

```kotlin
override fun onCreate() {
    super.onCreate()
    Appflow.setLogEnable(true)
    Appflow.init(this)
}
```



### 3、Event

You can send statistics events to Appflow backend in the following ways

```kotlin
//Send an event with no parameters
Appflow.sendEvent(eventName: String) 
//Send an event with parameters
Appflow.sendEvent(eventName: String, vararg params: EventObject)
//Send an event with parameters
Appflow.sendEvent(eventName: String, params: HashMap<String, Any>)
```

**Example**

```kotlin
//Send an event with no parameters
Appflow.sendEvent("test_event")
//Send an event with parameters
Appflow.sendEvent("test_event", EventObject("name", "abc"))
Appflow.sendEvent("test_event", EventObject("name", "abc"), EventObject("age", 18))
//Send an event with parameters
val params = HashMap<String, Any>()
params["name"] = "abc"
params["age"] = 18
Appflow.sendEvent("test_event", params)
```

**API Reference**

| EventObject  |                       |
| :----------- | --------------------- |
| key : String | event parameter name  |
| value : Any  | event parameter value |



### 4、Purchases

##### Ready to work

a、Configure product information in Google Play

b、Add product information in [dash.appflow.ai](https://dash.appflow.ai)



##### Displaying Products

To fetch the products, you have to call method:

```kotlin
/**
 * parameter1：skus，The product id list
 */
Appflow.getSkuDetails(skus: List<String>, listener: SkuDetailsListener)
```

**Example**

```kotlin
val skus = listOf("SKU_WEEKLY", "SKU_MONTHLY", "SKU_YEARLY")
Appflow.getSkuDetails(skus, object : SkuDetailsListener {
    override fun onError(error: PurchasesError) {}
    override fun onReceived(map: HashMap<String, SkuDetails>) {
        for (product in map) {
            val skuDetails = product.value
            Log.d(TAG,"sku:${skuDetails.sku}")
            Log.d(TAG,"title:${skuDetails.title}")
            Log.d(TAG,"iconUrl:${skuDetails.iconUrl}")
            Log.d(TAG,"description:${skuDetails.description}")
            Log.d(TAG,"introductoryPrice:${skuDetails.introductoryPrice}")
        }
    }
})
```

**API Reference**

| SkuDetailsListener                           |                  |
| -------------------------------------------- | ---------------- |
| onReceived(map: HashMap<String, SkuDetails>) | Success callback |
| onError(error : PurchasesError)              | Failure callback |

| PurchasesError |               |
| -------------- | ------------- |
| code           | error code    |
| message        | error message |

| SkuDetails                                |                                                              |
| ----------------------------------------- | ------------------------------------------------------------ |
| getDescription() : String                 | Returns the description of the product                       |
| getFreeTrialPeriod() : String             | Trial period configured in Google Play Console, specified in ISO 8601 format |
| getIconUrl() : String                     | Returns the icon of the product if present                   |
| getIntroductoryPrice() : String           | Formatted introductory price of a subscription, including its currency sign, such as €3.99 |
| getIntroductoryPriceAmountMicros() : Long | Introductory price in micro-units                            |
| getIntroductoryPriceCycles() : Int        | The number of subscription billing periods for which the user will be given the introductory price, such as 3 |
| getIntroductoryPricePeriod() : String     | The billing period of the introductory price, specified in ISO 8601 format |
| getOriginalJson() : String                | Returns a String in JSON format that contains SKU details    |
| getOriginalPrice() : String               | Returns formatted original price of the item, including its currency sign |
| getOriginalPriceAmountMicros() : Long     | Returns the original price in micro-units, where 1,000,000 micro-units equal one unit of the currency |
| getPrice() : String                       | Returns formatted price of the item, including its currency sign |
| getPriceAmountMicros() : Long             | Returns price in micro-units, where 1,000,000 micro-units equal one unit of the currency |
| getPriceCurrencyCode() : String           | Returns ISO 4217 currency code for price and original price  |
| getSku() : String                         | Returns the product Id                                       |
| getSubscriptionPeriod() : String          | Subscription period, specified in ISO 8601 format            |
| getTitle() : String                       | Returns the title of the product being sold                  |
| getType() : String                        | Returns the PurchaseType of the SKU.                         |

| PurchaseType |                                                |
| ------------ | ---------------------------------------------- |
| INAPP        | A type of SKU for Android apps in-app products |
| SUBS         | A type of SKU for Android apps subscriptions   |



##### Making Purchases

Making Purchases, you have to call method:

```kotlin
/**
 * parameter1：activity，The context must be of type Activity
 * parameter2：skuDetails，The SkuDetails object corresponding to the product
 */
Appflow.purchasePackage(activity: Activity, skuDetails: SkuDetails, listener: MakePurchaseListener)
```

**Example**

```kotlin
Appflow.purchasePackage(activity, skuDetails, object : MakePurchaseListener{
    override fun onCompleted(purchase: Purchase) {
        //After the purchase is successful, you can obtain the order id,
        //order token and other information through the Purchase object
    }

    override fun onError(error: PurchasesError, userCancelled: Boolean) {
        //When userCancelled is true, it means that the user cancels the purchase; 
        //when userCancelled is true, it means that the purchase fails
        //You can get specific failure information through PurchasesError
    }
})
```

**API Reference**

| MakePurchaseListener                                       |                  |
| ---------------------------------------------------------- | ---------------- |
| onCompleted(purchase : Purchase)                           | Success callback |
| onError(error : PurchasesError ,  userCancelled : Boolean) | Failure callback |

| Purchase                                     |                                                              |
| -------------------------------------------- | ------------------------------------------------------------ |
| getAccountIdentifiers() : AccountIdentifiers | Returns account identifiers that were provided when the purchase was made |
| getDeveloperPayload() : String               | Returns the payload specified when the purchase was acknowledged or consumed |
| getOrderId() : String                        | Returns a unique order identifier for the transaction        |
| getOriginalJson() : String                   | Returns a String in JSON format that contains details about the purchase order |
| getPackageName() : String                    | Returns the application package from which the purchase originated |
| getPurchaseTime() : Long                     | Returns the time the product was purchased, in milliseconds since the epoch (Jan 1, 1970) |
| getPurchaseToken() : String                  | Returns a token that uniquely identifies a purchase for a given item and user pair |
| getSignature() : String                      | Returns String containing the signature of the purchase data that was signed with the private key of the developer |
| isAutoRenewing() : Boolean                   | Indicates whether the subscription renews automatically      |
| isAcknowledged() : Boolean                   | Indicates whether the purchase has been acknowledged         |
| getSku() : String                            | Returns the product Id                                       |



##### Subscription Status

Get the subscription status of a product, you have to call method:

```kotlin
Appflow.getSubscriberInfo(listener: ReceivePurchaserInfoListener)
```

**Example**

```kotlin
Appflow.getSubscriberInfo(object : ReceivePurchaserInfoListener {
    override fun onReceived(subscriber: IapIap.Subscriber) {
        for (info in subscriber.entitlementsList) {
            Log.d(TAG,"sku:${info.productId}")
            Log.d(TAG,"isActive:${info.isActive}")
        }
    }

    override fun onError(error: PurchasesError) {}
})
```

**API Reference**

| ReceivePurchaserInfoListener              |                  |
| ----------------------------------------- | ---------------- |
| onReceived(subscriber: IapIap.Subscriber) | Success callback |
| onError(error: PurchasesError)            | Failure callback |

| IapIap.Subscriber                                    |                                  |
| ---------------------------------------------------- | -------------------------------- |
| getEntitlementsList() : List<iap.IapIap.Entitlement> | Returns product information list |

| iap.IapIap.Entitlement  |                                                              |
| ----------------------- | ------------------------------------------------------------ |
| getId() : String        | Returns product group id                                     |
| getProductId() : String | Returns the product Id                                       |
| getIsActive() : Boolean | Return product subscription\purchase status, true: subscribed\purchased; false: unsubscribed\unpurchased |
| getExpireAt() : Long    | Returns the subscription expiration time for the product     |



##### Upgrade/Downgrade product

To upgrade/downgrade a product, you have to call method:

```kotlin
/**
 * parameter1：activity，The context must be of type Activity
 * parameter2：skuDetails，The SkuDetails object corresponding to the product
 * parameter3：upgradeInfo，Upgrade/Downgrade Product Information Object
 */
Appflow.purchasePackage(
        activity: Activity,
        packageToPurchase: SkuDetails,
        upgradeInfo: UpgradeInfo,
        listener: MakePurchaseListener
    )
```

**Example**

```kotlin
val upgradeInfo = UpgradeInfo(oldSku, BillingFlowParams.ProrationMode.DEFERRED)
Appflow.purchasePackage(activity, skuDetails, upgradeInfo, object : MakePurchaseListener{
    override fun onCompleted(purchase: Purchase) {}
    override fun onError(error: PurchasesError, userCancelled: Boolean) {}
})
```

**API Reference**

| UpgradeInfo                                                  |
| ------------------------------------------------------------ |
| UpgradeInfo(oldSku : String, prorationMode : BillingFlowParams.ProrationMode) |
| oldSku : product id to upgrade                               |
| prorationMode  :  Upgrade/Downgrade mode                     |

| BillingFlowParams.ProrationMode                              |                                                              |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| BillingFlowParams.ProrationMode.DEFERRED                     | Replacement takes effect when the old plan expires, and the new price will be charged at the same time |
| BillingFlowParams.ProrationMode.IMMEDIATE_AND_CHARGE_PRORATED_PRICE | Replacement takes effect immediately, and the billing cycle remains the same |
| BillingFlowParams.ProrationMode.IMMEDIATE_WITHOUT_PRORATION  | Replacement takes effect immediately, and the new price will be charged on next recurrence time |
| BillingFlowParams.ProrationMode.IMMEDIATE_WITH_TIME_PRORATION | Replacement takes effect immediately, and the remaining time will be prorated and credited to the user |



##### Paywall

The SDK provides a shortcut for displaying paid products. After setting the style in [dash.appflow.ai](https://dash.appflow.ai), you can display it by call method:

```kotlin
Appflow.showPaywall(context: Context, listener: PaywallListener)
```

When the user clicks the back button, the paywall will automatically close, you can also actively close the following methods

```kotlin
Appflow.closePaywall()
```

**Example**

```kotlin
Appflow.showPaywall(context, object : Appflow.PaywallListener {
    override fun onPurchaseCompleted(purchase: Purchase) {
        Toast.makeText(this@MainActivity, "successful purchase ${purchase.sku}", Toast.LENGTH_SHORT).show()
    }

    override fun onClose() {}

    override fun onFail(msg: String?) {
        Toast.makeText(this@MainActivity, "show paywall fail $msg", Toast.LENGTH_LONG).show()
        // or
        // Appflow.closePaywall()
    }
})
```

**API Reference**

| PaywallListener                         |                                                              |
| --------------------------------------- | ------------------------------------------------------------ |
| onPurchaseCompleted(purchase: Purchase) | When the user successfully purchases the product through the paywall, the method will be called back |
| onClose()                               | This method is called when the user closes the paywall       |
| onFail(msg: String?)                    | This method will be called when the paywall loading error    |



##### Observer Mode

If your project has been connected to the Google Play billing library and you do not want to change the code logic, you only need to enable the observer mode of the SDK during initialization, and the SDK will automatically upload the purchase data to the [dash.appflow.ai](https://dash.appflow.ai)

```kotlin
override fun onCreate() {
    super.onCreate()
  	// setObserverEnable() When set to true, enable observer mode
		// setObserverEnable() When set to false, the observer mode is turned off
		// Default state is false
    Appflow.init(this, AppflowConfig.Builder().setObserverEnable(true).build())
}
```

> **Note: If you use the Appflow.purchasePackage() method in the SDK to purchase, you must turn off the observer mode, otherwise the purchase will fail**



### 5、Push notification

##### Add configuration file

Create a project via the Firebase console, and download the `google-services.json`  and move it into the app's modules (app level) directory



##### Apply gradle plugin

a、If your project doesn't have `dependencyResolutionManagement` in your `settings.gradle`, add the following to your top-level `build.gradle` at the end of repositories：

**top-level build.gradle**

```java
allprojects {
  repositories {
    ...
    // Check that you have the following line (if not, add it):
    google()  // Google's Maven repository
  }
}
```

Otherwise add the following to your `settings.gradle` in `repositories` of `dependencyResolutionManagement` section：

**settings.gradle**

```java
dependencyResolutionManagement {
    repositories {
        ...
        // Check that you have the following line (if not, add it):
    		google()  // Google's Maven repository
    }
}
```



b、 add the following to your top-level `build.gradle` 

**top-level build.gradle**

```java
buildscript {
  repositories {
    // Check that you have the following line (if not, add it):
    google()  // Google's Maven repository
  }

  dependencies {
    // Add the following line:
    classpath 'com.google.gms:google-services:4.3.10'  // Google Services plugin
  }
}
```



c、In your module (app-level) Gradle file (usually `app/build.gradle`), apply the Google Services Gradle plugin:

**app-level build.gradle**

```java
apply plugin: 'com.android.application'
// Add the following line:
apply plugin: 'com.google.gms.google-services'  // Google Services plugin

android {
  // ...
  // add the firebase dependency
  implementation platform('com.google.firebase:firebase-bom:29.0.3')
  implementation 'com.google.firebase:firebase-analytics'
  implementation 'com.google.firebase:firebase-messaging'  
}
```



##### notification message

To get push messages, you have to call method:

```kotlin
Appflow.setListener(listener: FirebaseMsgListener)
```

**Example**

```kotlin
Appflow.setFirebaseMsgListener(object : Appflow.FirebaseMsgListener {
    override fun onNewToken(token: String) {
				//When the Firebase token is updated, it will be called back through this method
    }

    override fun onMessageReceived(remoteMessage: RemoteMessage) {
      	//When the App is in the background, when a push notification message is received, 
        //a notification will pop up directly in the system bar.
      	//When the App is in the foreground, a push notification is received, 
        //and the message is called back through this method.
        remoteMessage.notification?.let {
            Log.d(TAG,"Title:${it.title} Body: ${it.body}")
        }
    }
})
```

**API Reference**

| FirebaseMsgListener                             |                                                              |
| ----------------------------------------------- | ------------------------------------------------------------ |
| onNewToken(token: String)                       | When the Firebase token is updated, it will be called back through this method |
| onMessageReceived(remoteMessage: RemoteMessage) | Push notification message callback (when the App is in the foreground, the push notification will be called back) |



### 6、Attribution

Appflow supports Appsflyer, Adjust, Branch, FacebookAds and custom attribution data upload，you have to call method:

```kotlin
Appflow.updateAttribution(attribution, AttributionType.APPSFLYER, userId)
```



##### Appsflyer

To upload Appsflyer attribution data, refer to the example below

```kotlin
val conversionListener: AppsFlyerConversionListener = object : AppsFlyerConversionListener {
    override fun onConversionDataSuccess(conversionData: Map<String, Any>) {
        Appflow.updateAttribution(
            conversionData,
            AttributionType.APPSFLYER,
            AppsFlyerLib.getInstance().getAppsFlyerUID(context)
        )
    }
}
```



##### Adjust

To upload Adjust attribution data, refer to the example below

```kotlin
val config = AdjustConfig(context, adjustAppToken, environment)
config.setOnAttributionChangedListener { attribution ->
    attribution?.let { attribution ->
        Appflow.updateAttribution(attribution, AttributionType.ADJUST)
    }
}
Adjust.onCreate(config)
```



##### Branch

To upload Branch attribution data, refer to the example below

```kotlin
Branch.getAutoInstance(this)
    .setIdentity("YOUR_USER_ID") { referringParams, error ->
        referringParams?.let { params ->
            Appflow.updateAttribution(params, AttributionType.BRANCH)
        }
    }
```



##### FacebookAds

To upload FacebookAds attribution data, refer to the following example

```kotlin
Appflow.updateAttribution(null, AttributionType.FACEBOOKADS, AppEventsLogger.getAnonymousAppDeviceGUID(context))
```



##### Custom

To upload custom attribution data, refer to the following example

```kotlin
val attribution = mapOf(
    "status" to "non_organic",
    "channel" to "Google Ads",
    "campaign" to "Christmas Sale"
)
Appflow.updateAttribution(attribution, AttributionType.CUSTOM, userId)
```



### 7、UserInfo

You can send user information to Appflow backend in the following ways

```kotlin
Appflow.uploadUserInfo(UserAttribute attribute, UploadUserInfoListener listener)
```

**Example**

```kotlin
val build = UserAttribute.Builder()
    .setName("user_name")
    .setPhone("user_phone")
    .setGender(UserGender.FEMALE)
    .setEmail("user_email")
    .setAge(18)
    .setCustomAttribute("address", "user_address")
    .setCustomAttribute("id", "user_id")
    .build()
Appflow.uploadUserInfo(build, object : UploadUserInfoListener {
    override fun onSuccess() {
    }
    override fun onFail(error: String?) {
    }
})
```

**API Reference**

| UserAttribute                               |                           |
| :------------------------------------------ | ------------------------- |
| setName(name: String)                       | Set user name             |
| setEmail(email: String)                     | Set user email            |
| setGender(gender: UserGender)               | Set user gender           |
| setPhone(phone: String)                     | Set user phone            |
| setAge(age: String)                         | Set user age              |
| setCustomAttribute(key: String, value: Any) | Set custom user attribute |

| UploadUserInfoListener |                  |
| ---------------------- | ---------------- |
| onSuccess()            | Success callback |
| onFail(error: String?) | Failure callback |



### 8、WelcomePage

The SDK provides a welcome page that you can add to your application in the following ways

add in layout page

```xml
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <com.imp.page.welcome.WelcomeView
        android:id="@+id/welcomeView"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>

</FrameLayout>
```

or dynamically add in the code

```kotlin
class SplashActivity : Activity() {

    private lateinit var mWelcomeView: WelcomeView

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        mWelcomeView = WelcomeView(this)
        setContentView(mWelcomeView)
    }
}
```

> **NOTE**
>
> **WelcomePage supports timeout setting，the default timeout is 30 seconds，you can set it in the following ways：**
>
> a、 in layout page，add timeOut property
>
> ```xml
> <?xml version="1.0" encoding="utf-8"?>
> <FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
>     xmlns:app="http://schemas.android.com/apk/res-auto"
>     xmlns:tools="http://schemas.android.com/tools"
>     android:layout_width="match_parent"
>     android:layout_height="match_parent">
> 
>     <!-- Set the timeout to 5 seconds -->
>     <com.imp.page.welcome.WelcomeView
>         android:id="@+id/welcomeView"
>         android:layout_width="match_parent"
>         android:layout_height="match_parent"
>         app:timeOut="5"/>
> 
> </FrameLayout>
> ```
>
> b、in the code，set by constructor
>
> ```kotlin
> class SplashActivity : Activity() {
> 
>     private lateinit var mWelcomeView: WelcomeView
> 
>     override fun onCreate(savedInstanceState: Bundle?) {
>         super.onCreate(savedInstanceState)
>         mWelcomeView = WelcomeView(this, 5)//Set the timeout to 5 seconds
>         setContentView(mWelcomeView)
>     }
> }
> ```
>



When WelcomePage displays success, failure, or timeout, it will tell you through WelcomeViewListener

```kotlin
WelcomeView.setListener(listener: WelcomeViewListener)
```

**Example**

```kotlin
class SplashActivity : AppCompatActivity() {

    private lateinit var mWelcomeView: WelcomeView

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_splash)

        mWelcomeView = findViewById(R.id.welcomeView)
        mWelcomeView.setListener(object : WelcomeView.WelcomeViewListener {
            override fun onShow() {
                Handler(Looper.getMainLooper()).postDelayed({ goMain() }, 2000)
            }

            override fun onFail(t: Throwable) {
                goMain()
            }
        })
    }

    private fun goMain() {
        startActivity(Intent(this@SplashActivity, MainActivity::class.java))
        finish()
    }

    override fun onBackPressed() {}
}
```

**API Reference**

| WelcomeViewListener  |                                 |
| -------------------- | ------------------------------- |
| onShow()             | Success callback                |
| onFail(t: Throwable) | Callback when fails or time out |
