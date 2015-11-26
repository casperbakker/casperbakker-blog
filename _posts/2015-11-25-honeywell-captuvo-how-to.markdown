---
layout: post
title: Honeywell Captuvo, how to get it to work
date: 2015-11-25 14:04:03
description: After years without a personal blog, I thought it would be fun to start a blog again.
categories: personal
---
<img src="/blog/images/honeywell-captuvo.jpg" class="photo photo-right photo-small photo-nice" alt="Honeywell Captuvo in action">For [Picqer](https://picqer.com) we are experimenting with barcode scanner sleds. (You can connect it to an iOS device and use it in warehouses instead of old and expensive Windows CE devices.) After reviewing the Honeywell Captuvo's and the Linea Pro's, we decided to try the Captuvo's first.

## Easy SDK
The SDK looked very easy to use. Drop in 2 libraries, write 5-6 lines of code and it should work.

But after a day of trying I could not get it to work. Also my colleague @[stephangroen](https://twitter.com/stephangroen) ran into the same problems the day after. We could retrieve the serial number and battery status. But after we 'start' the barcode scanner, the lights stay out.

So frustrating.

## The missing plist entry
At the end of the second day I found an abandoned forum with an extra entry to the Info.plist. Looked like exactly something we were missing. We tried it in 2 minutes and now it worked. And it worked flawlessly.

The missing entry, add this to your main Info.plist:
{% highlight xml %}
<key>UISupportedExternalAccessoryProtocols</key>
<array>
    <string>com.honeywell.scansled.protocol.decoder</string>
</array>
{% endhighlight %}

The plist entry is never mentioned in the docs. Also the docs are too much text for some really easy steps. So perfect for this first blog post to create a comprehansive how-to. :)

## Complete how-to
These are the minimum steps to get the Honeywell Captuvo to work in your iOS project:

- Get the Captuvo SDK from [honeywellaidc.com](https://www.honeywellaidc.com/HoneywelliOS/Developer-HSM-SDK.html), you can download it directly after you fill in the form
- Drag the *Captuvo.h* file into the file tree on the left in Xcode (in the pop-up check 'copy items if needed' and all targets)
- Do the same with the *libCaptuvoSDK.a* file
- Go to your project settings (click the first item in the file tree), click on the General tab, at *Linked Frameworks and Libraries* add *ExternalAccessory.framework*
- Open the *Info.plist* and add "Supported external accessory protocols" key as an array with item 0 as string with the contents "com.honeywell.scansled.protocol.decoder"
<img src="/blog/images/honeywell-missing-plist.png" class="photo photo-nice" alt="The missing Captuvo plist entry">
- Open *AppDelegate.h* and add '#import "Captuvo.h"' at the top and add "CaptuvoEventsProtocol" as extra delegate, like this:
{% highlight objc %}
#import <UIKit/UIKit.h>
#import "Captuvo.h"

@interface AppDelegate : UIResponder <UIApplicationDelegate,CaptuvoEventsProtocol>

@property (strong, nonatomic) UIWindow *window;
@end
{% endhighlight %}

- Open *AppDelegate.m* and add the following code:

{% highlight objc %}
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    [[Captuvo sharedCaptuvoDevice] addCaptuvoDelegate:self];
    [[Captuvo sharedCaptuvoDevice] startDecoderHardware] ;
}

-(void)decoderDataReceived:(NSString *)data{
    UIAlertView* alert = [[UIAlertView alloc]initWithTitle:nil
               message:data
              delegate:nil
     cancelButtonTitle:@"OK"
     otherButtonTitles:nil];
    [alert show];
}
{% endhighlight %}

Now you can build your app, put it on a iOS device and it should work. You now get a UIAlert for every barcode you scanned.

Hope this helps the next developer wasting a day :) If you have any questions regarding this, ping me on twitter: @[casperbakker](https://twitter.com/casperbakker)