---
layout: inner
title: 'How to use the iOS Keychain in Swift 2: Part One'
date: 2016-05-06 19:30:34
tags: [tutorial, swift, ios, encryption, keychain]
categories: ios
featured_image: 'https://flyingmoose.co/assets/keychain-part-one-featured.png'
lead_text: 'Learn how to encrypt the user data in your app using the iOS Keychain in Swift 2.'
---

If you've ever tried to read through [Apple's documentation](https://developer.apple.com/library/ios/documentation/Security/Reference/keychainservices/) for the iOS/OSX Keychain, you know how frustrating it can be. In addition to the Keychain API not being very "Swifty", there a tons of head-scratching snippets like this: 

> **Return value:** 
> 
> A result code. See Keychain Services Result Codes. Call SecCopyErrorMessageString **(OS X only)** to get a human-readable string explaining the result.

*(Well I guess that's one way to force devs to [make OS X apps](https://www.macworld.com/article/3007290/os-x/the-mac-app-store-not-gone-but-certainly-forgotten.html))*

It's like they *want* you to keep all your access tokens unencrypted.

But fear not! In part one of this tutorial I'll show you how to create your own KeychainManager wrapper to easily encrypt/decrypt items from the iOS Keychain. 

### Setting up the KeychainManager struct 

Create a new .swift file and name it "KeychainManager.swift". At the top of the file before you declare the KeychainManager struct, add the following enum: 

{% highlight swift %}
public enum KeychainError:ErrorType {
    case InvalidInput                      // If the value cannot be encoded as NSData
    case OperationUnimplemented         // -4 | errSecUnimplemented
    case InvalidParam                     // -50 | errSecParam
    case MemoryAllocationFailure         // -108 | errSecAllocate
    case TrustResultsUnavailable         // -25291 | errSecNotAvailable
    case AuthFailed                       // -25293 | errSecAuthFailed
    case DuplicateItem                   // -25299 | errSecDuplicateItem
    case ItemNotFound                    // -25300 | errSecItemNotFound
    case ServerInteractionNotAllowed    // -25308 | errSecInteractionNotAllowed
    case DecodeError                      // - 26275 | errSecDecode
    case Unknown(Int)                    // Another error code not defined
}
{% endhighlight %} 

*You don't need to add the commented blocks, those are just in case you want to reference the specific error codes thrown in the Keychain API*

I won't go too in depth on this, but the important thing here is that the enum conforms to the **ErrorType** protocol so that we can throw them in our **KeychainManager** methods. 

After adding the **KeychainError** enum to the top of the file, go ahead and setup the **KeychainManager** struct. 

{% highlight swift %}
public struct KeychainManager {
	
}
{% endhighlight %}

We'll go over why we're making everything public in part two of this tutorial, but the main reason is that we want all of this code to be reusable across your various projects. [DRY code](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) is key!

### KeychainManager Functions


Your **KeychainManager** will have four basic functions: 

* Adding data to the Keychain
* Deleting data from the Keychain
* Updating data in the Keychain
* Querying data in the Keychain

Before we start writing those methods out, let's create a helper function to help us map the error message to something more readable:

{% highlight swift %}
static func mapResultCode(result:OSStatus) -> KeychainError? {
        switch result {
        case 0:
            return nil
        case -4:
            return .OperationUnimplemented
        case -50:
            return .InvalidParam
        case -108:
            return .MemoryAllocationFailure
        case -25291:
            return .TrustResultsUnavailable
        case -25293:
            return .AuthFailed
        case -25299:
            return .DuplicateItem
        case -25300:
            return .ItemNotFound
        case -25308:
            return .ServerInteractionNotAllowed
        case -26275:
            return .DecodeError
        default:
            return .Unknown(result.hashValue)
        }
    }
{% endhighlight %}

*The **OSStatus** parameter here is an Int that's returned by most of the Keychain API methods.*


Now that we've got that out of the way, let's start creating the core of the **KeychainManager**. 

### Adding and deleting Keychain data

Adding and deleting data from the Keychain is actually relatively straightforward process, the thing that throws people off is the syntax. To add or delete from the Keychain, you need to construct a **CFDictionaryRef** that describes what type of data the Keychain will be working with. 

The four keys you'll need for these dictionaries are: 

* **kSecClass** : What type of data is being stored (we'll be using *kSecClassGenericPassword* which stores a *String*, but there are [multiple other data types](https://developer.apple.com/library/ios/documentation/Security/Reference/keychainservices/index.html#//apple_ref/doc/constant_group/Key_Class_Values))

* **kSecAttrAccount** : The key you want your data to be stored under

* **kSecValueData** : The data (in this case *String*) you want to actually store

* **kSecAttrAccessible** : How restrictive should Keychain be in storing your data (Again, lots of options here but we'll be using *kSecAttrAccessibleWhenUnlocked* for our use case)

To add data to the Keychain, we'll need to first convert the *String* to **NSData** before including it in our **CFDictionaryRef** object. The **SecItemAdd** function is what we'll call with our params dict, which will return an **OSStatus** object. After we get the **resultCode** object, we'll then run it through our **mapResultCode** function to check for any errors, or return if it succeeded. 

{% highlight swift %}
public static func addData(itemKey:String, itemValue:String) throws {
        guard let valueData = itemValue.dataUsingEncoding(NSUTF8StringEncoding) else {
            throw KeychainError.InvalidInput
        }
        
        let queryAdd: [String: AnyObject] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrAccount as String: itemKey,
            kSecValueData as String: valueData,
            kSecAttrAccessible as String: kSecAttrAccessibleWhenUnlocked
        ]
        let resultCode:OSStatus = SecItemAdd(queryAdd as CFDictionaryRef, nil)
        
        if let err = mapResultCode(resultCode) {
            throw err
        } else {
            print("KeychainManager: Item added successfully")
        }
    }
{% endhighlight %}
<br>

Deleting data from the Keychain is very similar to adding it, with the exception of only needing to include the **kSecClass** and **kSecAttrAccount** parameters in our **CFDictionaryRef** object. 

{% highlight swift %}
public static func deleteData(itemKey:String) throws {
        let queryDelete: [String: AnyObject] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrAccount as String: itemKey
        ]
        
        let resultCodeDelete = SecItemDelete(queryDelete as CFDictionaryRef)
        
        if let err = mapResultCode(resultCodeDelete) {
            throw err
        } else {
            print("KeychainManager: Successfully deleted data")
        }
    }
{% endhighlight %}


<br>

### Updating/Querying Keychain data

Updating existing Keychain data is largely the same thing as adding it, we just need to also make a call to the **SecItemCopyMatching** function before actually updating to ensure there is previous Keychain data existing for our key. 

{% highlight swift %}
public static func updateData(itemKey:String, itemValue:String) throws {
        guard let valueData = itemValue.dataUsingEncoding(NSUTF8StringEncoding) else {
            throw KeychainError.InvalidInput
        }
        
        let updateQuery: [String: AnyObject] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrAccount as String: itemKey
        ]
        
        let updateAttributes : [String:AnyObject] = [
            kSecValueData as String: valueData
        ]
        
        if SecItemCopyMatching(updateQuery, nil) == noErr {
            let updateResultCode = SecItemUpdate(updateQuery, updateAttributes)
            if let err = mapResultCode(updateResultCode) {
                throw err
            } else {
                print("KeychainManager: Successfully updated data")
            }
        }
    }
{% endhighlight %}
<br>

One word of caution here: I've never had much success with being able to update existing Keychain data. Typically I'll get a "-34018" error, [which Apple is apparently aware of but not in a hurry to fix](https://forums.developer.apple.com/thread/4743). What I usually do instead of calling the **updateData** function is try to call the **deleteData** function for the key I want to add prior to making the **addData** call, and just ignore any "-25300" (*ItemNotFound*) error I receive. 


Querying data also involves using the **SecItemCopyMatching** function, similar to the **queryData** function. The main difference here is the addition of two new parameters in the initial **CFDictionaryRef** object: 

* **kSecReturnData** : This deals with how data is returned if it requires the user to decrypt the returned **CFDataRef** with a password. We'll set it to *kCFBooleanTrue* since it's required, but doesn't apply for a simple *String* being stored. 

* **kSecMatchLimit** : This deals with how many results should be returned from the search. We'll set it to only return the first matching object (*kSecMatchLimitOne*).

The final thing we need to do here is cast the NSData object that's returned as a String (via NSString). 

{% highlight swift %}
public static func queryData (itemKey:String) throws -> AnyObject? {
        let queryLoad: [String: AnyObject] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrAccount as String: itemKey,
            kSecReturnData as String: kCFBooleanTrue,
            kSecMatchLimit as String: kSecMatchLimitOne
        ]
        var result: AnyObject?
        let resultCodeLoad = withUnsafeMutablePointer(&result) {
            SecItemCopyMatching(queryLoad, UnsafeMutablePointer($0))
        }
        
        if let err = mapResultCode(resultCodeLoad) {
            throw err
        }
        
        guard let resultVal = result as? NSData, keyValue = NSString(data: resultVal, encoding: NSUTF8StringEncoding) as? String else {
            print("KeychainManager: Error parsing keychain result: \(resultCodeLoad)")
            return nil
        }
        return keyValue
    }
{% endhighlight %}
<br>


Now you should have a fully functioning **KeychainManager** that you can use to encrypt access tokens or user data within your app! That's all for part one, but be sure to check out part two for how to make your **KeychainManager** even MORE reusable. 

If you want to check out the source code [I've created a gist here](https://gist.github.com/cabotmoose/529600f3c34b94acba151c3774d39aa1). 


Let me know if you have any questions/suggestions either by tweeting me [@FlyingMooseDev](https://twitter.com/FlyingMooseDev) or sending me an email [cam@flyingmoose.co](mailto:cam@flyingmoose.co)
