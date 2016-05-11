---
layout: inner
title: 'How to use the iOS Keychain in Swift 2: Part Two'
date: 2016-05-07 19:30:34
tags: [tutorial, swift, ios, framework, encryption, keychain, embedded_framework]
categories: ios
featured_image: 'https://flyingmoose.co/assets/posts/2016-05-07-swift-2-guide-to-keychain-encryption-part-two/build-framework.png'
lead_text: 'Use what you learned in part one of this tutorial to create your own reusable embedded Swift framework to encrypt data in the iOS Keychain.'
---

In [part one]({{ site.data.global.url }}/blog/ios/swift-2-guide-to-keychain-encryption-part-one.html) of this tutorial series you learned how to interact with Apple's [Keychain Services API](https://developer.apple.com/library/ios/documentation/Security/Reference/keychainservices/) in order to securely store and encrypt your app's data. In part two you will learn how to take the **KeychainManager** struct you created and turn it into it's own framework that you can reuse in all your future apps!

*[Here's a link](https://gist.github.com/cabotmoose/529600f3c34b94acba151c3774d39aa1) to the final **KeychainManager** file from part one if you need it.*

### Creating your own embedded framework
<br>
To start, create a new project in Xcode and choose *iOS -> Framework & Library -> Cocoa Touch Framework* as the project type. 

{::options parse_block_html="true" /}

<img class="img-responsive" src="{{ site.data.global.url }}/assets/posts/2016-05-07-swift-2-guide-to-keychain-encryption-part-two/create-new-framework.png"/>


Name the project whatever you want, but I named mine **KeychainAccess** which is how I'll refer to the project from here on. 

#### (Optional) Restrict your framework to only use App Extension APIs 

This section is only applicable if you would like to use this framework with any [App Extension](https://developer.apple.com/library/ios/documentation/General/Conceptual/ExtensibilityPG/ExtensionScenarios.html) projects you might create (which is what I used it for in [Swipe](http://flyingmoose.co/swipe))

Next, select **KeychainAccess** in the project navigator and navigate to *Targets -> KeychainAccess*. Under the *Deployment Info* section in the *General* tab, check the *Allow app extension API only*. 

{::options parse_block_html="true" /}

<img class="img-responsive" src="{{ site.data.global.url }}/assets/posts/2016-05-07-swift-2-guide-to-keychain-encryption-part-two/allow-app-extensions.png"/>


Apple [excludes certain APIs from being accessible](https://developer.apple.com/library/ios/documentation/General/Conceptual/ExtensibilityPG/ExtensionOverview.html#//apple_ref/doc/uid/TP40014214-CH2-SW6) to embedded frameworks, so this ensures that only allowed system libraries are being used in your extension. 


### Adding the KeychainManager code 

Now that you've got the basic structure of your framework in place, drag the **KeychainManager.swift** file from part one into the project. At this point your project should look like this: 

{::options parse_block_html="true" /}

<img class="img-responsive" src="{{ site.data.global.url }}/assets/posts/2016-05-07-swift-2-guide-to-keychain-encryption-part-two/add-keychain-manager.png"/>


The next step is to build the actual framework. 

### Build Your Framework

Due to the way Xcode handles the build architecture for frameworks, you will need to select the appropriate build scheme in the device drop down at the top of Xcode. If you want to use your framework on actual devices, either select your device or select *"Generic iOS Device"* like in the screenshot below. If you want to use your framework with the iOS simulators, you will need to select a simulator before building. 

There are ways to [automate this process](https://gist.github.com/brett-stover-hs/8196a9c237f291542910/raw/fbce985f8ba5b872d88c0eab983136eb3b5b855a/frameworks_blogpost_merge_script.sh), but they are outside the scope of this tutorial. 


{::options parse_block_html="true" /}

<img class="img-responsive" src="{{ site.data.global.url }}/assets/posts/2016-05-07-swift-2-guide-to-keychain-encryption-part-two/build-framework.png"/>


After you've selected the appropriate build scheme, go ahead and build (cmd + b) your project and look under the *"Products"* folder in the project navigator to find your compiled framework! At this point you're framework's ready to use in your actual projects. 

To locate your new framework, right click on the **KeychainAccess.framework** item in the project navigator and select *"Show in Finder"*. 

Using your new framework in other projects is simple. Just drag the framework into a project's **Embedded Binaries** section in the target's build settings and select *"Copy items if needed"* when importing. 

{::options parse_block_html="true" /}

<img class="img-responsive" src="{{ site.data.global.url }}/assets/posts/2016-05-07-swift-2-guide-to-keychain-encryption-part-two/copy-framework.png"/>


**And that's all there is!**

Let me know if you have any questions/suggestions either by tweeting me [@FlyingMooseDev](https://twitter.com/FlyingMooseDev) or sending me an email [cam@flyingmoose.co](mailto:cam@flyingmoose.co)
