---
layout: post
title:  "Making singleton XPCOM js services"
subtitle: ""
date:   2012-02-13 18:49:00
categories: [mozilla]
---

Currently defining a new XPCOM component in javascript has no implications regarding createInstance. This means any code or add-on may instance your component multiple times, even if you intended to make it service-like.

Why is this such a big deal?

Let's suppose that your component lazily creates and caches a Storage connection, and some other code createInstance() your component everytime it needs to make an API call to it. This actually means the add-on will create a new connection each time, and this connection will survive till the component intended to kill it, that is usually on some profile shutdown notification, like profile-before-change. While this example is about Storage connections, any kind of resource may be easily leaked this way.

While cpp services have various macros to make singletons, until XPCOMUtils there was no standardization of javascript components, and you had to make your own singleton factory every time. From version 12, XPCOMUtils includes a new method to generate singleton factories.

How to use it? XPCOMUtils supports some ["magic" properties](https://developer.mozilla.org/en/How_to_Build_an_XPCOM_Component_in_Javascript#Using_XPCOMUtils) in the component, one of these is _xpcom_factory. From version 12, to make a singleton component, you can just:

{% highlight javascript %}
function myservice() {
}
myservice.prototype = {
  ...
  _xpcom_factory: XPCOMUtils.generateSingletonFactory(myservice),
  ...
}
{% endhighlight %}
