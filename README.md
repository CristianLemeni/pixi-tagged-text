# pixi-rich-text

[![NPM](https://nodei.co/npm/pixi-rich-text.png)](https://nodei.co/npm/pixi-rich-text/)

Add a `RichText` object inside [pixi.js](https://github.com/GoodBoyDigital/pixi.js) to easily create text using different styles.

Inspired by the original [pixi-multistyle-text](https://github.com/tleunen/pixi-multistyle-text)

## Architecture

RichText allows you to control text with multiple styles and embedded images as though it were a single Pixi component. It was inspired by [pixi-multistyle-text](https://github.com/tleunen/pixi-multistyle-text) but is structurally rather different. Pixi-rich-text creates a separate PIXI.Text object for each separately positioned element (word) in its text. This allows developers to have control over individual words or even characters for the purposes of animation or other effects. It also makes embedding sprites into the layout easier. In contrast, pixi-multistyle-text creates a single bitmap drawing based on the contents of text. Cosmetically, they're very similar, however, the overhead of creating multiple Text objects is much larger making RichText a potentially very heavy component. **If you require maximum performance, consider using [pixi-multistyle-text](https://github.com/tleunen/pixi-multistyle-text)**.

### Life-cycle

In order to maximize performance of RichText, it helps to understand how it renders the input.

1. Constructor - creating the component is the first step. You can set text, styles, and options in the constructor.
2. `update()` - Update generates a list of plain JS object tokens that hold information on what type of text to create and where to position it. The tokens contain all the information you need to draw the text and are saved as the instance member `tokens` but also returned by the `update()` method. By default, this is called every time the text or style definitions are changed (e.g. `setTagStyles()`, `setText()`). This is a fairly expensive process but usually faster than `draw()`.
3. `draw()` - Draw creates the child objects based on the data generated by `update()`. It clears any existing children (if needed) then recreates and positions them. This is probably the costliest method in the life-cycle. By default, this is called automatically by `update()`.
4. Of course, you won't see anything on your screen until your component is added to a visible PIXI container.

The methods that normally trigger an update are:

- `setText()` & `myText.text =` (implicit setter)
- `setTagStyles()` & `tagStyles =` (implicit setter)
- `setStyleForTag()`
- `setDefaultStyle()` & `defaultStyle =` (implicit setter)
- `removeStylesForTag()`

The methods that normally trigger a draw:

- `update()`

**Please note that direct changes to styles or other objects will not trigger an automatic update.** For example:

```typescript
const t = new RichtText("<big>Big text</big>", { big: { fontSize: 25 } }); // renders "Big text" at 25px

t.getStyleForTag("big").fontSize = 100; // The change to the style wasn't detected. It still renders "Big text" at 25px

t.update(); // now it renders correctly.

t.textFields[0].visible = false; // Makes the word "Big" disappear.

t.draw(); // recreates the text fields restoring "Big"
```

#### `skipUpdates` & `skipDraw`

If performance is becoming an issue, you can use the `skipUpdates` and `skipDraw` flags in the options object with `new RichText()` to disable automatic updates and automatic drawing (more on that below). This gives you control over when the update() or draw() function will be called. However, the component can become out of sync with what you see on your screen so use this with caution.

Several other individual functions, such as `setText()` also give you the option to `skipUpdate` on an as needed basis.

```typescript
// Create a new RichText but disable automatic updates and draws.
const t = new RichText("", {}, {skipUpdate: true, skipDraw: true});
const words = ["lorem", "ipsum", ... ];
// add words until the length of text is > 500 characters.
while (t.untaggedText.length <= 500) {
  t.text += words[i];
}

// Normally, update() will draw() also, but we've disabled that.
// t.tokens will be updated to match the new text but it will not appear on the screen.
t.update();
t.textContainer.length; // 0 - text fields never got created.

// Manually call draw to generate the PIXI.Text fields
t.draw();
t.textContainer.length; // This will now contain all the PIXI.Text objects created by draw.
```

## Options

The third parameter in the RichText constructor is a set of options.

- `debug` - If `true`, generates debug information which is overlaid on the text and logged to the console during `draw()`. default is `false`.
<!-- - `splitStyle` - Allows you to specify how the text should be split into PIXI.Text objects. Optional values are "words" (default) and "characters". -->
- `imgMap` - An object that maps string names like `"myImage"` to Sprite objects like `PIXI.Sprite.from("./myImage.png")`. When a style contains `imgSrc="myImage"`, the matching sprite is used. By default, each of the keys you provide here will automatically be added as a style in the `tagStyles` (equivalent to `{ myImage: { imgSrc: "myImage"}}`) so you can add a tag `<myImage />`. default is `{}`.
- `skipUpdates` - When `true`, `update()` will not be called when text or styles are changed; it must be called explicitly or overridden using the skipUpdate parameter in functions such as `setText()`. default is `false`.
- `skipDraw` - When `true`, `draw()` will not be called by `update()` it must be called explicitly or overridden using the `skipDraw` parameter, e.g. `myRichText.update(false)`. default is `false`.

## Child DisplayObjects

RichText generates multiple display objects when it renders the text with `draw()`. Developers can access these children if desired for example to add additional interactivity or to animated individual elements.

**Please note that by default, these are recreated every time text or style properties change (technically, whenever `draw()` is called).** Manipulating the children directly may cause your view to become out of sync with the text and styles.

These properties are available:

- `textContainer` - The Sprite layer which holds all the text fields rendered by draw.
- `spriteContainer` - The Sprite layer which holds all the sprites rendered by draw if you're using an image map (`imgMap`).
- `debugContainer` - The Sprite layer which holds all debug overlay information (if you're using the `debug: true` setting).
- `textFields` - An array containing all the text fields generated by draw.
- `sprites` - If you're using an image map (`imgMap`), this array stores references to all the Sprites generated by draw.
- `spriteTemplates` - The sprites in `sprites` and `spriteContainer` are actually _clones_ of the originals passed in via the `imgMap` option. To get the originals, access them this way.

## Example

TBD

## Build instructions

```bash
yarn install
yarn build
```

## Usage

TBD

## Demo

You can view some examples using the command:

```bash
yarn demo
```

This will start a simple HTTP server running locally on port 8080.

## License

MIT, see [LICENSE.md](http://github.com/tleunen/pixi-multistyle-text/blob/master/LICENSE.md) for details.
