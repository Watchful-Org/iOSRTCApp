

#2016 04 03 Setting up to run
Note that the app can be connected to two different peer servers, https://appr.tc or https://apprtc.appspot.com
To set this, see file appwindow.js for:
var roomServer = 'https://apprtc.appspot.com';
//https://appr.tc/
var roomServer = 'https://appr.tc/';

Note further webRTC samples at: https://webrtc.github.io/samples/

iOSRTC Cordova plugin: https://github.com/eface2face/cordova-plugin-iosrtc
Building instructions (which apply to the plugin and the demo):
https://github.com/eface2face/cordova-plugin-iosrtc/blob/master/docs/Building.md
NOTE: will not work with Cordova ios4, need to use ios@3.9.2  Here's How:
cordova platform remove ios
cordova platform add ios@3.9.2


iOSRTCApp: https://github.com/eface2face/iOSRTCApp
Based on
AppRTC demo: https://github.com/webrtc/apprtc


Current state of affairs...
Will successfully build on both Android and iOS.
Can make successful calls on both Android and iOS when using WiFi.
When trying to make calls using cell network, get error No TURN server, unlikely that media will traverse networks.
Tried to debug this but have not succeeded.

#Build commands:
cordova run android --device
cordova build iOS
then open in XCode and change settings per iOSRTC Cordova plugin: https://github.com/eface2face/cordova-plugin-iosrtc/blob/master/docs/Building.md
If you still prefer to do it manually open it with Xcode and follow these steps:

Set "iOS Deployment Target" to 7.0 or higher within your project settings.
Set "Deployment Target" to 7.0 or higher within the project target settings.
Within the project "Build Settings" add an entry to the "Runpath Search Paths" setting with value @executable_path/Frameworks.
Within the project "Build Settings" set "Objective-C Bridging Header" to PROJECT_NAME/Plugins/cordova-plugin-iosrtc/cordova-plugin-iosrtc-Bridging-Header.h (read more about the "Bridging Header" above).
Within the project "Build Settings" set "Enable Bitcode" to "No".


# Turn server issue
ICE config for STUN and TURN servers were not provided by server, preventing iOS app to work over cellular connection.
Config retrieved from http://www.html5rocks.com/en/tutorials/webrtc/infrastructure/ was added in appwindow.js.

Note, turn server is loaded at:
      sendAsyncUrlRequest('GET', roomServer + '/params').then(function(result) {
        var serverParams = parseJSON(result);
        console.log('serverParams:',serverParams);
Turn server can be specified at command line using apprtc params.

Note addition of Turn server to AppRTC
https://groups.google.com/forum/#!topic/discuss-webrtc/gmutygXnqS8

Performing a diff on appwindow.js between iOSAppRTC and AppRTC shows that there is an important difference, 

newParams.offerConstraints = parseJSON(serverParams.offer_constraints);
becomes in the updated AppRTC:
newParams.offerOptions = parseJSON(serverParams.offer_options);
this is then referenced in apprtc.debug.js


#AppRTC parameters (with clickable links available at: https://apprtc.appspot.com/params.html)
 
 A number of settings for the AppRTC video chat application can be changed by adding URL parameters.
 
 For example: https://appr.tc/?hd=true&stereo=true&debug=loopback
 
 The file using the parameters is apprtc.py. More Google-specific parameters are available from the MediaConstraints interface.
 
 For more information see AppRTC : Google's WebRTC test app and its parameters.
 hd=true	Use HD camera resolution constraints, i.e. minWidth: 1280, minHeight: 720
 stereo=true	Turn on stereo audio
 debug=loopback	Connect to yourself, e.g. to test firewalls
 ts=[turnserver]	Set TURN server different from the default
 audio=true&video=false	Audio only
 audio=false	Video only
 echoCancellation=false	Disable all audio processing
 googEchoCancellation=false	Disable echo cancellation
 googAutoGainControl=false	Disable gain control
 audio=googNoiseReduction=false	Disable noise reduction
 asc=ISAC/16000	Set preferred audio send codec to be ISAC at 16kHz (use on Android)
 arc=opus/48000	Set preferred audio receive codec Opus at 48kHz
 dscp=true	Enable DSCP
 ipv6=true	Enable IPv6
 arbr=[bitrate]	Set audio receive bitrate, kbps
 asbr=[bitrate]	Set audio send bitrate
 vsbr=[bitrate]	Set video receive bitrate
 vrbr=[bitrate]	Set video send bitrate
 opusfec=false	Turn off Opus FEC
 opusdtx=true	Turn on Opus DTX
 opusmaxpbr=8000	Set the maximum sample rate that the receiver can operate, for optimal Opus encoding performance


#Tracking screen
Somewhere in the app, it's possible to bring up a detailed tracking screen with real time information about the webRTC parameters.
This as come up sometimes while the app was running.  Not sure how to get this to happen.  Looks super useful.

#Infobox
The infobox code puts up detailed information about the webRTC session, state, real time analytics data, etc.

# 2016_04_07
Verified that on Android, you do NOT need to delete the app locally on the phone and do NOT need to remove the android platform directory to deploy new code.
cordova run android does the trick, and launches the new app (even if the old one was running) with the code changes.

