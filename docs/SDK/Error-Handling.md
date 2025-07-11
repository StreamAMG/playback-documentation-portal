# Error Handling

## Error Overlay

Errors are displayed on the top of the video player as an overlay in the player as events raising errors are triggered.

The following error overlays are displayed by default (this may be overridden):

- Failed to play the video
- No playback information retrieved
- No session token
- No playback configuration

### Custom Overlay

The overlay is feature flagged and enabled by default. This allows third-party developers the ability to switch it off
and implement their own bespoke customer error notification screen using our thrown errors (see below). The feature is 
controlled through the [Playback Configuration API](../../reference/Playback-Configuration-API.yaml) `customErrorScreen` feature flag.

## Error Categories

There are [categories of errors](https://sdk-docs.playback.streamamg.com/v1/docs/enums/ErrorType.html) to help with the
identification of the error. All the error definitions, belonging to a particular category,  are defined as follows:

- [Playback Errors](https://sdk-docs.playback.streamamg.com/v1/docs/enums/PlaybackErrorName.html)
- [SDK Errors](https://sdk-docs.playback.streamamg.com/v1/docs/enums/SDKErrorName.html)
- [Resume Error](https://sdk-docs.playback.streamamg.com/v1/docs/enums/ResumeErrorName.html)

> Error details are logged to the console. In a JavaScript HTML page, you can inspect console logged error messages
> using the browser's developer tools for debugging or troubleshooting purposes.

### SDK Errors

[SDK Errors](https://sdk-docs.playback.streamamg.com/v1/docs/enums/SDKErrorName.html)
this section provides an overview of potential error codes that may be encountered while using the SDK.
Each error represents a specific issue that should be handled appropriately by the consuming application.

`CONFIGURATION_NOT_RETRIEVED`  
This error occurs when the SDK is unable to retrieve the required configuration. It may indicate that an API call to
fetch configuration settings failed or that necessary environment variables are missing.

`SDK_NOT_INITIALIZED`  
This error indicates that the SDK has not been properly initialized before attempting an operation. Ensure that the SDK
initialization method has been called successfully before proceeding with other API calls - `Playback.initialize(apiKey)`

`PLAYER_NOT_INITIALIZED`  
This error occurs when an operation is attempted on a player instance that has not been initialized. Ensure that a
player instance has been created and properly set up before calling player-related methods.

`REQUIRED_PARAMETERS_NOT_PASSED`  
This error is thrown when one or more required parameters are missing from an SDK function call, for most cases it's
missing containerId or videoId. Check the method documentation to verify that all mandatory parameters are included.

`SESSION_TOKEN_NOT_RETRIEVED`  
This error indicates that the session token required for authentication or authorization could not be retrieved. It
may be due to a failed API request or incorrect authentication setup. Thrown only once trying to get playback information
for direct entry.

`NO_PLAYER_EXISTS`  
This error is raised when an operation is attempted on a player that does not exist. Ensure that a valid player instance
has been created before executing player-related functions.

`NO_VIDEO_DETAILS`  
This error means that video details are unavailable. This might happen if the provided containerId
is incorrect or if there was an issue retrieving playback config.

`NO_PLAYBACK_DETAILS`  
This error occurs when playback configuration fetch failed or missing. Thrown once attempting to load config before
playing new entry.

`UNKNOWN_PARAMETER`  
This error is triggered when unknown event is passed to `player.off(event)`.

### Playback Errors

[Playback Errors](https://sdk-docs.playback.streamamg.com/v1/docs/enums/PlaybackErrorName.html)
this section provides an overview of potential Playback-side related error codes.

`NO_ENTITLEMENT`  
This error occurs when playback configuration request returns error with `errorData.reason === PlaybackErrorName.NoEntitlement`

### Resume Errors

[Resume API](https://sdk-docs.playback.streamamg.com/v1/docs/enums/ResumeErrorName.html) responsible for server-to-client
communication handling watched timestamps/multiple devices support requires WebSocket connection.
`RESUME_ERROR` or `WebSocket Connect Error` may happen due to several reasons such as environment, CORS, firewall,
certificates issues, trying to stable that connection.

## Troubleshooting

Reasons for an error may be as follows:

- Authentication token is invalid
- No entitlements for the video to play
- The customer account is blocked as there have been too many failed login attempts
- Customer has failed the concurrency check as there are too many devices in use
- No permissions to view the video entry
- Customer has insufficient entitlements
- Customer is not authenticated, user must be authenticated to play the video
- Subscription is deactivated, renewal requires customer authentication
- Customer has no subscriptions

The user related errors are those defined here [Playback User Errors](https://sdk-docs.playback.streamamg.com/v1/docs/enums/PlaybackErrorName.html).

## Handling Errors
- Ensure that all required configurations and parameters are correctly set before invoking SDK functions.
- Implement error handling and logging to capture additional context when an error occurs.
- Refer to the SDK documentation for additional troubleshooting steps related to specific errors.

