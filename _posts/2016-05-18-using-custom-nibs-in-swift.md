---
layout: inner
title: 'How To Use Custom Nibs in Swift 2'
date: 2016-05-18 19:30:34
tags: [tutorial, ios, swift 2, nib, xib, storyboard, UIView, how to]
categories: ios
featured_image: 'https://flyingmoose.co/assets/posts/2016-05-18-using-nibs-in-swift/add-nib-view.png'
lead_text: 'Learn how to design reusable UIViews visually using Nibs.'
---

When I started doing iOS programming, there were a couple of views I wanted to layout visually in interface builder but didn't want to create an entirely new view controller for. Then I discovered NIB (or XIB now) files, but one thing that wasn't very clear to me was how to actually **use** them in my project. [The merits of creating views programatically vs. visually](https://www.toptal.com/ios/ios-user-interfaces-storyboards-vs-nibs-vs-custom-code) are well documented, but no matter what your preference you will likely need to know how to implement them at some point. 


### Creating a new NIB file 

The first step here is to obviously add a new NIB file to your project. NIB files are located under the iOS -> User Interface templates when you create a new file: 


{::options parse_block_html="true" /}

<img class="img-responsive" src="{{ site.data.global.url }}/assets/posts/2016-05-18-using-nibs-in-swift/add-nib-view.png"/>


Name your NIB file anything you want, and then proceed to add whatever UIButtons, UITextFields, etc. how you would in a regular storyboard file. 


The next step is to create a corresponding .swift file (subclass of UIView) to manage your NIB. 

**Note: your Swift file and your NIB file must be named the same thing**. 

After you create your new file, you need to set it as the NIB view's class like so: 


{::options parse_block_html="true" /}

<img class="img-responsive" src="{{ site.data.global.url }}/assets/posts/2016-05-18-using-nibs-in-swift/link-nib-class.png"/>


Make sure you've selected the top-level view before adjusting the custom class attribute. Next up is to create IBOutlets/IBActions like the below screenshot: 

{::options parse_block_html="true" /}

<img class="img-responsive" src="{{ site.data.global.url }}/assets/posts/2016-05-18-using-nibs-in-swift/link-iboutlets.png"/>

*This is an example that will be used in the upcoming login controller tutorial* 

### Instantiating your NIB view 

Now that you have your NIB view laid out and associated with your new view's class, it's time to actually instantiate your new view. This is the part that I got tripped up on when I was going through this process, but I found two ways to easily implement the new NIB-based views. 

### Option 1: Using NSBundle

This option is super simple, but not as "elegant" as the second option. To add a new instance of a NIB-based UIView, simply add the following to the presenting view controller: 

{% highlight swift %}

let loginScreen = NSBundle.mainBundle().loadNibNamed("LoginScreen", owner: self, options: nil).first as? LoginScreen
loginScreen?.frame = view.frame
view.addSubview(loginScreen!)

{% endhighlight %}

The important thing to note here is that because the NIB view isn't tied to any particular window size, you must set the size of it's frame before adding it to the view heirarchy. 


### Option 2: Adding a class func to your NIB view

The next option I got from [a Github thread discussing this very issue](http://stackoverflow.com/a/25513605). Basically, we move the initialization code from the ViewController to the NIB view itself. This allows us to reuse this same function in multiple view controllers without re-writing the same code again. (You could even make this into a protocol to maximize reusability..)

{% highlight swift %}

class func instanceFromNib() -> YourViewName {
    return UINib(nibName: "nib file name", bundle: nil).instantiateWithOwner(nil, options: nil)[0] as YourViewName
}

{% endhighlight %}

**And that's all there is!**


Let me know if you have any questions/suggestions either by tweeting me [@FlyingMooseDev](https://twitter.com/FlyingMooseDev) or sending me an email [cam@flyingmoose.co](mailto:cam@flyingmoose.co)
