# StreamAMG Playback SDK for JavaScript Documentation

The StreamAMG Playback SDK for JavaScript enables developers to build user-interface based applications integrating with
a video player of choice and configuration.

You can use the SDK inside your Web and Mobile Javascript applications to obtain video player capability resulting in
an embedded video player inside a container with video player capabilities. You may choose and configure the video
player of choice with capabilities e.g. autoplay.

The goal of the SDK is to offer versatility and a range of tools for crafting a personalized multimedia encounter,
allowing you various methods to seamlessly incorporate a video player into your web application, enabling you to make
well-informed decisions regarding the most optimal integration approach.

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
called 'playback.js' is available for downloading into your application at the following URL: 

>https://sdk.playback.streamamg.com/v1/playback.js
>
>> Currently, there is no public NPM package published for this SDK.

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

All the optional [PlayerOptions](https://sdk-docs.playback.streamamg.com/v1/docs/interfaces/PlayerOptions.html) available are as follows :
 - [autoplay](https://sdk-docs.playback.streamamg.com/v1/docs/interfaces/PlayerOptions.html#autoplay) - a boolean value indicating if the video should play automatically.
 - [muted](https://sdk-docs.playback.streamamg.com/v1/docs/interfaces/PlayerOptions.html#muted) - a boolean value indicating if the audio output of the video should be muted.
 - [seekInterval](https://sdk-docs.playback.streamamg.com/v1/docs/interfaces/PlayerOptions.html#seekInterval) - the time interval in milliseconds at which the video should be seeked.
 - [forwardButtonClassName](https://sdk-docs.playback.streamamg.com/v1/docs/interfaces/PlayerOptions.html#forwardButtonClassName) - the class name of the forward button.
 - [rewindButtonClassName](https://sdk-docs.playback.streamamg.com/v1/docs/interfaces/PlayerOptions.html#rewindButtonClassName) - the class name of the rewind button.
 - [retrieveSessionToken](https://sdk-docs.playback.streamamg.com/v1/docs/interfaces/PlayerOptions.html#retrieveSessionToken) - function to fetch/refresh the session token.

##### Access Tokens #####

To use the SDK with protected video resources, you will need to provide an access token from your authentication provider e.g. AWS Cognito, Auth0, etc. This is passed through 
when playing the video through the static [Playback.play](https://sdk-docs.playback.streamamg.com/v1/docs/classes/Playback.html#play) or instance [player.play](https://sdk-docs.playback.streamamg.com/v1/docs/classes/Playback.html#play-2) methods. 
This access token can be automatically refreshed on expiry by providing a [retrieveSessionToken](https://sdk-docs.playback.streamamg.com/v1/docs/classes/Playback.html#retrieveSessionToken) function (the implementation of
the actual function is dependent on your authentication provider) to refresh the access token. The token is only refreshed once per session using the supplied bespoke function which is optional.

A generic example of defining and passing over a [retrieveSessionToken](https://sdk-docs.playback.streamamg.com/v1/docs/classes/Playback.html#retrieveSessionToken) function is shown below:
```javascript
// Define a function that returns a session token
function getSessionToken() {
    // Logic to retrieve or generate the session token
    return 'example_session_token';
}

// Pass over the retrieveSessionToken function into the initialise playback call
Playback.initialize('client-api-key', { retrieveSessionToken: getSessionToken });
```

A further example of a function to refresh an AWS Cognito session token is shown below (this example uses the AWS 
Cognito OAuth2 token endpoint and is only intended for illustrative purposes):

```javascript
function getSessionToken(refreshToken, clientId, clientSecret = null) {
    const params = new URLSearchParams({
        grant_type: 'refresh_token',
        client_id: clientId,
        refresh_token: refreshToken,
        client_secret: clientSecret
    });

    // Call the Cognito OAuth 2.0 Token endpoint for your user pool by submitting your refresh token.
    return fetch('https://your-user-pool-domain.auth.eu-west-1.amazoncognito.com/oauth2/token', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/x-www-form-urlencoded',
        },
        body: params
    })
    .then(response => {
        // Check for errors and if all ok return the new access token.
    })
}

// Pass over the retrieveSessionToken function into the initialise playback call with the refresh token etc required.
Playback.initialize('client-api-key', { retrieveSessionToken: () => getSessionToken('your_refresh_token', 'your_client_id', 'your_client_secret') });
```

#### Playing a Video

Next, to play a video in the player identify the video to play and start the player:
```javascript
 // Identify the video to play by the entry identifier.
 //  - also provide the DOM element container identifier:
 //     e.g. <div id="player"></div> so pass over 'player'.  
 //  - also provide an access token from your authentication provider:
 //     this is required to access protected video resources.
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




