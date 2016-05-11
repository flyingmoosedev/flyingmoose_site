---
layout: inner
title: 'The Swift Guide to Concurrency - NSOperations'
date: 2016-05-10 19:30:34
tags: [tutorial, swift, ios, nsoperation, nsoperationqueue, concurrency, asynchronous]
categories: ios
featured_image: 'https://flyingmoose.co/assets/posts/2016-05-10-swift-guide-to-nsoperations/nsoperation-featured.png'
lead_text: 'Learn how to use NSOperation and NSOperationQueue in Swift to bundle tasks in your app.'
---

Lately I've been trying to refactor the codebase in my [Swipe app](https://flyingmoose.co/swipe), and one area that stood out in particular were my asynchronous network calls. Feeling inspired by one of [Ray Wenderlich's awesome RWDevCon 2016 talks](https://www.raywenderlich.com/store/rwdevcon2016-vault), I realized this might be a great place for some [NSOperationQueues](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/NSOperationQueue_class/). 

### Async Madness

If you've spent anytime dealing with asynchronous network calls in iOS, you're likely very familiar with completion handlers. Completion handlers allow you to initialize some long running request (In this case a network call) without waiting around for it to finish, and in the process blocking the current thread. 

This is great if you want to say make an API request on a background thread, and then on completion of that request grab the main queue to do some UI updates. Unfortunately it can quickly make your code super unreadable if you have multiple asynchronous requests that depend on the previous request, like this: 

{% highlight swift %}
static func refreshAuthToken(completion:(RedditOauthStatus) -> Void) {
		// Make call to refresh OAuth token
        KeychainAccess.SwipeitKeychain.refreshAuthToken({
            status in
            switch status {
            case .Success(let token):
                RedditOauth.sharedInstance.authToken = token
                
                // Make call to retrieve latest profile info for user 
                Alamofire.request(RedditAPI.Router.PullProfile)
                .responseJSON { response in
                    guard response.result.error == nil else {
                        return
                    }
                    if let responseJSON = response.result.value! as? [String:AnyObject] {
                        let json = JSON(responseJSON)
                        
                        ...
						
						completion(RedditOauthStatus.Refreshed)
                		return
                    }
            }
            ...

{% endhighlight %}


There *must* be a better way!



### Enter NSOperationQueue
 
 
An *NSOperationQueue* is made up of an array of, you guessed it, *NSOperation*s. At it's most basic level, you can think of every NSOperation as a single task that needs to be run. An NSOperationQueue is in charge of managing the current state of each NSOperation in it's queue, and determining how to go about running the operations.

The above code snippet shows a sequence of network calls to be made when refreshing a user's OAuth token. First, I actually refresh the token. Then, I make a call to update that user's profile info with the refreshed auth token from step one. Even though this is a simplified version of what actually happens after login, it's still pretty messy and is a great candidate to be broken up into it's own NSOperationQueue. 


### Creating the NSOperations

Now that we've seen the two network calls that need to be made as part of the authentication process, let's break out each call into it's own NSOperation. [According to the NSOperation documentation](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/NSOperationQueue_class/), there are four methods/properties we need to override when creating our asynchronous NSOperation subclass:

* start()
* asynchronous
* executing
* finished 


Let's start with the refreshing auth token piece: 

{% highlight swift %}
class RefreshTokenOperation: NSOperation {
    private var _executing = false
    private var _finished = false
    
    override internal(set) var executing: Bool {
        get {
            return _executing
        }
        set {
            willChangeValueForKey("isExecuting")
            _executing = newValue
            didChangeValueForKey("isExecuting")
        }
    }
    
    override internal(set) var finished: Bool {
        get {
            return _finished
        }
        set {
            willChangeValueForKey("isFinished")
            _finished = newValue
            didChangeValueForKey("isFinished")
        }
    }
    
    override var asynchronous: Bool {
        return true
    }
    
    override func start() {
        if cancelled {
            finished = true
            return
        }
        
        executing = true

        dispatch_async(dispatch_get_global_queue(QOS_CLASS_UTILITY, 0)) {
            KeychainAccess.SwipeitKeychain.refreshAuthToken({ status in

            switch status {
            case .Success(let token):
                RedditOauth.sharedInstance.authToken = token
                self.executing = false
                self.finished = true

             ...
        }
    }
}
{% endhighlight %}

*The getters and setters are required to maintain KVO compliance for our NSOperation subclass*


The important thing to note here is that we're required to update our **executing** and **finished** properties manually, otherwise the refresh queue we'll create won't know to move onto the next operation. 

Now let's create the NSOperation to pull the updated profile info for our user: 

{% highlight swift %}
class UpdateProfileInfoOperation: NSOperation {
    
    ... 
    
    override func start() {
        if cancelled {
            finished = true
            return
        }
        
        executing = true

        dispatch_async(dispatch_get_global_queue(QOS_CLASS_UTILITY, 0)) {
            Alamofire.request(RedditAPI.Router.PullProfile)
                .responseJSON { response in
	                ...
					
	        		self.executing = false
	        		self.finished = true
            }
        }
    }
}
{% endhighlight %}



**UpdateProfileInfoOperation** is identical to our **RefreshTokenOperation**, with the exception of the *start()* method which is responsible for the actual work being performed by the operation. 


### Adding dependencies

One of the awesome features of NSOperations are their ability to mantain dependencies. This allows us to perform our asynchronous tasks consecutively, without relying on the messiness of nested completion handlers.

To add our **UpdateProfileInfoOperation** as a dependencies of our **RefreshTokenOperation** operation, we just have to add the following: 

{% highlight swift %}
//Get instances of both our operations
let refreshOperation = RefreshTokenOperation()
let updateProfileOperation = UpdateProfileInfoOperation()

updateProfileOperation.addDependency(refreshOperation)

{% endhighlight %}

Now the *updateProfileOperation* won't run until the *refreshOperation* finishes. 

To run either of these tasks individually, all we would have to do is call their *start()* method and it would kick off our task. But to really benefit from our updated approach we need to add them to our own NSOperationQueue. 


### Creating our NSOperationQueue

Compared to creating our NSOperations, the NSOperationQueue is a piece of cake. 

First we need to create an instance of NSOperationQueue. After that all we need to do is call *addOperation* on our instance and pass in each NSOperation we want on the queue, our operations will automatically start after being added to the queue. 

{% highlight swift %}
//Get instances of both our operations
let refreshOperation = RefreshTokenOperation()
let updateProfileOperation = UpdateProfileInfoOperation()

updateProfileOperation.addDependency(refreshOperation)

let authenticationQueue = NSOperationQueue()
authenticationQueue.addOperation(refreshOperation)
authenticationQueue.addOperation(updateProfileOperation)

{% endhighlight %}


There are many ways to tweak NSOperationQueues to become more powerful (including limiting number of concurrent operations, specifying priority of operations, etc.), but this is the most basic implementation.  


Just like that we were able to refactor the messy nested async code into the three lines you see above, while at the same time making our code more modular for us to use with future projects. 


**And that's all there is!**

Let me know if you have any questions/suggestions either by tweeting me [@FlyingMooseDev](https://twitter.com/FlyingMooseDev) or sending me an email [cam@flyingmoose.co](mailto:cam@flyingmoose.co)
