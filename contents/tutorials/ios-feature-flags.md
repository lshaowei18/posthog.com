---
title: How to set up feature flags in iOS
date: 2024-02-20
author:
  - lior-neu-ner
tags:
  - feature flags
---

import { ProductScreenshot } from 'components/ProductScreenshot'
export const EventsInPostHogLight = "https://res.cloudinary.com/dmukukwp6/image/upload/posthog.com/contents/images/tutorials/ios-feature-flags/events-light.png"
export const EventsInPostHogDark = "https://res.cloudinary.com/dmukukwp6/image/upload/posthog.com/contents/images/tutorials/ios-feature-flags/events-dark.png"
export const CreateFlagLight = "https://res.cloudinary.com/dmukukwp6/image/upload/posthog.com/contents/images/tutorials/ios-feature-flags/create-flag-light.png"
export const CreateFlagDark = "https://res.cloudinary.com/dmukukwp6/image/upload/posthog.com/contents/images/tutorials/ios-feature-flags/create-flag-dark.png"

[Feature flags](/feature-flags) help you conditionally roll out and release features safely. This tutorial shows you how integrate them in iOS using PostHog. 

We'll create a basic iOS app, add PostHog, create a feature flag, and then implement the flag to control content in our app.

## 1. Create a new iOS app

Our app will have two screens. The first screen will have a button which takes you to a second screen. The second screen will either have a `red` or `green` background color depending on whether our feature flag is enabled or not.

The first step is to create a new app. Open XCode and click "Create new project". Select iOS as your platform, then "App" and press next. Give your app a name, select `SwiftUI` as the interface, and the defaults for everything else. Click next and then "Create".

Then, replace your code in `ContentView.swift` with the following:

```swift file=ContentView.swift
import SwiftUI

struct ContentView: View {
    @State private var navigateToFeatureScreen = false

    var body: some View {
        NavigationStack {
            VStack {
                Image(systemName: "globe")
                    .imageScale(.large)
                    .foregroundStyle(.tint)
                Text("Hello, world!")

                Button("Go to Next Screen") {
                    navigateToFeatureScreen = true
                }
                .padding()
                .background(Color.blue)
                .foregroundColor(.white)
                .cornerRadius(10)
            }
            .padding()
            .navigationDestination(isPresented: $navigateToFeatureScreen) {
                FeatureScreenView(isFlagEnabled: false) // We'll update this later
            }
        }
    }
}
```

Then, create a new SwiftUI View called `FeatureScreenView`. Replace the default code in the new file with the following:

```swift file=FeatureScreenView.swift
import SwiftUI

struct FeatureScreenView: View {
    var isFlagEnabled: Bool

    var body: some View {
        ZStack {
            Color(isFlagEnabled ? .green : .red)
                .edgesIgnoringSafeArea(.all)
            .padding()
            .cornerRadius(10)
        }
    }
}
```

Our basic set up is now complete. Build and run your app to see it in action.

![Basic setup of the iOS app](https://res.cloudinary.com/dmukukwp6/image/upload/v1710055416/posthog.com/contents/images/tutorials/ios-feature-flags/basic-app.png)

## 2. Add PostHog to your app

With our app set up, it’s time to install and set up PostHog. If you don't have a PostHog instance, you can [sign up for free](https://us.posthog.com/signup).

First, add [`posthog-ios`](/docs/libraries/ios) as a dependency to your app using [Swift Package Manager](https://developer.apple.com/documentation/xcode/adding-package-dependencies-to-your-app) (or if you prefer, you can use [CocoaPods](/docs/libraries/ios#cocoapods).

To add the package dependency to your Xcode project, select `File > Add Package Dependency` and enter the URL `https://github.com/PostHog/posthog-ios.git`. Select `posthog-ios` and click Add Package.

Note that for this tutorial we use version `3.1.3` of the SDK.

![Add PostHog from Swift Package Manager](https://res.cloudinary.com/dmukukwp6/image/upload/v1710055416/posthog.com/contents/images/tutorials/ios-feature-flags/swift-npm.png)

Next, configure your PostHog instance in `App.swift` using your project API key and instance address (you can find these in [your project settings](https://us.posthog.com/project/settings)):

```swift file=App.swift
import SwiftUI
import PostHog

@main
struct posthog_feature_flagsApp: App {
    init() {
        let POSTHOG_API_KEY = "<ph_project_api_key>"
        // usually 'https://us.i.posthog.com' or 'https://eu.i.posthog.com'
        let POSTHOG_HOST = "<ph_client_api_host>"
        let configuration = PostHogConfig(apiKey: POSTHOG_API_KEY, host: POSTHOG_HOST) // TIP: host is optional if you use https://us.i.posthog.com
        PostHogSDK.shared.setup(configuration)
    }
    
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}
```

To check your setup, build and run your app. Click your button a few times. You should start seeing events in the [activity tab](https://us.posthog.com/events).

<ProductScreenshot
  imageLight={EventsInPostHogLight} 
  imageDark={EventsInPostHogDark} 
  alt="Events captured in PostHog" 
  classes="rounded"
/>

## 3. Create a feature flag in PostHog

With PostHog set up, your app is ready for feature flags. To create one, go to the [feature flags tab](https://us.posthog.com/feature_flags) in PostHog and click **New feature flag**. Enter a flag key (like `my-cool-flag`), set the release condition to roll out to 100% of users, and press "Save."

<ProductScreenshot
  imageLight={CreateFlagLight} 
  imageDark={CreateFlagDark} 
  alt="Feature flag created in PostHog" 
  classes="rounded"
/>

You can customize your [release conditions](/docs/feature-flags/creating-feature-flags#release-conditions) with rollout percentages, and [user](/docs/product-analytics/person-properties) or [group properties](/docs/product-analytics/group-analytics) to fit your needs.

## 4. Implement the flag code

To implement the feature flag, we: 

1. Fetch the `my-cool-flag` flag using [`PostHogSDK.shared.isFeatureEnabled()`](/docs/libraries/ios#feature-flags).
2. Change the background color of `FeatureScreenView` based on the value of the flag.

To do this, update the code in `ContentView.swift` with the following:

```swift file=ContentView.swift
import SwiftUI
import PostHog

// ...

    @State private var isFlagEnabled = false

// ...
    var body: some View {
        NavigationStack {
            VStack {
               // ...

               Button("Go to Next Screen") {
                    // Fetch feature flag here
                    isFlagEnabled = PostHogSDK.shared.isFeatureEnabled("my-cool-flag")
                    navigateToFeatureScreen = true
                }

// ...

            .navigationDestination(isPresented: $navigateToFeatureScreen) {
                FeatureScreenView(isFlagEnabled: isFlagEnabled) 
            }
        }
    }
}
```

That's it! When you restart your app and click the button, you should see the green background color on the second screen. 

## Further reading

- [How to set up iOS session replay](/tutorials/ios-session-replay)
- [How to run A/B tests in iOS](/tutorials/ios-ab-tests)
- [How to set up analytics in iOS](/tutorials/ios-analytics)
- [How to set up remote config in iOS](/tutorials/ios-remote-config)
<NewsletterForm />