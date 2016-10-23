---
title: Http/Https call interception for MobileApps
date: 2016-10-09 23:01:05
tags:
- mobile
- testing
---

Many a times while doing mobile testing I(we) always wanted to analyze what all http/https call are happening in the background or may be just to tweak the response a bit to verify some scenarios.

One example of such use case would be to verify display of name if the length of characters are more than 35 etc.

Or verify analytics data.

In my endeavour to reduce manual task of
1. Picking up mobile
1. Setting proxy on the device(though this can solved using arpspoof but not in scope of this blog)
1. Starting zap proxy on my system
1. Setting the IP, installing certificate on each device.

I searched for few suggestion and tools which can perform just the task needed for the moment.

The exact problem statement which i was trying play around with was mimicking network error on the fly to showcase to a dev a bug.

if we just want to see what all things are happening behind the scene in mobile then easiest of all is

**Android**:`adb logcat`.  
**iOS**: Attach console or use [ios-sim](https://github.com/phonegap/ios-sim).

Though logs are pretty straight forward but tweaking or changing response is not possible.
And again you need to attach your device to laptop in order to get logs.

So the best bet at this time was configuring proxy in mobile/emulator.

## Emulator/Simulator instead of Devices

As we are interested in testing http/s traffic here using real device is too much to ask for, moreover havent encountered any functional anomaly which is present on real device and not able to replicate on emulator/Simulator.

Difference between simulator and emulator is nicely explained in answer over [here](http://programmers.stackexchange.com/questions/134746/whats-the-difference-between-simulation-and-emulation).

In order to start emu/simulator and using proxy we need to configure one.

 iOS simulator uses the OSX network setting to proxy traffic, so it can be achieved refer [here](http://blog.daanraman.com/coding/using-the-iphone-simulator-with-an-https-proxy/)

 Starting emulator with http-proxy to my surpise was also very simple.
 `emulator -avd Nexus_5_API_22 -http-proxy xx.xxx.xx.xx:8080`

### Quest for Proxy server

There are many decent proxy-server available like [charles proxy](https://www.charlesproxy.com/). If you like GUI based tools then go for it and it is paid tool.

Then you have Owasp [ZAP](https://www.owasp.org/index.php/OWASP_Zed_Attack_Proxy_Project) proxy which can used to intercept traffic and is open sourced but yet again it does lot of other things as well(related to security) and as i said before demand of situation was simple easy to use.

**mitmproxy**
if you haven't heard of it go checkout there [website](https://mitmproxy.org/). Its built with python and one of the brilliant and easy to use commandline tool.

with mitmproxy, it very easy to edit response/request, save them replay etc and more over ssl certificate installation is pretty smooth.

mitm by default runs on 8080 port and there are other nice tools which works with mitproxy like mitmdump.


### Certificate for https

Now only pending task is installing certificate for intercepting https traffic.  

**With mitm it is as easy as visiting `mitm.it` and selcting platform.** certificate will be installed automatically.


For zap proxy installation of certificate is bit tricky with probably following solution.
- Share certificate in email and open the email in emulator/device and download -> install certificate.
- On mac mount the sd card `hdid ~/.android/avd/Nexus_5_API_22.avd/sdcard.img` and copy the certificate in location of your preference open folder in emulator/device and install certificate.  


So in order to conclude, clear winner over here is emu/simulator with mitm proxy. Do share your thoughts in the comment section.
