# StreamAMG Playback SDK Playlist Documentation

The **Playback SDK** allows users to work with playlists seamlessly, enabling them to handle multiple video entries in a structured manner. With playlist support, you can:

- Automatically play videos in sequence.
- Navigate between videos using `next`, `previous` and `goTo` functionality.

## Example Usage

### Initialize and Play a Playlist

First step is to initialize `Playback` instance with your api key.

```javascript
try {
  Playback.initialize(apiKey);
} catch (error) {
  console.error(error);
}
```

You can use the `Playback.runPlaylist()` method to initialize player and start playlist.

```javascript
Playback.runPlaylist({
  containerId: 'player',
  entryId: ['entry1', 'entry2', 'entry3'],
  authorizationToken: 'access_token',
});
```

### Navigating between playlist entries

The `Playback` provides you with `playlistController` property with following methods:

- `next()` switching player to next entry
- `prev()` switching player to previous entry
- `goTo(entryId)` switching player to direct entry by entryId

for each method there is an error thrown to await for.

> Error messages are not finalized yet so they are just strings -
> `End of playlist reached` \ `Start of playlist reached` \ `Entry ID ${entryId} not found in playlist`

#### Next

```javascript
try {
  await Playback.playlistController.next();
} catch (error) {
  console.error(error.message);
}
```

#### Previous

```javascript
try {
  await Playback.playlistController.prev();
} catch (error) {
  console.error(error.message);
}
```

#### GoTo

```javascript
try {
  await Playback.playlistController.goTo(entryId);
} catch (error) {
  console.error(error.message);
}
```

---

> **WARNING:** You should be careful working with `playlistController` using **Playback SDK** instance methods,
> as this approach is not supported. `playlistController` is static property in `Playback` and it's re-written each time
> `runPlaylist()` called, so "player" not storing state of playlists, it's ***only one active playlist available*** in `Playback`.
> ```javascript
> this.playerX = await Playback.player('playerX');
> this.playerY = await Playback.player('playerY');
> await this.playerX.runPlaylist(optionsX);
> await this.playerY.runPlaylist(optionsY);
>```
> 