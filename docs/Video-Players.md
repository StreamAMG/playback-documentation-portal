# Video Player Configuration

## Video Player Styling

### Bitmovin Player

The supported [Bitmovin Video Player](https://bitmovin.com/video-player) may be further styled with CSS. The UI is 
designed to be as flexible as possible. Please visit the [Bitmovin Player UI Styling](https://bitmovin.com/demos/player-ui-styling) 
documentation for more information regarding how you can adapt the UI to better fit the rest of your content.

>All elements can be restyled using CSS:  the [documentation](https://developer.bitmovin.com/playback/docs/player-ui-css-class-reference)
> provides a complete reference of all CSS classes used in the UI - this is the simplest way to customise the look and feel of Bitmovin Player.

To overwrite default styles with your own values, you can put them anywhere you would normally put CSS styles (i.e. external .css file or inline style blocks).

#### Examples

The CSS for overriding the font and disabling animations of the play button overlay would thus look like this (where #player is the id of your player element container):

```css
#player .bmpui-ui-uicontainer {
font-family: 'Roboto';
font-size: 10pt;
}

#player .bmpui-ui-hugeplaybacktogglebutton .bmpui-image:hover {
animation: none;
}

#player .bmpui-ui-hugeplaybacktogglebutton.bmpui-on .bmpui-image {
animation: none;
}

#player .bmpui-ui-hugeplaybacktogglebutton.bmpui-off .bmpui-image {
animation: none;
}
```

If the css is in an external file called say client-styles.css, you can include it in your player HTML file as:

```html
<link rel="stylesheet" type="text/css" href="client-styles.css">
``` 

#### Adding Watermarks

Player specific watermarks are disabled by default. Client specific watermarks may be added as follows:

1. Create a bespoke css file like client-styles.css if one doesn't already exist.
2. Add the following content:
```css
/* Override the default watermark which is turned off (use css styles to control display as required) */
#player .bmpui-ui-watermark {
  background-image: url("<your-watermark-image>") !important;
  display: block; /* (mandatory) to override default behaviour */ 
    
  /* FOLLOWING ARE EXAMPLES OF CONTROLLING THE DISPLAY USING CSS STYLES TO TAILOR THE WATERMARK TO YOUR REQUIREMENTS */
  background-size: cover; /* (optional) scale image to fit the watermark container */
  opacity: 0.2; /* (optional) set desired opacity etc for your watermark */
}
```
3. Include the css file in your player HTML file as:
```html
<link rel="stylesheet" type="text/css" href="client-styles.css">
```

### Rewind and Forward Buttons

Rewind and Forward Buttons can be styled by passing into the player options via the Javascript SDK the following CSS (optional styles):

- [forwardButtonClassName](https://sdk-docs.playback.streamamg.com/v1/docs/interfaces/PlayerOptions.html#forwardButtonClassName) - the class name of the forward button.
- [rewindButtonClassName](https://sdk-docs.playback.streamamg.com/v1/docs/interfaces/PlayerOptions.html#rewindButtonClassName) - the class name of the rewind button.

These options are then passed through during [initialisation](https://sdk-docs.playback.streamamg.com/v1/docs/classes/Playback.html#initialize) of the player e.g.
```javascript
Playback.initialize('client-api-key', { forwardButtonClassName: 'my-forward-button-class', rewindButtonClassName: 'my-rewind-button-class' });
```
