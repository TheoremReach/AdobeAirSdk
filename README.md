# TheoremReach ANE
Adboe AIR Native Extension for TheoremReach SDK

## Setup
Create an [app](https://theoremreach.com/developer/apps) and grab your API Key and optionally User Id.

## Install ANE

1. Download [com.theoremreach.TheoremReach.ane](https://github.com/TheoremReach/AdobeAirSdk/blob/master/com.theoremreach.TheoremReach.ane) ANE and [add it as a dependency](http://bit.ly/2xTSJry) to your project. Optionally you may include corresponded SWC to your project, that could be found on the same place.

1. This ANE requires several supporting ANEs from [Distriqt](http://distriqt.com/) for Android part, you have to download them and add as a dependencies as described in step 1, these ANEs are:
   * [com.distriqt.androidsupport.V4.ane](https://github.com/distriqt/ANE-AndroidSupport/blob/master/lib/com.distriqt.androidsupport.V4.ane) contains Android Support v4 library
   * [com.distriqt.androidsupport.AppCompatV7.ane](https://github.com/distriqt/ANE-AndroidSupport/blob/master/lib/com.distriqt.androidsupport.AppCompatV7.ane) contains Android Compat v7 library
   * [com.distriqt.playservices.Base.ane](https://github.com/distriqt/ANE-GooglePlayServices/blob/master/lib/com.distriqt.playservices.Base.ane) for Google Play Services

1. Edit your [Application Descriptor](http://help.adobe.com/en_US/air/build/WS5b3ccc516d4fbf351e63e3d118666ade46-7ff1.html) with next changes: 
   * Register new ANEs additionally to ones you may already have:
    ```xml
    <extensions>
        <extensionID>com.theoremreach.TheoremReach</extensionID>
        <extensionID>com.distriqt.playservices.Base</extensionID>
        <extensionID>com.distriqt.androidsupport.AppCompatV7</extensionID>
        <extensionID>com.distriqt.androidsupport.V4</extensionID>
    </extensions>
    ```
   * Enable arbitrary loads on iOS:
    ```xml
    <iPhone>
        <!-- A list of plist key/value pairs to be added to the application Info.plist -->
        <InfoAdditions>
            <![CDATA[
            <key>NSAppTransportSecurity</key>
            <dict>
                <key>NSAllowsArbitraryLoads</key>
                <true/>
            </dict>
            ]]>
        </InfoAdditions>
    </iPhone>
    ```
    * Make sure `INTERNET` and `ACCESS_NETWORK_STATE` permissions are enabled for Android:
    ```xml
    <android>
        <manifestAdditions>
            <![CDATA[
            <manifest android:installLocation="auto">
                <uses-permission android:name="android.permission.INTERNET"/>
                <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
                ...
            </manifest>
            ]]>
        </manifestAdditions>
    </android>
    ```
    * Include `com.google.android.gms.version` property into application descriptor like this:
    ```xml
    <android>
        <manifestAdditions>
            <![CDATA[
            <manifest android:installLocation="auto">
                ...
                <application android:enabled="true">
                    <meta-data android:name="com.google.android.gms.version" android:value="@integer/google_play_services_version" />
                    ...
                </application>
            </manifest>
            ]]>
        </manifestAdditions>
    </android>
    ``` 
    * And register `RewardCenterActivity` and `MomentSurveyActivity` activities for Android application, so Android part will look like this:
    ```xml
    <android>
        <manifestAdditions>
            <![CDATA[
            <manifest android:installLocation="auto">
                <uses-permission android:name="android.permission.INTERNET"/>
                <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
                <application android:enabled="true">
                    <meta-data android:name="com.google.android.gms.version" android:value="@integer/google_play_services_version" />
                    <activity
                        android:name="theoremreach.com.theoremreach.RewardCenterActivity"
                        android:configChanges="keyboard|keyboardHidden|screenSize|orientation"
                        android:label="TheoremReach"
                        android:theme="@style/LibraryTheme" />
                    <activity
                        android:name="theoremreach.com.theoremreach.MomentSurveyActivity"
                        android:configChanges="keyboard|keyboardHidden|screenSize|orientation"
                        android:label="TheoremReach"
                        android:theme="@style/MomentTheme" />
                </application>
            </manifest>
            ]]>
        </manifestAdditions>
    </android>
    ```
1. Patch AIR SDK. It could be an unusual part, but some iOS API uses `@available` keyword that is [not supported](https://tracker.adobe.com/#/view/AIR-4198557) by AIR SDK. As a workaround we need to patch AIR SDK with including `libclang_rt.ios.a` library to it. You may find this file in your Xcode application folder like this `/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/lib/clang/10.0.0/lib/darwin/libclang_rt.ios.a`, or download one attached to this repo at `./build/ios/libclang_rt.ios.a` path. Then put this file into your AIR SDK at path `${AIR_SDK}/lib/aot/lib`, same you can do for Flex and Feathers SDKs.

## Usage

### Initialize TheoremReach SDK

Initialize TheoremReach SDK calling `initWithApiKeyAndUserId` method with API Key and User Id:
```actionscript3
TheoremReach.initWithApiKeyAndUserId("${API_KEY}", "${USER_ID}");
```

### Reward Center

Call `showRewardCenter` method when you are ready to show Reward Center:
```actionscript3
if (TheoremReach.shared.isSurveyAvaialable) {
    TheoremReach.shared.showRewardCenter();
}
```

You may want to customize NavigationBar appearance:
```actionscript3
TheoremReach.shared.navigationBarText = "New Reward Center Title"
TheoremReach.shared.navigationBarColor = "#282a36";
TheoremReach.shared.navigationBarTextColor = "E0DFE0"; 
```

### Reward Callback

To ensure safety and privacy, we notify you of all awards via a server side callback. In the developer dashboard for your App add the server callback that we should call to notify you when a user has completed an offer. Note the user ID pass into the initialize call will be returned to you in the server side callback. More information about setting up the callback can be found in the developer dashboard.

The quantity value will automatically be converted to your virtual currency based on the exchange rate you specified in your app. Currency is always rounded in favor of the app user to improve happiness and engagement.

### Client Side Award Callback

For security purposes we always recommend that developers utilize a server side callback, however we also provide APIs for implementing a client side award notification if you lack the server structure or a server altogether or want more real-time award notification. It's important to only award the user once if you use both server and client callbacks (though your users may not be opposed!).
```actionscript3
TheoremReach.shared.onReward = function(quantity: int): void {
    trace("TheoremReach onReward: " + quantity);
};
```

#### Reward Center Events

You can optionally set callbacks for `onRewardCenterOpened` and `onRewardCenterClosed` events:
```actionscript3
TheoremReach.shared.onRewardCenterOpened = function(): void {
    trace("onRewardCenterOpened");
};

TheoremReach.shared.onRewardCenterClosed = function(): void {
    trace("onRewardCenterClosed");
};
```

#### Survey Available Callback

If you'd like to be notified when a survey is available you can add a callback:
```actionscript3
TheoremReach.shared.onTheoremReachSurveyAvailable = function(available: Boolean): void {
    log("onTheoremReachSurveyAvailable: " + available);
};

```

## Other platforms:

[TheoremReach iOS SDK Integration](https://theoremreach.com/docs/ios)

[TheoremReach Android SDK Integration](https://theoremreach.com/docs/android)

[TheoremReach Javascript Web SDK Integration](https://theoremreach.com/docs/web)  

## Contact
Please send all questions, concerns, or bug reports to admin@theoremreach.com.

## FAQ

##### What do you do to protect privacy?
We take privacy very seriously. All data is encrypted before being sent over the network. We also use HTTPS to ensure the integrity and privacy of the exchanged data.

##### What kind of analytics do you provide?
Our dashboard will show metrics for sessions, impressions, revenue, and much more. We are constantly enhancing our analytics so we can better serve your needs.

##### What is your fill rate?
We have thousands of surveys and add hundreds more every day. Most users will have the opportunity to complete at least one survey on a daily basis.

##### I'm ready to go live! What are the next steps?
Let us know! We'd love to help ensure everything flows smoothly and help you achieve your monetisation goals!