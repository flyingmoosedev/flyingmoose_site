---
layout: inner
title: 'Creating Custom Error Pages for Apache Server'
date: 2016-05-12 19:30:34
tags: [tutorial, apache, web development, custom error page, ubuntu, security, how to]
categories: web
featured_image: 'https://flyingmoose.co/assets/posts/2016-05-12-creating-custom-error-pages-with-apache/localized-error-conf.png'
lead_text: 'Learn how to create your own custom error page for an Apache 2 web server.'
---

If you're running your own (or your company's) site, chances are pretty high that you're using an Apache server. [In fact, Apache's web server powers over half of all websites worldwide](http://w3techs.com/technologies/overview/web_server/all). Because of Apache's popularity, you've no doubt run into a screen like this before: 

{::options parse_block_html="true" /}

<img class="img-responsive" src="{{ site.data.global.url }}/assets/posts/2016-05-12-creating-custom-error-pages-with-apache/apache-error.png"/>


While this might not seem like a big deal (other than being annoyed at broken links..), it's actually displaying a lot more info than you probably want. After seeing this error message, any would-be hacker knows the exact version of Apache you're using and [can use this info to find security flaws to gain root access to your site](https://pentesterlab.com/exercises/cve-2007-1860/course). 

One way around this is to create your own custom error page that doesn't display this info so obviously.

### Creating your custom error page

The first step in adding a custom error page to your site is to actually build the page. If you wanted a super simple "Not Found" heading to appear, you could just create an HTML file using this: 

{% highlight html %}
<!doctype html>
<html>
    <body>
        <h1>Not Found</h1>
    </body>
</html>
{% endhighlight %}

Of course you can also make your own version more visually appealing, the key here is that you're not making your OS/server info quite so public. 

### Editing your Apache configuration

Now that you have your custom error page coded (referred to as **error.html** from here on), you'll need to transfer that to the home directory of your server. If you haven't changed it manually, the default directory is usually **/var/www/html/**. 


The next step is editing your Apache *localized-error-pages* file to use **error.html** instead of the default error page. 

### Editing your Localized-Error-Pages configuration

You can find your localized-error-pages configuration file in the **/etc/apache2/conf-available/** directory. The file will be called *localized-error-pages.conf* and should look like this: 


{::options parse_block_html="true" /}

<img class="img-responsive" src="{{ site.data.global.url }}/assets/posts/2016-05-12-creating-custom-error-pages-with-apache/localized-error-conf.png"/>


As you can see there are multiple different types of error pages that can be overridden. If you just want to use your new custom error page for *404* (Not Found) errors, uncomment line 8 that starts with "ErrorDocument 404" and add **"/error.html"** instead of the default location for that error page. You can simply repeat this for each type of error you want to override, or feel free to create different custom error pages for each type of error. 

We're almost done, the last step is to restart your Apache server in order for the changes to take effect: 

{% highlight bash %}
sudo service apache2 restart
{% endhighlight %}



**And that's all there is!**

If you want to test out your new custom error page, try to navigate to a page of your site that doesn't exist to see it in action (ex. www.example.com/foobar). 


Let me know if you have any questions/suggestions either by tweeting me [@FlyingMooseDev](https://twitter.com/FlyingMooseDev) or sending me an email [cam@flyingmoose.co](mailto:cam@flyingmoose.co)
