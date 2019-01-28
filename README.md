# WEBRTC plug-in Sample for RICOH THETA

This article was [originally published](https://qiita.com/shrhdk_/items/28b1e502e5a68ec2a906) in Japanese by [Hideki Shiro](https://qiita.com/shrhdk_) (@shrhdk_ on Qiita).

# Introduction

Hello, @shrhdk_ of Ricoh.

I will introduce a plug-in that uses WebRTC to trigger the shutter of a RICOH THETA V while remotely checking the 360° image of RICOH THETA V in real time.  

The plug-in is a result of trying to build a web-based application that provided the remote shutter function of the THETA standard application using WebRTC only.

![graphic-1|690x241](https://community.theta360.guide/uploads/default/original/2X/6/6eaa09e26d6a55c2251d28b457e152cb8a7ab92f.png) 

## Source Code

https://github.com/ricohapi/theta-plugin-webrtc-sample

# What is WebRTC?

First of all, WebRTC is a technology for exchanging data such as audio and video in real time on a Web browser.

It is open source by Google, standardization of related protocols by IETF and W3C browser compatible API is being done.

Since the source code, build tools, and official build libraries are distributed under the BSD license under the WebRTC library, development of native applications is also possible.

For details, please refer to the official website. https://webrtc.org/

# WebRTC P2P connection flow

WebRTC P2P connection is roughly divided into 3 steps.

First of all, exchange session information (SDP). Roughly speaking, "When sending a picture - Codec can use H.264 and VP9!" "Then, I will exchange m (__) m asking for H.264".

Next, exchange communication path candidate (ICE Candidate). Roughly speaking, "I can connect on port 1111 of IP address x.x.x.x" "This is port 2222 of y.y.y.y".

Finally, we will establish a P2P connection using the exchanged information and send and receive the video decided by session information exchange.

A server that mediates exchange of session information and exchange of communication path candidates is called a signaling server.

Actually, this exchange method is not specified and is free. Extreme story, exchanging verbally ... Entering with hands ... it is ok too. In fact it is common to exchange using WebSocket.

![graphic-2|690x315](upload://fMYPk2s8lgdJqaKvmQjXIkvAHzV.png) 


# Building the Plug-in 

Although the introduction has become longer, let's build the sample and run it. Those who want to know only the overview can skip this section.

## Set THETA's Wi-Fi to client mode

In order to use the plug-in developed this time, it is necessary to keep THETA's Wi-Fi in client mode. Live mode with the WebRTC of the essence does not work when it is in AP mode.

For details on how to put THETA into client mode, please see this video. 

https://www.youtube.com/watch?v=jE9_VqaVuLM

## Enable Wi-Fi during USB connection

This plugin accesses and uses THETA via Wi-Fi.

However, in THETA as it is it is inconvenient because you can not check the Web UI while running the plug-in in Android Studio because Wi-Fi will be invalid if you connect USB.

## Download and Build Plugins

First, download the project from https://github.com/ricohapi/theta-plugin-webrtc-sample.

Once you have downloaded it, import the project in Android Studio. Please import the folder with `build.gradle`.

![image|690x428](upload://8vlFIr7H7TegLqhGJUJvKYNEa5h.png) 

![graphic-4|690x434](upload://mxQ4apikrlBdDBMAwKouA4QEmsA.png) 

Introduction It takes time to install tool chains and libraries. Please wait a while.

When the import is completed, please connect THETA V with debug mode enabled and press execute button on tool bar. The build and execution of the plug-in begin.

![image|440x164](upload://oG34fGppAUpcNA2OB9WwaFpdfye.png) 

If the following error occurs during the build, install the tool chain according to the error message.

![image|343x41](upload://x2bNJSRko7vW7VbtuxsgknvwTq5.png) 

When the build succeeds and the plug-in is executed, the sound will beeping from THETA and it will end immediately. This is because the authority is not set in the plugin, so grant the privilege from the application settings. 

![image|529x360](upload://ze3BtbdbBZ49VEuXGuhX7h5Tpe8.png) 

You can also set it with the following command.

```shell
$ adb shell pm grant com.theta360.pluginapplication.webrtc.sample android.permission.CAMERA 
$ adb shell pm grant com.theta360.pluginapplication.webrtc.sample android.permission.RECORD_AUDIO
```

Permission setting is necessary only for the first time. Also, when actually distributing with plug-in store, it is set to be automatically set.

After setting, let's run it again from the toolbar.

![image|440x164](upload://oG34fGppAUpcNA2OB9WwaFpdfye.png) 

This screen will appear when you start up.

![image|529x452](upload://3x4HFlBgpMm5rO7oJQ0uDUbSFaV.png) 

In this example, the HTTP server is running at 192.168.1.5:8888. The IP address depends on the environment.
Try using the application

Once the plug-in is activated, you access http://192.168.1.5:8888 from the web browser of the terminal on the same network as THETA. The IP address depends on the network environment.

This time I visited from Google Chrome on Nexus 5X. If access is successful, the following page will be displayed.

![image|281x500](upload://99LjLfGOdI12siWrA1FUnR40Ixu.jpeg) 

Because the room is dirty it is mosaicing, but in fact the clear live view is displayed.

With this, the operation confirmation is successful! You can set options like the standard application or turn off the shutter.

# Plugin configuration

I will introduce the configuration of the plug-in made this time. The light blue block is the plug-in, the yellow block is the part we implemented this time.

As a feature, I have a signaling server (WebSocket server) and a web server in the plugin.
You can use it as a web application by directly accessing THETA without an external server.

![graphic-5|690x276](upload://t52uNh7MVZVJemIfw3OOF9IPX99.png) 

# Signaling server

As mentioned above, the signaling server exchanges WebRTC session information (SDP) and communication path candidate (ICE Candidate).
Actually, it is a simple WebSocket server. It is designed to broadcast received messages to sender.

I use a library called [Java WebSockets](https://github.com/TooTallNate/Java-WebSocket).

The implementation is in [SignalingServer.java](https://github.com/ricohapi/theta-plugin-webrtc-sample/blob/master/app/src/main/java/com/theta360/pluginapplication/webrtc/sample/network/SignalingServer.java).

# Web server

The Web server distributes application files (HTML / CSS / JavaScript) to run on a Web browser. It is also responsible for accepting commands such as changing shooting settings and shutter operation.

I use a library called [AndroidAsync](https://github.com/koush/AndroidAsync).

The implementation is in [WebServer.java](https://github.com/ricohapi/theta-plugin-webrtc-sample/blob/master/app/src/main/java/com/theta360/pluginapplication/webrtc/sample/network/SignalingServer.java).

The application file (HTML / CSS / JavaScript) is put in the assets folder and is included in the APK.
In addition, we also include settings.json which is a setting file proprietary to THETA to start the Web server.
The commentary on this area is detailed in the following articles.

[How to implement Web UI of THETA plug-in](https://qiita.com/3215/items/d156307dfadd7dc2fbf6). (Japanese). Note: English version will be published soon. Please check this site.

# WebRTC library

I use [google-webrtc](https://bintray.com/google/webrtc/google-webrtc) as a WebRTC library. Generation of session information (SDP), video encoding / decoding, streaming are processed in this library.

THETA cameras require some unique configuration, so we added extensions for THETA to set them.

The circumstances around here and how to use the WebRTC library are explained in the following article.

[Deliver 360 ° video in real time with THETA V alone - Qiita](https://qiita.com/shrhdk_/items/fa7bb0feab443e3037e8)

# THETA basic application

It is an application that realizes the function as ordinary THETA, not a plugin. It is also a server of [THETA API v2](https://developers.theta360.com/en/docs/v2.1/api_reference/) (the web API rather than the plugin API).

In this time, by calling the THETA API v2 from the plug-in, we can realize the shooting setting change and the shutter operation.

## MainActivity

MainActivity controls peripheral blocks. This is the entry point of the plugin. It sends and receives session information, mediates shooting settings, displays IP address and operation status on the screen.

## Application (front end)

The front end is a basic configuration of HTML + CSS + JavaScript, and the JavaScript part is also rustic using jQuery. [Swiper](https://idangero.us/swiper/) is used to implement touch operation slider.

It is responsible for providing GUI and exchanging with the signaling server, sending commands to the Web server, and controlling WebRTC.

# Summary

* I created a shooting application plug-in with live view function.
* By using WebRTC, it realized as a web application that runs on the browser.
* It enabled to run without external server by using P2P mode.
