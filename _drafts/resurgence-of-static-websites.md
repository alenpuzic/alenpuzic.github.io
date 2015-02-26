---
layout:     post
title:      Resurgence of Static Websites
date:       2015-02-20 18:00:00
summary:    A look into numerous benefits of static websites and their drawbacks, along with potential solutions.  
categories: web architecture
---

Everything old is new again. In the recent years we've seen a resurgence in use of statically generated websites. 
There's a number of legitimate reasons leading this trend. Software security vulnerabilities, scalability limitations, 
high prices and time consumed by development/maintenance are only some of the bullet points in a long list of 
disadvantages driving people away from dynamic web development. There's also the overkill factor. Blogs, 
landing pages, even certain web applications simply do not require any of the advanced functionality offered by 
dynamic web platforms. Sometimes less is more. 

With that said, static sites are not the holy grail. They've not come back to rapidly take over the world and
and stake their claim as king of the web. They are just another recycled paradigm (think NoSQL databases)
that is proving to be very useful in solving the problems of today. With the incredible improvements in client side
development that we've seen in the past 5 years came a shift in how we think about web development. For a modern website
a backend is no longer a necessity, at least not in the way we've traditionally thought of it. I believe that with the
now ubiquitous everything-as-a-service pattern we can implement just about any web application as a combination of 
static files and RESTful web services. 

In the rest of this article I'll discuss in some detail the advantages of going with a static site, as well as 
some limitations and possible workarounds. Also, I will assume your static files are hosted
on a content distribution network (CDN). You certainly don't have to use a CDN but you probably should.
  
### Advantages

Going static brings about some pretty substantial benefits.

##### Security
  
It's not hard to imagine the security advantages of switching to a static site. You no longer have to worry about the 
security of the full solution stack (OS, web server, database, programming language). You don't have to manage any
servers, physical or virtual. No operating systems to keep patched. No web servers to secure and maintain. No hosted 
databases to worry about. And finally, you don't have to spend time developing or maintaining a codebase. Many of the
most common website security risks that keep us up at night are eliminated:

  * URL manipulation attacks
  * Path traversal attacks
  * Input validation attacks
  * SQL injections
  * Buffer overflows
  
Not to mention that other types of attacks, such as DDoS, become significantly more ineffective. 
  
##### Performance & Scalability

Granted, with correct usage of load balancing technologies, clustering and proper programming paradigms one can
build a __very__ scalable site. The problem is that such an endeavor is very costly both in terms of resources
as well as time. Just because _you can_ build something does not mean you should. Time is the
most limited resource we have and we should strive to manage it wisely. With static sites we don't need to worry about
scalability. If you're using a CDN then your content will automatically be distributed as efficiently as possible to
users in all parts of the world. Sure, different CDNs perform differently due to many variables but for the most part
they're all reasonably fast. The uptime for the most CDNs is usually pretty stellar as well.

##### Cost

You can host a static site on a CDN for less than a few dollars per month and still be able to handle large 
traffic spikes without a hiccup. 

Here's some example pricing from Amazon Web Services that should give you an idea of what to expect:
 
  * S3 data storage: $0.0300 per GB
  * CloudFront - Data out to Internet: $0.085 per GB (USA Only)
  * CloudFront - 10,000 HTTP Requests: $0.0075 (USA Only)
  * Route53: $0.400 per million (DNS) queries
  
Other providers have similar pricing models. You can also host a static site on GitHub Pages, for __free__, though 
you'll be missing out on some basic functionality such as SSL support. 

##### Platform Lock-In

Lock-ins occur for any number of reasons. Perhaps you started your website on a proprietary platform and there's
no pain-free path for data migration. Or, maybe your site relies too much on one particular technology and the cost 
of migrating to something newer is simply too high. We've all been there. With static sites you're not locked in to 
any particular platform, because all you're really working with are flat HTML/JS/CSS files. You can host the files 
on your own server or you can use any number of commercially available CDNs, and you can switch between them 
painlessly at any given moment.


### Drawbacks and Solutions

With everything great that can be said about static websites there are still a number of drawbacks. Fortunately, a large
portion of those can be addressed with new tools and methodologies. 

##### Requires basic web development knowledge

It is true that static sites usually require some knowledge of HTML, CSS and JavaScript. However, I feel like that 
requirement has all but disappeared. Today we have dozens of extremely easy to use and very capable static site 
generators. Check out any of the following projects to see what these tools can do for you:

  * [Jekyll](http://jekyllrb.com)
  * [MiddleMan](https://middlemanapp.com)
  * [Pelican](http://blog.getpelican.com)
  * [Nikola](http://getnikola.com)
  * [Hugo](http://gohugo.io)
  * [Hyde](http://hyde.github.io)
  * [OctoPress](http://octopress.org)

Any of these applications will help you setup a website in minutes and with the huge number of themes available for each one
you'll likely never have to mess around with HTML or CSS directly. There are hundreds, if not thousands, of themes and
templates available for both personal and commercial purposes.  

##### Increased Code Exposure

With a purely static site your application code usually resides in JavaScript files and will therefore be exposed to the 
world. For most people this is usually not an issue. However, if you're worried about keeping proprietary code secure 
then you may want to consider alternative approaches. You'll probably want to design your website with a 
hybrid approach in mind, where most of the user facing code is static while the proprietary algorithms are tucked 
away on a secure sever, accessible only via a RESTful API. A lot of the clients I've worked with have designed 
their web applications in such a manner.

There are also solutions out there that will obfuscate, scramble or encrypt your JavaScript code. However, based on my 
experience there is no real way of keeping your source code secure. One way or another somebody will be able to get it.

##### Limited Functionality

A lot of people think that with a static site they have to forgo all dynamic functionality. That is simply not true. 
Instead, you have to shift the way you think about web development. Just about any feature that you could think of on a 
dynamic web platform can be implemented on a static website with a few JavaScript libraries and some 3rd party services.
With JavaScript we can have fully featured comment system on our blogs, authentication service for our users, 
payment processing, even a fully managed remote database with access controls. 

Here's a short list of some free and commercial services that allow you to augment the functionality of your 
static site:

  * [Disqus](https://disqus.com) JavaScript powered comment system
  * [FormSpree](https://formspree.io) HTML contact form
  * [FireBase](https://www.firebase.com) Data storage API and much more
  * [Stripe](https://stripe.com) Card/payment processing API service

There are many more services available besides the ones I listed here. 

### Conclusion

With the incredible growth in web usage we've been forced to come up with new and innovative ways of building
highly scalable websites that can handle millions of users and frequent spikes in traffic. One of the ways this has 
manifested itself in is the advent of new client-side technologies that enable us to migrate more and more web 
application functionality away from the backend. 

These technologies enable us to build static websites that are leaps and bounds beyond what we could even imagine to build
just 5 years ago. Sometimes we can't implement everything we need with only a static site. There might be functionality 
that requires us to utilize the power and flexibility of a Python/Java/PHP/etc web framework. But we shouldn't always 
jump towards a dynamic stack whenever we need to design something. A lot of the times less is actually more. 

 

