# StreamAMG Playback SDK for JavaScript Documentation

The StreamAMG Playback SDK for JavaScript enables developers to build user-interface based applications integrating with
a video player of choice and configuration.

You can use the SDK inside your Web and Mobile Javascript applications to obtain video player capability resulting in
an embedded video player inside a container with video player capabilities. You may choose and configure the video
player of choice with capabilities e.g. autoplay.

The goal of the SDK is to offer versatility and a range of tools for crafting a personalized multimedia encounter,
allowing you various methods to seamlessly incorporate a video player into your web application, enabling you to make
well-informed decisions regarding the most optimal integration approach.

## Latest SDK To Download

>The latest JavaScript SDK for Playback v1  is available here:
>[v1 Playback SDK](https://sdk.playback.streamamg.com/v1/playback.js)

## SDK Reference Documentation

>The latest JavaScript SDK for Playback v1 reference documentation is available here:
>[v1 Playback SDK Documentation](https://sdk-docs.playback.streamamg.com/v1/docs/index.html)

## Integrated Video Players Supported

Currently, these are the supported video players:

- [Bitmovin Player](https://bitmovin.com/video-player) 

## Video Player Supported Features

Currently, these are the supported video player features you may configure:

- autoplay      - videos automatically play one after another or back-to-back.
- muted         - the audio output of the video is muted.

These may be defined during [initialisation](https://sdk-docs.playback.streamamg.com/v1/docs/classes/Playback.html#initialize) of the player by passing in some optional player [options](https://sdk-docs.playback.streamamg.com/v1/docs/interfaces/PlayerOptions.html).

## Setting Up and Configuring the SDK

### Installing the SDK

The StreamAMG SDK for JavaScript provides a JavaScript API for Video Player Integration. A single Javascript file
called 'playback.js' is available for downloading into your application.

>Versions are available for every major and minor release e.g. v1, v1.23, v1.23.9 should you require a specific version.
> 
>Specific versions may be obtained from ```https://sdk.playback.streamamg.com/<version>/playback.js```
> 
>However, it is advisable to use the latest major version which is available [here](https://sdk.playback.streamamg.com/v1/playback.js).

Here is an example of loading the SDK into an HTML 5 script:

```html
<head>
    <script src="https://sdk.playback.streamamg.com/v1/playback.js"/>
</head>
```

Including type="text/javascript" is considered good practice in older versions of HTML (prior to HTML5), but it's not
required in modern HTML5 documents. Leaving it out can help keep your HTML cleaner and more concise.

Alternatively, to load the external JavaScript file from the above URL within a mobile app, you can use native networking
libraries or methods provided by the platform's SDK.

After installing the library you can then use the functionality provided by the SDK in your JavaScript code.
You should wait for the SDK to be fully loaded and initialized before using its features.

### Using the SDK

#### Initialising the SDK

First, you would initialise the SDK by providing your <b>Client API Key</b> and any playback capabilities (as options) you
would like to use as follows:

```javascript
// Initialize the Playback SDK with your provided Client API Key:
//  also pass over any playback options you desire e.g. 'autoplay'.
Playback.initialize('client-api-key', { autoplay: true });
```

#### Playing a Video

Next, to play a video in the player identify the video to play and start the player:
```javascript
 // Identity the video to play by the entry identifier:
 //     this is the MediaPlatform identifier.
 //  - also provide the DOM element container identifier:
 //     e.g. <div id="player"></div> so pass over 'player'.  
 //  - also provide the access token:
 //     this is required to access the protected resources.
 const playOptions = {
   container: 'player',
   entryId: '0_xxxxxxxx',
   token: 'access_token'
 };

 // Play the video (this example uses the 'static' calling approach):
 //     pass over any play options (as specified above).   
 Playback.play(playOptions)
   .catch((error) => {
     // Handle any unexpected error as you desire.
     console.log('error playing the video:', error);
   });
```

#### Further Integration 

Refer to the SDK reference documentation starting with the [Playback Class](https://sdk-docs.playback.streamamg.com/v1/docs/classes/playback.Playback.html) which is the main entry for the playback service offered. 

The underlying video player, e.g. [Bitmovin](https://cdn.bitmovin.com/player/web/8/docs/interfaces/Core.PlayerAPI.html), may also be obtained [here](https://sdk-docs.playback.streamamg.com/v1/docs/classes/Bitmovin.html#getRawPlayer) - this is 
offered for advanced integration and is typically not a recommended use-case.



