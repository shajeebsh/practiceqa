---
title: "jQuery Custom Events"
date: 2012-09-20 02:29:06 
author: shajeebhameed
layout: page
slug: jquery-custom-events
status: publish
---

## [jQuery Custom Events](<http://dailyjs.com/2009/11/14/jquery-custom-events>)

14 Nov 2009 | By Ric Roberts | ![Tags](http://dailyjs.com/images/tag.gif) [jquery](<http://dailyjs.com/tags.html#jquery>) [events](<http://dailyjs.com/tags.html#events>) [tips](<http://dailyjs.com/tags.html#tips>) [ringo](<http://dailyjs.com/tags.html#ringo>)

jQuery makes it easy to bind and trigger custom events. Although it’s not immediately apparent from [the documentation](<http://docs.jquery.com/Events/bind>), creating custom events is simple and very useful. 

### Binding Custom Events

Events can be bound to DOM nodes like this: 
    
    
    $("#myElementId").bind("myCustomEvent", function(event) {
      alert("myCustom event happened!");
    });
    

### Triggering Custom Events

jQuery’s [trigger](<http://docs.jquery.com/Events/trigger>) function is used to instigate the event, passing the name of the event as a string like this: 
    
    
    $("#myElementId").trigger("myCustomEvent");
    

If you want to pass data to the handler you can use an [event object](<http://docs.jquery.com/Events/jQuery.Event>): 
    
    
    var myCustomEvent = jQuery.Event("myCustomEvent");
    myCustomEvent.usefulInformation = "some data";
    myCustomEvent.otherStuff = "some other data";
    $("#myElementId").trigger(event);
    

### Namespaces

Another cool feature of jQuery events is [namespacing](<http://docs.jquery.com/Namespaced_Events>), which supports triggering or unbinding specific groups of handlers without having to reference them directly. You can add a namespace when binding an event which can be referenced when you unbind or trigger it. It’s possible to use multiple namespaces at once. 
    
    
    $('.myCSSClass').bind('myEvent.myNamespace.myOtherNamespace', function(){alert('namespaced event')}); 
    $('.myCSSClass').trigger('myEvent.myNamespace');
    $('.myCSSClass').unbind('myEvent.myOtherNamespace');
    

If you specify a namespace when unbinding or triggering events, you need to match one of the bound namespaces (or alternatively, if no namespace is specified it matches everything). 
    
    
    $('.myCSSClass').bind('myEvent.myNamespace', function(){alert('namespaced event')}); 
    $('.myCSSClass').trigger('myEvent.myNamespace'); // mentioned explicitly: will trigger
    $('.myCSSClass').trigger('myEvent');  // not specified: will trigger 
    $('.myCSSClass').unbind('myEvent.differentNamespace'); // won't unbind
    

` `
