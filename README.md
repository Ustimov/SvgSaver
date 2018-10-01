# SvgSaver

Save SVGs as PNGs from the browser or as URIs in Node.js

## Installation

```
npm install svg-saver
```

## Prerequisites

SvgSaver relies on JavaScript promises, so any browsers that don't natively support the standard `Promise` object will need to have a polyfill.

## Usage

### Browser

To save a PNG, include the script `svg-saver.js` in your page and create a new `SvgSaver` instance:

```javascript
var svgSaver = new SvgSaver(window);
```

Then call the `saveSvgAsPng` object's method with an SVG node and a filename:

```javascript
svgSaver.saveSvgAsPng(document.getElementById("diagram"), "diagram.png");
```

The filename is the preferred filename when saving the image to the file system. The browser may change the name of the file if there is already a file by that name in the target directory.

If you want to scale the image up or down, you can pass a scale factor in an options object:

```javascript
svgSaver.saveSvgAsPng(document.getElementById("diagram"), "diagram.png", {scale: 0.5});
```

Other options are documented below.

If you just want a dataURI for an SVG, you can call `svgAsDataUri` with an SVG node, options, and a callback:

```javascript
svgSaver.svgAsDataUri(document.getElementById("diagram"), {}, function(uri) {
  ...
});
```

If you want a dataURI of a PNG generated from an SVG, you can call `svgAsPngUri` with an SVG node, options, and a callback:

```javascript
svgSaver.svgAsPngUri(document.getElementById("diagram"), {}, function(uri) {
  ...
});
```

### Options

- `backgroundColor` — Creates a PNG with the given background color. Defaults to transparent.
- `canvg` - If canvg is passed in, it will be used to write svg to canvas. This will allow support for Internet Explorer
- `encoderOptions` - A Number between 0 and 1 indicating image quality. The default is 0.8
- `encoderType` - A DOMString indicating the image format. The default type is image/png.
- `fonts` - A list of `{text, url, format}` objects the specify what fonts to inline in the SVG. Omitting this option defaults to auto-detecting font rules.
- `height` - Specify the image's height. Defaults to the viewbox's height if given, or the element's non-percentage height, or the element's bounding box's height, or the element's CSS height, or the computed style's height, or 0.
- `left` - Specify the viewbox's left position. Defaults to 0.
- `modifyCss` - A function that takes a CSS rule's selector and properties and returns a string of CSS. Supercedes `selectorRemap` and `modifyStyle`. Useful for modifying properties only for certain CSS selectors.
- `modifyStyle` - A function that takes a CSS rule's properties and returns a string of CSS. Useful for modifying properties before they're inlined into the SVG.
- `scale` — Changes the resolution of the output PNG. Defaults to `1`, the same dimensions as the source SVG.
- `selectorRemap` — A function that takes a CSS selector and produces its replacement in the CSS that's inlined into the SVG. Useful if your SVG style selectors are scoped by ancestor elements in your HTML document.
- `top` - Specify the viewbox's top position. Defaults to 0.
- `width` - Specify the image's width. Defaults to the viewbox's width if given, or the element's non-percentage width, or the element's bounding box's width, or the element's CSS width, or the computed style's width, or 0.

### Support

[Chrome limits data URIs to 2MB](http://stackoverflow.com/questions/695151/data-protocol-url-size-limitations/41755526#41755526), so you may have trouble generating PNGs beyod a certain size.

Internet Explorer will only work if [canvg](https://github.com/canvg/canvg) is passed in, otherwise it will throw a `SecurityError` when calling `toDataURL` on a canvas that's been written to. [canvg](https://github.com/canvg/canvg) may have it's own issues with SVG support, so ensure you test the output after switching.

## Node.js

In order to use the library with Node.js you need to install [jsdom](https://github.com/jsdom/jsdom) and [node canvas v2](https://github.com/Automattic/node-canvas).

Then create an `JSDOM` instance, decorate its `createElement` method to create a node canvas instance and replace the `Image` constructor:


```javascript
const { createCanvas, Image } = require('canvas');
const jsdom = require("jsdom");
const { JSDOM } = jsdom;

const { window } = new JSDOM(`
    <html>
        <!-- So you can use local resources -->
        <link rel="stylesheet" href="file://bootstrap.min.css">
        <body></body>
    </html>`, {
    resources: "usable"
});

var backup = window.document.createElement;
window.document.createElement = function (el) {
    if (el === 'canvas') {
        return createCanvas(500, 500);
    } else {
        return backup.bind(window.document)(el);
    }
}
window.Image = Image;
```

Now create a new `SvgSaver` instance and pass the `window` instance:

```javascript
const SvgSaver = require("svg-saver");
const svgSaver = new SvgSaver(window);

svgSaver.svgAsDataUri(window.document.getElementById("diagram"), {}, function(uri) {
  ...
});
```

Then you can use the library like in [browser](#Browser), but only "as uri" methods.
