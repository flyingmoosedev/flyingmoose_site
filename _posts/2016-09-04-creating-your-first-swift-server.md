---
layout: inner
title: Creating your first Swift server
date: 2016-09-04 13:42:12
tags: [tutorial, swift, server, web development, heroku, swift-3, vapor]
categories: web
featured_image: 'https://flyingmoose.co/assets/posts/2016-09-04-creating-your-first-swift-server/vapor-splash.png'
lead_text: Learn how to bring your Swift 3 code to the server-side.
---

Since Apple's announcement that Swift would be made open source, there has been a lot of talk in the Swift community about the potential for server-side Swift code. In this tutorial, I'll show you how easy it is to create your first Swift server using Swift 3 and the Vapor framework.

[Vapor](http://vapor.codes/) is on of the most "Swifty" web frameworks out there for Swift, which is one of the reasons why I prefer it over other competitors such as [Perfect](https://github.com/PerfectlySoft/Perfect) or [Kitura](https://developer.ibm.com/swift/kitura/).


### Getting started: Setting up your environment (macOS Sierra)
<br>
To demonstrate the flexibility we have in creating our Swift server, I will be using the excellent [Atom](https://atom.io/) cross-platform editor instead of Xcode. If you want to follow along, I highly recommend installing the [Swift-Debugger](https://atom.io/packages/swift-debugger) package which extends familiar Xcode features such as breakpoints to the Atom editor.

{% highlight swift %}
// Install Swift packages via Atom's Package Manager
apm install swift-debugger language-swift
{% endhighlight %}

For this tutorial, we will be writing our server using the *2016-08-18-a* Swift 3 snapshot. While you can certainly develop our server on a linux OS such as Ubuntu, this guide will show the steps used on macOS Sierra.

If you don't already have [Swiftenv](https://github.com/kylef/swiftenv) installed, you'll need to do the following:
{% highlight swift %}
//Clone Swiftenv repo
git clone https://github.com/kylef/swiftenv.git ~/.swiftenv

//Add Swiftenv to your Bash profile **NOTE: Ubuntu uses ~/.bashrc instead of ~/.bash_profile**
export SWIFTENV_ROOT="$HOME/.swiftenv"
export PATH="$SWIFTENV_ROOT/bin:$PATH"
eval "$(swiftenv init -)"

//In a new terminal window, verify Swiftenv was successfully installed
swiftenv --version

{% endhighlight %}

If all goes well, you should see an output like this:
{::options parse_block_html="true" /}

<img class="img-responsive" src="{{ site.data.global.url }}/assets/posts/2016-09-04-creating-your-first-swift-server/swiftenv.png"/>


Now that you've got Swiftenv setup, you'll need to download the latest Swift 3 snapshot. Type the following in your terminal:

{% highlight swift %}
//NOTE: This is the latest version as of 4 Sep 2016
swiftenv install DEVELOPMENT-SNAPSHOT-2016-08-18-a

//After this has finished, add the following to make this the default 'swift' snapshot
swiftenv global DEVELOPMENT-SNAPSHOT-2016-08-18-a

//To verify your environment has been setup correctly, enter the following:
curl -sL check.vapor.sh | bash
{% endhighlight %}

Almost there! The final piece you'll need to follow along is the [Vapor Toolbox](https://vapor.github.io/documentation/getting-started/install-toolbox.html), which adds command line functionality for things like deployment. Vapor's Toolbox can be installed by typing the following in your terminal:

{% highlight bash %}
curl -sL toolbox.vapor.sh | bash
{% endhighlight %}

<br/>

### Hello, world!

First thing's first, let's create a new Vapor project:

{% highlight swift %}
//Create a new Vapor project named 'Hello' in terminal
vapor new Hello
{% endhighlight %}

If successfully created, you should see a screen like this in your terminal window:
{::options parse_block_html="true" /}

<img class="img-responsive" src="{{ site.data.global.url }}/assets/posts/2016-09-04-creating-your-first-swift-server/vapor-splash.png"/>
<br/>

By calling **vapor new**, we create a new Vapor project with a bunch of boilerplate code which is very useful for understanding Vapor's project structure.

The **main.swift** file is the one that we'll be focusing on for the purposes of this tutorial, so let's head there and remove any existing code so we can start fresh.

Similar to **app** in the Express/Node.js framework, Vapor uses **Droplets** to manage things like registering routes, starting the server, and appending middleware. Let's create our droplet and first route:

{% highlight swift %}
//Required imports
import Vapor
import HTTP

//Create Droplet
let drop = Droplet()

//Create the 'hello' route
drop.get("hello") { request in
  return "Hello, world!"
}

//Start the server
drop.serve()
{% endhighlight %}

The code above creates a very simple web server with an endpoint of '/hello' (i.e. http://localhost:8080/hello) that will respond to a GET request with the text "Hello, world!".

Now that we have our server code finished, let's go ahead and build/run our server to test it out.

{% highlight swift %}
vapor build // will take a bit

vapor run serve //Start running the server at http://localhost:8080/hello
{% endhighlight %}

<br/>

**And that's all there is!**

Coming up, I'll show you how to deploy your new server-side Swift code in Heroku as well as go into more detail on routing and creating views for your server.

Let me know if you have any questions/suggestions either by tweeting me [@FlyingMooseDev](https://twitter.com/FlyingMooseDev) or sending me an email [cam@flyingmoose.co](mailto:cam@flyingmoose.co)
