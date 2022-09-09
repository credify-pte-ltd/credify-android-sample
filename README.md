# Credify SDK Sample

The SDK supports API level 23 and above ([distribution stats](https://developer.android.com/about/dashboards)).

This guideline is for the SDK `v0.6.0+`.

If you are using version `v0.5.0` or lower you should follow [this guideline](./readMe/README-v0.5.0-or-lower.md)

Update the `build.gradle`(project level).

```java
allprojects {
    repositories {
        ...
        mavenCentral()
    }
}
```

Update the `build.gradle`(app level).

```java
dependencies {
    implementation 'one.credify.sdk:android-sdk:[last-version]'
}
```

Sync project with Gradle file.

**Note:** If you gets the below error when you build your project.

<img width="569" alt="Screenshot 2021-07-01 at 20 10 33" src="https://user-images.githubusercontent.com/18586774/124133379-f9320600-daab-11eb-8ae1-84c4c6deed5a.png">

Then you need to add this line *tools:replace="android:supportsRtl"* into the *<application>* element in the *AndroidManifest.xml*.

<img width="622" alt="Screenshot 2021-07-01 at 20 24 52" src="https://user-images.githubusercontent.com/18586774/124133651-3d250b00-daac-11eb-80fd-981cf85568c8.png">

If the `compileSdkVersion` is less than `31` then you need to add the below scripts:

```java
android {
  ...
  configurations.all {
      resolutionStrategy { force 'androidx.core:core-ktx:1.6.0' }
      resolutionStrategy { force 'androidx.appcompat:appcompat:1.3.1' }
  }
  ...
}
```

## Getting stated

### Set up the SDK

Create `CredifySDK` instance.

If you have created a class that extends from `Application` class. You only add the below code to the `onCreate()` method.
You can generate a new API key on the serviceX dashboard.

```kotlin
override fun onCreate() {
    super.onCreate()
    ...
    CredifySDK.Builder()
        .withApiKey([Your API Key])
        .withContext(this)
        .withEnvironment([Environment])
        .withTheme([ServiceXThemeConfig])
        .build()
    ...
}
```

Otherwise, you will need to create a new class that extends from `Application` class. In this example, I named it with `DemoApplication`.

```kotlin
...
class DemoApplication : Application() {
    override fun onCreate() {
        super.onCreate()

        // Generates a singleton here
        CredifySDK.Builder()
            .withApiKey([Your API Key])
            .withContext(this)
            .withEnvironment([Environment])
            .withTheme([ServiceXThemeConfig])
            .build()
    }
    ...
}
```

**AND** update your `AndroidManifest.xml` file.

```xml
<application
    android:name=".DemoApplication"
    ...
    >
    ...
</application>
```

After creating the `CredifySDK` instance, you can access the singleton like following:

```kotlin
val credifySDK = CredifySDK.instance
```

### Offer usage

#### Get offers list

Your backend need to provide an API for getting offer list. The response should obtain the `credify_id` if the user have created account on Credify.

> **Important**: you need to keep `credifyId` on your side. You have to send the `credifyId` to Credify SDK when you use the methods that require `credifyId`. E.g: `CredifySDK.instance.offerApi.showOfferByCode`

#### Show an offer detail

Create `one.credify.sdk.core.model.UserProfile` object.

```kotlin
val user = UserProfile(
    id = // Your user id,
    name = UserName(
        firstName = // Your user's first name,
        lastName = // Your user's last name,
        middleName = // Your user's middle name (Optional),
        name = // Your user's full name (Optional)
    ),
    phone = UserPhoneNumber(
        phoneNumber = // Your user's phone number,
        countryCode = // Your user's phone country code
    ),
    email = // Your user's email,
    dob = // Your user's day of birth (Optional),
    address = // Your user's address (Optional),
    credifyId = // Your user's credify id. If your user have created Credify account then it should not be null
)
```

To show an **offer detail** by using:

```kotlin
CredifySDK.instance.offerApi.showOfferByCode(
    context = // Context,
    offerCode = // the selected offer code,
    userProfile = // one.credify.sdk.core.model.UserProfile object,
    pushClaimCallback = // CredifySDK.PushClaimCallback callback,
    offerPageCallback = // CredifySDK.OfferPageCallback callback
)
```

You have to handle the `CredifySDK.PushClaimCallback` callback for pushing claims. For example:

```kotlin
CredifySDK.instance.offerApi.showOfferByCode(
    context = // Context,
    offerCode = // the selected offer code,
    userProfile = // one.credify.sdk.core.model.UserProfile object,
    // Callback for pushing claims
    pushClaimCallback = object : CredifySDK.PushClaimCallback {
        override fun onPushClaim(
            credifyId: String,
            user: UserProfile, // It's removed from version v.0.2.x
            resultCallback: CredifySDK.PushClaimResultCallback
        ) {
            // Code for pushing claims, you need to call your API to do this task.
            // After pushing claims, you have to notify to Credify SDK. For example:
            resultCallback.onPushClaimResult(
                isSuccess = [true if success. Otherwise, pass false]
            )
        }
    },
    // Callback for closing the page
    offerPageCallback = object : CredifySDK.OfferPageCallback {
        override fun onClose(status: RedemptionResult) {
            // There are three status
            // - COMPLETED: the user redeemed offer successfully and the offer transaction status is COMPLETED.
            // - PENDING:   the user redeemed offer successfully and the offer transaction status is PENDING.
            // - CANCELED:  the user redeemed offer successfully and he canceled this offer afterwords OR he clicked
            //              on the back button in any screens in the offer redemption flow.
        }

        override fun onOpenUrl(url: String) {
        }
    }
)
```

> **Important**: you need to keep `credifyId` on your side. You have to send the `credifyId` to Credify SDK when you use the methods that require `credifyId`. E.g: `CredifySDK.instance.offerApi.showOfferByCode`

#### Show promotion offer list

Using below method to show promotions offer list. The parameters are the same with the `CredifySDK.instance.offerApi.showOfferByCode` except you have to pass `offerCodes` list instead of an `offerCode`.

```kotlin
CredifySDK.instance.offerApi.showPromotionOffersByCodes(
    context = // Context,
    offerCodes = // The list of offer codes,
    userProfile = // one.credify.sdk.core.model.UserProfile object,
    pushClaimCallback = object : CredifySDK.PushClaimCallback {
        override fun onPushClaim(
            credifyId: String,
            user: UserProfile, // It's removed from version v.0.2.x
            resultCallback: CredifySDK.PushClaimResultCallback
        ) {
            // Code for pushing claims, you need to call your API to do this task.
            // After pushing claims, you have to notify to Credify SDK. For example:
            resultCallback.onPushClaimResult(
                isSuccess = [true if success. Otherwise, pass false]
            )
        }
    },
    offerPageCallback = object : CredifySDK.OfferPageCallback {
        override fun onClose(status: RedemptionResult) {
            // There are three status
            // - COMPLETED: the user redeemed offer successfully and the offer transaction status is COMPLETED.
            // - PENDING:   the user redeemed offer successfully and the offer transaction status is PENDING.
            // - CANCELED:  the user redeemed offer successfully and he canceled this offer afterwords OR he clicked
            //              on the back button in any screens in the offer redemption flow.
        }

        override fun onOpenUrl(url: String) {
        }
    }
)
```

#### Setting language

- Using `CredifySDK.instance.setLanguage(language: String)` to setup the language that will be used for the localization in the SDK.

### BNPL usage

#### Get offers list

Your backend need to provide an API for getting offer list. The response should obtain the `credify_id` if the user have created account on Credify.

> **Important**: you need to keep `credifyId` on your side. You have to send the `credifyId` to Credify SDK when you use the methods that require `credifyId`. E.g: `CredifySDK.instance.bnplApi.showBNPLByCodes`

#### Show BNPL

you use `CredifySDK.instance.bnplApi.showBNPLByCodes` to start the BNPL flow.

```kotlin
CredifySDK.instance.bnplApi.showBNPLByCodes(
    context = // Context,
    offerCodes = // The list of offer codes,
    packageCode = // Optional, you can pass nil for now,
    userProfile = // one.credify.sdk.core.model.UserProfile object,
    orderInfo = // The order info object, you need to create it on your side,
    pushClaimCallback = object : CredifySDK.PushClaimCallback {
        override fun onPushClaim(
            credifyId: String,
            resultCallback: CredifySDK.PushClaimResultCallback
        ) {
            // Code for pushing claims, you need to call your API to do this task.
            // After pushing claims, you have to notify to Credify SDK. For example:
            resultCallback.onPushClaimResult(
                isSuccess = [true if success. Otherwise, pass false]
            )
        }
    },
    bnplPageCallback = object : CredifySDK.BNPLPageCallback {
        override fun onClose(
            status: RedemptionResult,
            orderId: String,
            isPaymentCompleted: Boolean
        ) {
            // Receive the result and update on your side if needed
        }
    }
)
```

### Show Passport

Using the below code for showing the **Passport web app**. This page will show all the offers which the user has redeemed.

> **Important**: This method must be used after the user created an account. That's mean you have `credifyId` on your side

```kotlin
CredifySDK.instance.passportApi.showPassport(
    context = // Context,
    userProfile = // one.credify.sdk.core.model.UserProfile object,
    credifyId = // Your user's credify id,
    pushClaimCallback = // CredifySDK.PushClaimCallback callback. It's the same argument in the above CredifySDK.instance.offerApi.showOffer
    callback = object : CredifySDK.PassportPageCallback {
        override fun onShow() {
            // The page is showing on the UI
        }

        override fun onClose() {
            // The page is closed
        }
    }
)
```

### Show the Service detail

Using the below code for showing the **Service detail page**. It will show all the BNPL details which the user has used.

```kotlin
CredifySDK.instance.passportApi.showServiceInstance(
    context = // Context,
    userProfile = // one.credify.sdk.core.model.UserProfile object,
    marketId = // Your orgnization id that you have registered with Credify,
    productTypeList = // Product type list,
    callback = object : CredifySDK.PageCallback {
        override fun onClose() {
            // The page is showing on the UI
        }

        override fun onShow() {
            // The page is closed
        }
    }
)
```

### Customize theme

Below is an example result if you do customize theme.

![Theme](./imgs/ThemeOverview.png)

You can use `ServiceXThemeConfig` class to config the `fonts`, `colors`, `input fields` and so on for the serviceX SDK.
You have to create `ServiceXThemeConfig` object when initializing the SDK. But it is optional. The SDK will use the default theme if you don't want to customize theme

```kotlin
    CredifySDK.Builder()
        .withApiKey([Your API Key])
        .withContext(this)
        .withEnvironment([Environment])
        .withTheme([ServiceXThemeConfig])
        .build()
```

#### ServiceXThemeConfig class

```kotlin
/**
 * @param color: it is [ThemeColor] object
 * @param font: it is [ThemeFont] object
 * @param icon: it is [ThemeIcon] object
 * @param actionBarTopLeftRadius: top-left action bar radius
 * @param actionBarBottomLeftRadius: bottom-left action bar radius
 * @param actionBarTopRightRadius: top-right action bar radius
 * @param actionBarBottomRightRadius: bottom-right action bar radius
 * @param datePickerStyle: data picker theme(style id)
 * @param elevation: it is in dp
 * @param inputFieldRadius: it is in dp
 * @param modelRadius: it is in dp
 * @param buttonRadius: it is in dp
 */
class ServiceXThemeConfig(
    val context: Context,
    val color: ThemeColor,
    val font: ThemeFont,
    val icon: ThemeIcon,
    val actionBarTopLeftRadius: Float,
    val actionBarBottomLeftRadius: Float,
    val actionBarTopRightRadius: Float,
    val actionBarBottomRightRadius: Float,
    val datePickerStyle: Int,
    val elevation: Float,
    val inputFieldRadius: Float,
    val modelRadius: Int = 10,
    val buttonRadius: Float = 50F,
)
```

![Theme](./imgs/ThemeBorderRadius.png)

#### ThemeColor class

```kotlin
/**
 * All properties are in [Int]. We can use 
 * - Color.parse([String]),
 * - ContextCompat.getColor([Context], [Color resource id])
 * - Color.WHITE, Color.BLUE,...
 
 * For example: Color.parseColor("#ff00ff")
 */
class ThemeColor(
    val primaryBrandyStart: Int,
    val primaryBrandyEnd: Int,
    val primaryText: Int,
    val secondaryActive: Int,
    val secondaryDisable: Int,
    val secondaryText: Int,
    val secondaryComponentBackground: Int,
    val secondaryBackground: Int,
    val topBarTextColor: Int,
    val primaryButtonTextColor: Int,
    val primaryButtonBrandyStart: Int,
    val primaryButtonBrandyEnd: Int,
)
```

![Theme](./imgs/ThemeColor1.png)
![Theme](./imgs/ThemeColor2.png)
![Theme](./imgs/ThemeColor3.png)

#### ThemeFont class

```kotlin
/**
 * @param primaryFontFamily it is [Typeface] object. We can use ResourcesCompat.getFont(Context, R.font.your_font)
 * @param secondaryFontFamily it is [Typeface] object. We can use ResourcesCompat.getFont(Context, R.font.your_font)
 */
class ThemeFont(
    val context: Context,
    val primaryFontFamily: Typeface?,
    val secondaryFontFamily: Typeface?,
    val secondaryFontWeight: Int,
    val bigTitleFontSize: Float,
    val bigTitleFontLineHeight: Int,
    val modelTitleFontSize: Float,
    val modelTitleFontLineHeight: Int,
    val sectionTitleFontSize: Float,
    val sectionTitleFontLineHeight: Int,
    val bigFontSize: Float,
    val bigFontLineHeight: Int,
    val normalFontSize: Float,
    val normalFontLineHeight: Int,
    val smallFontSize: Float,
    val smallFontLineHeight: Int,
    val boldFontSize: Float,
    val boldFontLineHeight: Int,
)
```

![Theme](./imgs/ThemeFont1.png)
![Theme](./imgs/ThemeFont2.png)
![Theme](./imgs/ThemeFont3.png)
![Theme](./imgs/ThemeFont4.png)

#### ThemeIcon class

```kotlin
class ThemeIcon(
    val context: Context,
    val backIcon: Drawable?,
    val closeIcon: Drawable?,
)

```

![Theme](./imgs/ThemeIcon1.png)
![Theme](./imgs/ThemeIcon2.png)

## Contacts

If you find any bugs or issues, please let us know (dev@credify.one). Of course, welcome to open new issues in this GitHub!
