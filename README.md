[![Swift 5.0](https://img.shields.io/badge/swift-5.0-ED523F.svg?style=flat)](https://swift.org/download/)
[![Platform](https://img.shields.io/cocoapods/p/SwiftyRate.svg?style=flat)]()

# SwiftyAd

A Swift helper to integrate Ads from AdMob so you can easily show banner, interstitial and rewarded video ads anywhere in your project.

This helper follows all the best practices in regards to ads, like creating shared banners and correctly preloading interstitial and rewarded video ads so they are always ready to show.

## Requirements

- iOS 11.4+
- Swift 5.0+

## GDPR in EEA (European Economic Area)

Please [READ](https://developers.google.com/admob/ios/eu-consent#collect_consent)

As of version 9.0 this helper supports full GDPR support via a new class called `SwiftyAdConsentManager`. It can show the default google consent form or a custom consent form.

NOTE: To show the google consent form please read the instructions above, mainly you have to set your ad technology providers in your AdMob console to manual and not recommended.

The custom consent form is only supported in English, please add your own languages to the String extension in both .swift files.

It is also required that the user has the option to change their GDPR consent settings at anytime, usually via a button in the apps settings menu.

## Mediation

Admob reward videos will only work when using a 3rd party mediation network such as Chartboost or Vungle. 
Please read the AdMob mediation guidlines 

https://developers.google.com/admob/ios/mediation

NOTE:

Make sure to include your mediation networks when setting up SwiftyAd (see How To Use)

## DEBUG

AdMob uses 2 types of AdUnit IDs, 1 for testing and 1 for release. This helper will automatically change the AdUnitID from test to release mode and vice versa. 

You should not show real ads when you are testing your app. In the past Google has been quite strict and has closed down AdMob/AdSense accounts because ads were clicked on apps that were not live. Keep this in mind when you for example test your app via TestFlight because your app will be in release mode when you send a TestFlight build which means it will show real ads.

With the latest xCode it is no longer necessary to setup the DEBUG flag manually.

## Installation

https://developers.google.com/ad-manager/mobile-ads-sdk/ios/quick-start

### Step 1: 

Sign up for a Google [AdMob account](https://support.google.com/admob/answer/3052638?hl=en-GB&ref_topic=3052726) and create your real adUnitIDs for your app, one for each type of ad you will use (Banner, Interstitial, Reward Ads).

### Step 2: 

[CocoaPods](https://developers.google.com/admob/ios/quick-start#streamlined_using_cocoapods) is a dependency manager for Cocoa projects. Simply install the pod by adding the following line to your pod file

```swift
pod 'Google-Mobile-Ads-SDK'
pod 'PersonalizedAdConsent'
```

There is now an [app](https://cocoapods.org/app) which makes handling pods much easier

### Step 3: 

Copy the `source`` folder and its containing files into your project.

### Step 4:

Add a new entry in the info.plist called `GADIsAdManagerApp` (String) with a value of `YES` when using SDK 7.42 or higher

https://developers.google.com/ad-manager/mobile-ads-sdk/ios/quick-start#update_your_infoplist

## Usage

### Setup 

Enter your ad settings into the SwiftyAd.plist file you added to your project. Than call the setup method as soon as your app launches. 

View Controller
```swift
SwiftyAd.shared.setup(with: self, delegate: self, mediationManager: nil) { hasConsent in
    guard hasConsent else { return }
    DispatchQueue.main.async {
        SwiftyAd.shared.showBanner(from: self)
    }
}

NOTE: MediationManager parameter is a protocol with 1 method that you can optionally implement to update your mediation network consent status. Please read the relevant documentation
```

Than create an extension conforming to the AdsDelegate protocol.
```swift
extension GameViewController: SwiftyAdDelegate {
    func swiftyAdDidOpen(_ swiftyAd: SwiftyAd) {
        // pause your game/app if needed
    }
    
    func swiftyAdDidClose(_ swiftyAd: SwiftyAd) { 
       // resume your game/app if needed
    }
    
    func swiftyAd(_ swiftyAd: SwiftyAd, didChange consentStatus: SwiftyAd.ConsentStatus) {
        // see setup for using mediationManager protocol to update your consent status
    }
    
    func swifyAd(_ swiftyAd: SwiftyAd, didRewardUserWithAmount rewardAmount: Int) {
       // Reward amount is a decimel number I converted to an integer for convenience. This value comes from your AdNetwork.
       if let scene = (view as? SKView)?.scene as? GameScene {
            scene.coins += rewardAmount
        }
       
     
       // You can ignore this and hardcode the value if you would like but than you cannot change the value dynamically without having to update your app.
       
       // You could also do something else like unlocking a level or bonus item.
       
       // Leave empty if unused
    }
}
```

AppDelegate
```swift
if let viewController = window?.rootViewController {
      SwiftyAd.shared.setup...
}
```

SKScene
```swift
if let viewController = view?.window?.rootViewController {
      SwiftyAd.shared.setup...
}
```

### Show Ads

UIViewController
```swift
SwiftyAd.shared.showBanner(from: self) 
SwiftyAd.shared.showInterstitial(from: self)
SwiftyAd.shared.showInterstitial(from: self, withInterval: 4) // Shows an ad every 4th time method is called
SwiftyAd.shared.showRewardedVideo(from: self) // Should be called when pressing dedicated button
```

SKScene (Do not call this in didMoveToView as .window property is still nil at that point. Use a delay or call it later)
```swift
if let viewController = view?.window?.rootViewController {
     SwiftyAd.shared.show...
}
```

Note:

You should only show rewarded videos with a dedicated button and you should only show that button when a video is loaded (see below). If the user presses the rewarded video button and watches a video it might take a few seconds for the next video to reload. Incase the user immediately tries to watch another video this helper will show a "no video is available at the moment" alert. 

### Check if ads are ready

```swift
if SwiftyAd.shared.isRewardedVideoReady {
    // show reward video button
}

if SwiftyAd.shared.isInterstitialReady {
    // maybe show custom ad or something similar
}

// When these return false the helper will try to preload an ad again.
```

### Remove Banner ads

e.g during gameplay 

```swift
SwiftyAd.shared.removeBanner() 
```

### Remove/Disable ads (in app purchases)

```swift
SwiftyAd.shared.disable()
```

NOTE:

If set to true the methods to show banner and interstitial ads will not fire anymore and therefore require no further editing. 
This will not stop rewarded videos from showing as they should have a dedicated button. Some rewarded videos are not skipabble and therefore should never be shown automatically. This way you can remove banner and interstitial ads but still have a rewarded videos. 

For permanent storage you will need to create your own "removedAdsProduct" property and save it in something like UserDefaults, or preferably Keychain. Than at app launch check if your saved property is set to true and than update the SwiftyAds poperty e.g

```swift
if UserDefaults.standard.bool(forKey: "RemovedAdsKey") == true {
    SwiftyAd.shared.disable()
}
```

### To ask for consent again (GDPR) 

It is required that the user has the option to change their GDPR consent settings, usually via a button in settings.

```swift
SwiftyAd.shared.askForConsent(from: viewController)
```

The consent button can be hidden for non EEA users like so

```swift
consentButton.isHidden = !SwiftyAd.shared.isRequiredToAskForConsent
```

## When you submit your app to Apple

When you submit your app to Apple on iTunes connect do not forget to select YES for "Does your app use an advertising identifier", otherwise it will get rejected. If you use reward videos you should also select the 3rd bulletpoint.

## Tip

From my personal experience and from a user perspective you should not spam full screen interstitial ads all the time. This will also increase your revenue because user retention rate is higher so you should not be greedy. Therefore you should

1) Not show an interstitial ad everytime a button is pressed 
2) Not show an interstitial ad everytime you die in a game

## Final Info

The sample project is the basic Apple spritekit template. It now shows a banner Ad after launch and an interstitial ad randomly when touching the screen. After a certain amount of clicks all ads will be removed to simulate what a remove ads button would do. 

Please feel free to let me know about any bugs or improvements. 

Enjoy
