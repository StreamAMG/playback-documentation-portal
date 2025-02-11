# StreamAMG Playback SDK Playlist Documentation

The **Playback SDK** allows users to work with playlists seamlessly, enabling them to handle multiple video entries in a structured manner. With playlist support, you can:

- Automatically play videos in sequence.
- Navigate between videos using `next`, `previous`, and `goTo` functionality.
- Use both **static and instance-based playlist handling**.

## Example Usage

### Initialize and Play a Playlist

First step is to initialize `Playback` instance with your API key.

```javascript
try {
  Playback.initialize(apiKey);
} catch (error) {
  console.error(error);
}
```

You can use the `Playback.runPlaylist()` method to initialize player and start playlist.

#### **Static Playlist Usage**
```javascript
const playlistOptions = {
  container: `player-container`,
  entryId: ['entry1', 'entry2', 'entry3'],
  token: this.playbackService.authToken,
  options: { muted: false, autoplay: true },
};

Playback.runPlaylist(playlistOptions)
```

#### **Instance-Based Playlist Usage**
```javascript
const playlistOptions = {
  container: `player-container-1`,
  entryId: ['entry1', 'entry2', 'entry3'],
  token: this.playbackService.authToken,
};

Playback.player('player-container-1', { muted: false, autoplay: true })
  .then((player: VideoPlayer) => {
    this.player = player;

    return player.runPlaylist(playlistOptions);
  })
  .then((playlist: PlaylistController) => {
    this.playlist = playlist;
  })
```

### Navigating between playlist entries

Each `Playback` instance now provides a `playlist` controller that allows you to interact with the playlist:

- `next()` switches the player to the next entry.
- `prev()` switches the player to the previous entry.
- `goTo(entryId)` switches the player to a specific entry by `entryId`.
- `getCurrentIndex()` returns currently playing playlist's entries index.
- `getCurrentEntry()` returns currently playing playlist's entry id.

Each method throws an error if an invalid operation is performed.

> Error messages include:
> - `End of playlist reached`
> - `Start of playlist reached`
> - `Entry ID ${entryId} not found in playlist`

#### Next
```javascript
// Static
try {
  await Playback.playlist.next();
} catch (error) {
  alert(error.message);
}

// Instance
try {
  await player.playlist.next();
} catch (error) {
  console.error(error.message);
}
```

#### Previous
```javascript
// Static
try {
  await Playback.playlist.prev();
} catch (error) {
  alert(error.message);
}

// Instance
try {
  await player.playlist.prev();
} catch (error) {
  console.error(error.message);
}
```

#### GoTo
```javascript
// Static
try {
  await Playback.playlist.goTo(entryId);
} catch (error) {
  alert(error.message);
}

// Instance
try {
  await player.playlist.goTo(entryId);
} catch (error) {
  console.error(error.message);
}
```

---

> **NOTE:** You can now use **instance-based playlist handling**.
> This means each player instance maintains its own playlist state.
>
> Example:
> ```javascript
> const playerX = await Playback.player('playerX');
> const playlistX = await playerX.runPlaylist(optionsX);
>
> const playerY = await Playback.player('playerY');
> const playlistY = await playerY.runPlaylist(optionsY);
>
> await playlistX.next();
> await playlistY.prev();
> // Alternative 
> await playerX.playlist.next();
> await playerY.playlist.prev();
> ```
>
> This allows multiple active playlists across different players.
