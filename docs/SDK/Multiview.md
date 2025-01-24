# StreamAMG Playback SDK Multiview Feature Documentation

The **StreamAMG Playback SDK** supports creating dynamic, interactive multiview players, enabling clients to add,
remove, and rearrange multiple video players within a single layout.
This guide explains the implementation of such functionality and provides space for customization based on specific requirements.
Examples provided for Angular consumer.
Main multiview reference from Bitmovin sample: https://streams.bitmovin.com/ct4qh1mf04t5ug4mfup0/share

## Overview

The multiview feature is built around the concept of **playerContainers**,
which serve as placeholders for the SDK video players. By managing these containers dynamically, you can:

- Play several videos in single layout.
- Add new players to specific containers.
- Rearrange the order of players using drag-and-drop.

## Key Concepts
### Player Containers
The concept of player containers is essential for dynamically adding video players to your layout. <br>
**Each player resides in a unique container identified by a `containerId` string (e.g., `playback-container-0`).** <br>
You can choose any iterable data structure to manage these containers, such as an array, a map, or even a custom
implementation that suits your needs. In this example, we use an array to manage the layout of the player containers and
a separate array to store references to the video players themselves.
```html
<div class="multiview">
    <ng-container *ngFor="let containerId of playerContainers; let i = index">
        <div class="player-container">
           <div [id]="containerId"></div> <!-- template to be filled with video by id from SDK -->
        </div>
    </ng-container>
</div>
```

```typescript
public playerContainers: string[] = []; // container IDs
private players: VideoPlayer[] = []; // video players references
```

### Selection/Feed
The selection block serves as a view and it also acts as an event controller.
While it is not directly tied to the multiview layout, interactions with this block are likely to trigger changes to the
players, such as creating or removing containers or loading video players into them.

```html
<div class="selection">
    <ng-container *ngFor="let item of selection">
        <div class="item" (click)="handleSelectionClick(item.entry)">
            <!-- any required video data -->
            <img [src]="item.source">
        </div>
    </ng-container>
</div>
```

```typescript
public selection: VideoDetails[] = [];
```

### Adding a Player
Use the `addPlayer(entryId)` method to add a new video player to the layout. If the maximum limit of players is reached,
the method prevents additional players from being added.

```typescript
addPlayer(entryId: string) {
  if (this.players.length >= PLAYERS_MAX_LIMIT || this.playerContainers.length >= PLAYERS_MAX_LIMIT) return;

  const containerIndex = this.availableIndexes.length > 0
    ? this.availableIndexes.shift()!
    : this.playerContainers.length;
	
  const containerId = `playback-container-${containerIndex}`;
  this.playerContainers.push(containerId);

  this.cdr.detectChanges(); // Ensures the DOM is updated before SDK async logic

  this.createPlaybackPlayer(entryId, containerId, this.playerContainers.length - 1);
}
```

```typescript
private createPlaybackPlayer(entryId: string, containerId: string) {
  try {
    // @ts-ignore
    Playback.initialize(this.playbackService.apiKey, { ...this.playbackService.sdkGlobalConfig });
  } catch (error: any) {
    alert(`Error: ${error.message}`);
  }

  const playOptions = {
    container: containerId,
    entryId: entryId,
    token: this.playbackService.authToken,
  };

  // @ts-ignore
  Playback.player(containerId)
    .then((player: VideoPlayer) => {
      this.players.push(player);
      return player.play(playOptions.entryId, playOptions.token);
    })
    .then(() => {
      this.logIndexes(); // Optional: logs the current state of players, containers, and indexes
    })
    .catch((error: any) => {
      alert(error.message);
    });
}
```

### Removing a Player
To remove a player, use the `removePlayer(index)` method. This method ensures that the video player is destroyed, the
container is removed, and its index is made available for reuse. This prevents duplicate containers and maintains proper
order.

```typescript
removePlayer(index: number) {
  const player = this.players[index];
  if (player) {
    player.destroy();
  }

  const removedContainerId = this.playerContainers.splice(index, 1)[0];
  this.players.splice(index, 1);

  const containerIndex = parseInt(removedContainerId.split('-').pop() || '0', 10);
  if (!this.availableIndexes.includes(containerIndex)) {
    this.availableIndexes.push(containerIndex);
    this.availableIndexes.sort((a, b) => a - b);
  }

  this.logIndexes(); // Optional: logs the current state of players, containers, and indexes
}
```

### Available Indexes
To avoid duplicate or conflicting container IDs, the implementation tracks removed containers and reuses their indexes
for new players. It depends on data structure you'll use for handling iterable to render for containers/keep players
references, so might not be needed.
```typescript
private availableIndexes: number[] = []; // handling deleted indexes so no duplicate containers
```

### Layout
Styling containers in a way we have bigger one, several small around it, in case there is only one video - to scale it
full size, suggested to use css grid-templates.
```html
<div class="multiview" [ngClass]="{ 'single-player': playerContainers.length === 1 }">
    <ng-container *ngFor="let containerId of playerContainers; let i = index">
        <div class="player-container">
            <div [id]="containerId"></div>
        </div>
    </ng-container>
</div>
```
```css
/*pseudo code example*/

.multiview {
  display: grid;
  grid-template-columns: 1fr 35%;
  grid-auto-rows: minmax(150px, auto);
  flex: 1;
}
.player-container {
  display: flex;
  justify-content: center;
  align-items: center;
  box-sizing: border-box;
}
.player-container:first-child {
  grid-column: 1 / 2;
  grid-row: span 3;
}
.multiview.single-player {
  grid-template-columns: 1fr;
  grid-template-rows: 1fr;
}
.multiview.single-player .player-container:first-child {
  grid-column: 1 / -1;
  grid-row: 1;
  height: 100%;
  width: 100%;
}
```

### Drag-and-Drop Functionality
Players can be rearranged in the layout by dragging and dropping their containers. This enhances the user experience by
providing a flexible and interactive interface. Basic solution is HTML Drag and Drop API.

```html
<div
  class="player-container"
  draggable="true"
  (dragstart)="onDragStart($event, i)"
  (dragover)="onDragOver($event)"
  (drop)="onDrop($event, i)"
>
    <div [id]="containerId"></div>
</div>
```

```typescript
private draggedIndex: number | null = null;

onDragStart(event: DragEvent, index: number): void {
  this.draggedIndex = index;
  event.dataTransfer?.setData('text/plain', index.toString());
  event.dataTransfer!.effectAllowed = 'move';
}

onDragOver(event: DragEvent): void {
  event.preventDefault();
  event.dataTransfer!.dropEffect = 'move';
}

onDrop(event: DragEvent, targetIndex: number): void {
  event.preventDefault();

  if (this.draggedIndex === null || this.draggedIndex === targetIndex) return;

  // rearrange the dragged and target items in playerContainers
  const draggedContainer = this.playerContainers[this.draggedIndex];
  this.playerContainers.splice(this.draggedIndex, 1);
  this.playerContainers.splice(targetIndex, 0, draggedContainer);

  const draggedPlayer = this.players[this.draggedIndex];
  this.players.splice(this.draggedIndex, 1);
  this.players.splice(targetIndex, 0, draggedPlayer);

  this.draggedIndex = null;

  this.logIndexes(); // Optional: logs the current state of players, containers, and indexes
}
```