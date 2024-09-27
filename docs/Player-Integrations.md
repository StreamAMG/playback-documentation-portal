# Player Integrations

The supported player(s) may be integrated with the following features:

- <b>Global Adverts</b> (Injection of adverts at points in the playing video across all content and players) - By default, ads are enabled globally to all content, this can be manually configured in CloudMatrix to only show ads to content at the article/entry ID level. For example. ads can be served to LIVE content but not VOD, and visa versa.)

- <b>Resume Capability</b> (allow playback from saved/last watched positions in videos across devices)
- <b>MUX Analytics</b> (collection of video performance metrics)

All additional features, as above, are managed by the [onboarding process](./Client-Onboarding.md) so please contact the
client delivery team for further information and the onboarding of these features etc.


## Global Adverts
To integrate global advertisements with CloudMatrix, you'll need to include the `globalAdverts` configuration when setting up a new Playback Configuration or updating an existing one. You can find this information in the API documentation under:

`Playback Configuration API > Playback Configuration`

- Create a new playback configuration for a client
- Partially update an existing playback configuration

## Staging Playback SDK

Staging Playback SDK is available at 
```
https://sdk.playback.staging.streamamg.com/v1/playback.js
```
This is required when testing with Staging Playback Keys.
