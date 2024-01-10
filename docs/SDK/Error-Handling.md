# Error Handling

Errors are displayed on the top of the video player as an overlay in the player as events raising errors are triggered.

The following error overlays may be displayed by default:

- Failed to play the video
- No playback information retrieved
- No session token
- No playback configuration

Some errors may happen during initialization when we fetch internally the video player config for the client whilst
other errors may happen during playing of the video (e.g. entitlement issues).

> Error details are logged to the console. In a JavaScript HTML page, you can inspect console logged error messages 
> using the browser's developer tools for debugging or troubleshooting purposes. 

## Custom Overlay

The overlay is feature flagged and enabled by default. This allows third-party developers the ability to switch it off
and implement their own bespoke customer error notification screen using our thrown errors. The feature is controlled 
through the [Playback Configuration API](../../reference/Playback-Configuration-API.yaml) `customErrorScreen` feature 
flag.

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

