# Migration Plan: MUX Analytics to Bitmovin Analytics (Core-5228)

## Overview

We cannot migrate from MUX Analytics to Bitmovin Analytics instantly. To ensure a smooth transition, we must
**maintain both analytics providers** for a period of time and allow switching based on configuration. This approach
enables us to validate Bitmovin’s functionality before completely deprecating MUX, also leaves us an option to rollback.

## Migration Strategy for Playback backend

### 1. Extend the `client-configs` Table
- Create a new config based on existing (for dev - a7c2d960-fccd-4a4e-b9ab-fb51ab2b0f6e), but with another `integrations` object, storing both Bitmovin configurations instead of MUX.
- Current configuration-api /player controller response with comments:
    ```json
    {
      // not related fields taken from config.player, mapped in /player controller
      ...
      "integrations": {
         "resume": {
             "enabled": true
         },
         "mux": { // to become bitmovin || bitmovinAnalytics
             "env_key": "hvksvvjt57jn0ijt55kubng1n",
              // pass in key from bitmovin Dashboard -> Licences
             "player_name": "bitmovin-12345678"
              // not sure whether we need that field at all
         }
      },
    }
    ```

The `client-keys` table continues referencing the correct configuration from `client-configs`. This ensures that analytics settings are dynamically applied at runtime.

- The `client-keys` table will map client keys to configurations, maintaining dynamic selection between MUX and Bitmovin.
- This allows seamless switching without changing SDK logic.

## Migration Strategy for Playback SDK

### 1. Analytics Requirements

For `Bitmovin Analytics` to start working (according the [docs](https://developer.bitmovin.com/playback/docs/setup-analytics-web)),
there are requirements:

- provide analytics key
- make sure to add Analytics module, if using Modular approach

### 2. Await for new property in `config.integrations`

In core `bitmovin.ts` class, we used to add integrations after applying all the logic/state/headers in playVideo()
function.

```typescript
playVideo() {
  ...
  // Load player source and integrate playback config
  this.player.load(this.playerSourceConfig).then(() => {
    console.log(`PLAYER analytics: `, this.player.analytics);
    return this.addIntegration(playbackConfig);
  });
}
```

But for Bitmovin Analytics integration we have to do that earlier, so the **player** instance already initialized with
analytics key, as we don't have a possibility to re-configure existing **player**. My suggestion is to
temporarily add analytics logic into `getConfig()` to conditionally pass analytics smth like that:

```typescript
private getPlayerConfig(config: PlayerConfig, options: PlayerOptions): PlayerConfig {
  const playbackConfig = { ...config.playback, autoplay: options.autoplay, muted: options.muted };
  
  return this.integration.bitmovin
    ? { ...config, playback: playbackConfig, analytics: { key: this.integration.bitmovin.key } }
    : { ...config, playback: playbackConfig };
}
```

### 3. Analytics Module
To add new row into `bitmovin.ts` setup

```typescript
...
Player.addModule(EngineBitmovinModule); // Provides common adaptive streaming functionality
Player.addModule(NativeEngineModule);
Player.addModule(PolyfillModule); // Provides polyfills for legacy browsers which don't support state-of-the-art JavaScript features like Promise or String.prototype.includes
Player.addModule(Crypto); // Provides Decyption of AES encrypted content
Player.addModule(MseRendererModule); // State-of-the-art video rendering of DASH, HLS or Smooth using the browser's MediaSource Extension
Player.addModule(ContainerTSModule); // Provides support for trans-multiplexing MPEG-2 TS to fMP4
Player.addModule(ContainerMp4Module); // Provides support for playback of MP4 container formats in supported browsers
Player.addModule(StyleModule); // Provides basic styling of the player
Player.addModule(HlsModule); // Provides support for HLS playback
Player.addModule(XmlModule); // Handling of XML files, like DASH or VAST manifests
Player.addModule(LowLatencyModule); // Provides low latency live-streaming support
Player.addModule(DashModule); // Provides MPEG-DASH support
Player.addModule(AbrModule); // Provides the available Adaptive BitRate algorithms
Player.addModule(DrmModule); // Provides support for Widevine, PlayReady, PrimeTime and Fairplay DRM systems
...
// Analytics events starts to fire after adding this and initializing a key
Player.addModule(Analytics); 
```
