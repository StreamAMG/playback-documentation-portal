## Playback Sdk

Playback SDK allowing to initialize player in a container and play a video.

There are two ways the Playback SDK can be included, either via HTML or an NPM package.

**Via HTML**

```
<script src="https://sdk.playback.streamamg.com/v1/playback.js"></script>
```

**Via NPM**

```
npm i <PACKAGE_NAME>
```

#### Static approach

To use the Playback SDK with the static approach, follow these steps:

1. Initialize the Playback SDK by calling the `initialize` method with your API key:
   ```
    Playback.initialize( <YOUR_API_KEY> , options );
   ```
2. Create an object `playOptions` that contains the necessary details for playing a video. This includes the container ID, entry ID, and token:
   ```
    const playOptions = {
   	container: 'player', // The ID of the container where the player will be initialized
   	entryId: '0xabc123', // The ID of the video to be played
   	token: '<YOUR_AUTH_TOKEN>', // The authorization token for the client
   	options
   };
   ```
3. Call the `play` method of the Playback class and pass the `playOptions` object as a parameter:
   ```
   Playback.play(playOptions)
   	.catch((error) => {
   		console.log('error:', error);
   });
   ```

By following these steps, you will be able to initialize the Playback SDK and play a video using the static approach.

#### Dynamic approach

To use the Playback SDK with the dynamic approach, follow these steps:

1. Initialize the Playback SDK by calling the `initialize` method with your API key:
   ```
    Playback.initialize( <YOUR_API_KEY>, options );
   ```
2. Create a player instance by calling the `player` method and passing the container ID:
   ```
   Playback.player("player", options)
   		.then((player) => {
   			// Use the player instance for further operations
   		})
   		.catch((error) => {
   			console.log('error:', error);
   		});
   ```
3. Once you have the player instance, you can call the `play` method to play a specific video:
   ```
    player.play('0xabc123', '<YOUR_AUTH_TOKEN>', options)
   	.catch((error) => {
   		console.log('error:', error);
   	});
   ```
   When using the Playback SDK, you can optionally add the `options` object that allows you to customize the behavior of the player instance. This object can be set at different levels: SDK level, player level, and specific entry level.
   Example of object with all available props:
   ```
    const options = {
   	autoplay: true,	// (boolean, optional), autoplay content on load, default false
   	muted: true,	// (boolean, optional), mute player, default false
   	retrieveSessionToken: () => {} // (function, optional), a callback function that retrieves a token to refresh it when it expired
    }
   ```
