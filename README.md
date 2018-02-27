# timesnap

**timesnap** is a Node.js program that records screenshots of web pages that use JavaScript animations. It uses [puppeteer](https://github.com/GoogleChrome/puppeteer) to open a web page, overwrite its time-handling functions, and record snapshots at virtual times. For some web pages, this allows frames to be recorded slower than real time, while appearing smooth and consistent when recreated into a video.

You can use **timesnap** from the command line or as a Node.js library. It requires Node v6.4.0 or higher and npm to be installed.

To record screenshots and compile them into a video using only one command, see **[timecut](https://github.com/tungs/timecut)**.

## <a name="limitations" href="#limitations">#</a> **timesnap** Limitations
**timesnap** only overwrites JavaScript functions, so pages where changes occur via other means (e.g. through video or transitions/animations from css rules) will likely not render as intended.

## Read Me Contents

* [From the Command Line](#cli-use)
  * [Global Install and Use](#cli-global-install)
  * [Local Install and Use](#cli-local-install)
  * [Command Line *url*](#cli-url-use)
  * [Command Line Examples](#cli-examples)
  * [Command Line *options*](#cli-options)
* [From Node.js](#node-use)
  * [Node Install](#node-install)
  * [Node Examples](#node-examples)
  * [Node API](#node-api)

## <a name="from-cli" href="#from-cli">#</a> From the Command Line

### <a name="cli-global-install" href="#cli-global-install">#</a> Global Install and Use
To install:

Due to [an issue in puppeteer](https://github.com/GoogleChrome/puppeteer/issues/375) with permissions, timesnap is not supported for global installation for root. You can configure `npm` to install global packages for a specific user following this guide: https://docs.npmjs.com/getting-started/fixing-npm-permissions#option-two-change-npms-default-directory

After configuring, run:
```
npm install -g timesnap
```

To use:
```
timesnap "url" [options]
```

### <a name="cli-local-install" href="#cli-local-install">#</a> Local Install and Use
To install:
```
cd /path/to/installation/directory
npm install timesnap
```

To use:
```
node /path/to/installation/directory/node_modules/timesnap/cli.js "url" [options]
```

*Alternatively*:

To install:
```
cd /path/to/installation/directory
git clone https://github.com/tungs/timesnap.git
cd timesnap
npm install
```

To use:
```
node /path/to/installation/directory/timesnap/cli.js "url" [options]
```

### <a name="cli-url-use" href="#cli-url-use">#</a> Command Line *url*
The url can be a web url (e.g. `https://github.com`) or a relative path to the current working directory (e.g. `index.html`). If no url is specified, defaults to `index.html`. For urls with special characters (like `#` and `&`), enclose the urls with quotes.

### <a name="cli-examples" href="#cli-examples">#</a> Command Line Examples

**<a name="cli-example-default" href="#cli-example-default">#</a> Default behavior**:
```
timesnap
```
Opens `index.html` in the current working directory, sets the viewport to 800x600, captures at 60 frames per second for 5 virtual seconds, and saves the frames to `001.png` to `300.png` in the current working directory. The defaults may change in the future, so for long-term scripting, it's a good idea to explicitly pass those options, like in the following example.

**<a name="cli-example-viewport-fps-duration-output" href="#cli-example-viewport-fps-duration-output">#</a> Setting viewport size, frames per second, duration, and output pattern**:
```
timesnap index.html --viewport=800,600 --fps=60 --duration=5 --output-pattern="%03d.png"
```
Equivalent to the current default `timesnap` invocation, but with explicit options. Opens `index.html` in the current working directory, sets the viewport to 800x600, captures at 60 frames per second for 5 virtual seconds, and saves the frames to `001.png` to `300.png` in the current working directory.

**<a name="cli-example-selector" href="#cli-example-selector">#</a> Using a selector**:
```
timesnap drawing.html -S "canvas,svg" --output-pattern="frames/%03d.png"
```
Opens `drawing.html` in the current working directory, crops each frame to the bounding box of the first canvas or svg element, and captures frames using default settings (5 seconds @ 60fps), saving to `frames/001.png`... `frames/300.png` in the current working directory, making the directory `frames` if needed.

**<a name="cli-example-offsets" href="#cli-example-offsets">#</a> Using offsets**:
```
timesnap "https://tungs.github.io/truchet-tiles-original/#autoplay=true&switchStyle=random" \ 
  -S "#container" \ 
  --left=20 --top=40 --right=6 --bottom=30 \
  --duration=20 --output-directory=frames
```
Opens https://tungs.github.io/truchet-tiles-original/#autoplay=true&switchStyle=random (note the quotes in the url are necessary because of the `#` and `&`). Crops each frame to the `#container` element, with an additional crop of 20px, 40px, 6px, and 30px for the left, top, right, and bottom, respectively. Captures frames for 20 virtual seconds at 60fps to `frames/0001.png`... `frames/1200.png` in the current working directory, making the directory `frames` if needed.

**<a name="cli-example-piping" href="#cli-example-piping">#</a> Piping**:
```
timesnap https://breathejs.org/examples/Drawing-US-Counties.html \
  -V 1920,1080 -S "#draw-canvas" --fps=60 --duration=10 --even-width --even-height \
  --stdout | ffmpeg -framerate 60 -i pipe:0 -y -pix_fmt yuv420p movie.mp4
```
Opens https://breathejs.org/examples/Drawing-US-Counties.html, sets the viewport size to 1920x1080, crops each frame to the bounding box of #draw-canvas and records at 60 frames per second for ten virtual seconds and pipes the output to ffmpeg, which reads in the data from stdin, encodes the frames, and saves the result as `movie.mp4` in the current working directory. Does not save individual frames to disk. Uses the `--even-width` and `--even-height` options to ensure the dimensions of the frames are even numbers, which ffmpeg requires for certain encodings.

### <a name="cli-options" href="#cli-options">#</a> Command Line *options*
* <a name="cli-options-output-directory" href="#cli-options-output-directory">#</a> Output Directory: `-o`, `--output-directory` *directory*
    * Saves images to a *directory* (default './').
* <a name="cli-options-output-pattern" href="#cli-options-output-pattern">#</a> Output Pattern: `-O`, `--output-pattern` *pattern*
    * Saves each file to a *pattern* as a printf-style string (e.g. `image-%03d.png`).
* <a name="cli-options-fps" href="#cli-options-fps">#</a> Frame Rate: `-R`, `--fps` *frame rate*
    * Frame rate (in frames per virtual second) of capture (default: 60).
* <a name="cli-options-duration" href="#cli-options-duration">#</a> Duration: `-d`, `--duration` *seconds*
    * Duration of capture, in *seconds* (default: 5).
* <a name="cli-options-frames" href="#cli-options-frames">#</a> Frames: `--frames` *count*
    * Number of frames to capture.
* <a name="cli-options-selector" href="#cli-options-selector">#</a> Selector: `-S`, `--selector` "*selector*"
    * Crops each frame to the bounding box of the first item found by the CSS *selector*.
* <a name="cli-options-stdout" href="#cli-options-stdout">#</a> stdout: `--stdout`
    * Output images to stdout. Useful for piping. Command line only option.
* <a name="cli-options-viewport" href="#cli-options-viewport">#</a> Viewport: `-V`, `--viewport` *dimensions*
    * Viewport dimensions, in pixels. For example, `800` (for width) or `800,600` (for width and height).
* <a name="cli-options-start" href="#cli-options-start">#</a> Start: `-s`, `--start` *n seconds*
    * Runs code for n virtual seconds before saving any frames (default: 0).
* <a name="cli-options-x-offset" href="#cli-options-x-offset">#</a> X Offset: `-x`, `--x-offset` *pixels*
    * X offset of capture, in pixels (default: 0).
* <a name="cli-options-y-offset" href="#cli-options-y-offset">#</a> Y Offset: `-y`, `--y-offset` *pixels*
    * Y offset of capture, in pixels (default: 0).
* <a name="cli-options-width" href="#cli-options-width">#</a> Width: `-W`, `--width` *pixels*
    * Width of capture, in pixels.
* <a name="cli-options-height" href="#cli-options-height">#</a> Height: `-H`, `--height` *pixels*
    * Height of capture, in pixels.
* <a name="cli-options-even-width" href="#cli-options-width">#</a> Even Width: `--even-width`
    * Rounds width up to the nearest even number.
* <a name="cli-options-even-height" href="#cli-options-height">#</a> Even Height: `--even-height`
    * Rounds height up to the nearest even number.
* <a name="cli-options-transparent-background" href="#cli-options-transparent-background">#</a> Transparent Background: `--transparent-background`
    * Allows background to be transparent if there is no background styling.
* <a name="cli-options-left" href="#cli-options-left">#</a> Left: `-l`, `--left` *pixels*
    * Left edge of capture, in pixels. Equivalent to `--x-offset`.
* <a name="cli-options-right" href="#cli-options-right">#</a> Right: `-r`, `--right` *pixels*
    * Right edge of capture, in pixels. Ignored if `width` is specified.
* <a name="cli-options-top" href="#cli-options-top">#</a> Top: `-t`, `--top` *pixels*
    * Top edge of capture, in pixels. Equivalent to `--y-offset`.
* <a name="cli-options-bottom" href="#cli-options-bottom">#</a> Bottom: `-b`, `--bottom` *pixels*
    * Bottom edge of capture, in pixels. Ignored if `height` is specified.
* <a name="cli-options-load-delay" href="#cli-options-load-delay">#</a> Load Delay: `--load-delay` *n seconds*
    * Waits *n real seconds* after loading the page before starting to capture.
* <a name="cli-options-quiet" href="#cli-options-quiet">#</a> Quiet: `-q`, `--quiet`
    * Suppresses console logging.
* <a name="cli-options-version" href="#cli-options-version">#</a> Version: `-v`, `--version`
    * Displays version information. Immediately exits.
* <a name="cli-options-help" href="#cli-options-help">#</a> Help: `-h`, `--help`
    * Displays command line options. Immediately exits.

## <a name="node-use" href="#node-use">#</a> From Node.js
**timesnap** can also be included as a library inside Node.js programs.

### <a name="node-install" href="#node-install">#</a> Node Install
```
npm install timesnap --save
```

### <a name="node-examples" href="#node-examples">#</a> Node Examples

**<a name="node-example-basic" href="#node-example-basic">#</a> Basic Use:**
```node
const timesnap = require('timesnap');
timesnap({
  url: 'https://tungs.github.io/truchet-tiles-original/#autoplay=true&switchStyle=random',
  viewport: {
    width: 800,               // sets the viewport (window size) to 800x600
    height: 600
  },
  selector: '#container',     // crops each frame to the bounding box of '#container'
  left: 20, top: 40,          // further crops the left by 20px, and the top by 40px
  right: 6, bottom: 30,       // and the right by 6px, and the bottom by 30px
  fps: 30,                    // saves 30 frames for each virtual second
  duration: 20,               // for 20 virtual seconds 
  outputDirectory: 'frames'   // to frames/001.png... frames/600.png
                              // of the current working directory
}).then(function () {
  console.log('Done!');
});
```

**<a name="node-example-multiple" href="#node-example-multiple">#</a> Multiple pages (Requires Node v7.6.0 or higher):**
```node
const timesnap = require('timesnap');
var pages = [
  {
    url: 'https://tungs.github.io/truchet-tiles-original/#autoplay=true',
    outputDirectory: 'truchet-tiles'
  }, {
    url: 'https://breathejs.org/examples/Drawing-US-Counties.html',
    outputDirectory: 'counties'
  }
];
(async () => {
  for (let page of pages) {
    await timesnap({
      url: page.url,
      outputDirectory: page.outputDirectory,
      viewport: {
        width: 800,
        height: 600
      },
      duration: 20
    });
  }
})();
```

### <a name="node-api" href="#node-api">#</a> Node API

There are a few options for the Node API that are not accessible through the command line interface: `config.logToStdErr`, and `config.frameProcessor`.

**timesnap(config)**
*  <a name="js-api-config" href="#js-api-config">#</a> `config` &lt;[Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object)&gt;
    * <a name="js-config-url" href="#js-config-url">#</a> `url` &lt;[string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#String_type)&gt; The url to load. It can be a web url, like `https://github.com` or a relative path to the current working directory, like `index.html` (default: `index.html`).
    * <a name="js-config-output-directory" href="#js-config-output-directory">#</a> `outputDirectory` &lt;[string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#String_type)&gt; Saves images to a directory. Makes one if necessary.
    * <a name="js-config-output-pattern" href="#js-config-output-pattern">#</a> `outputPattern` &lt;[string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#String_type)&gt; Saves each file to a pattern as a printf-style string (e.g. `image-%03d.png`)
    * <a name="js-config-fps" href="#js-config-fps">#</a> `fps` &lt;[number](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#Number_type)&gt; frame rate, in frames per virtual second, of capture (default: `60`).
    * <a name="js-config-duration" href="#js-config-duration">#</a> `duration` &lt;[number](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#Number_type)&gt; Duration of capture, in seconds (default: `5`).
    * <a name="js-config-frames" href="#js-config-frames">#</a> `frames` &lt;[number](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#Number_type)&gt; Number of frames to capture. Overrides default fps or default duration.
    * <a name="js-config-selector" href="#js-config-selector">#</a> `selector` &lt;[string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#String_type)&gt; [CSS selector](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Selectors) of item to capture.
    * <a name="js-config-viewport" href="#js-config-viewport">#</a> `viewport` &lt;[Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object)&gt;
        * <a name="js-config-viewport-width" href="#js-config-viewport-width">#</a> `width` &lt;[number](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#Number_type)&gt; Width of viewport.
        * <a name="js-config-viewport-height" href="#js-config-viewport-height">#</a> `height` &lt;[number](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#Number_type)&gt; Height of viewport.
        * <a name="js-config-viewport-scale-factor" href="#js-config-viewport-scale-factor">#</a> `deviceScaleFactor` &lt;[number](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#Number_type)&gt; Device scale factor (default: `1`).
        * <a name="js-config-viewport-mobile" href="#js-config-viewport-mobile">#</a> `isMobile` &lt;[boolean](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#Boolean_type)&gt; Specifies whether the `meta viewport` tag should be used (default: `false`).
        * <a name="js-config-viewport-touch" href="#js-config-viewport-touch">#</a> `hasTouch` &lt;[boolean](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#Boolean_type)&gt; Specifies whether the viewport supports touch (default: `false`).
        * <a name="js-config-viewport-landscape" href="#js-config-viewport-landscape">#</a> `isLandscape` &lt;[boolean](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#Boolean_type)&gt; Specifies whether the viewport is in landscape mode (default: `false`).
    * <a name="js-config-start" href="#js-config-start">#</a> `start` &lt;[number](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#Number_type)&gt; Runs code for `config.start` virtual seconds before saving any frames (default: `0`).
    * <a name="js-config-x-offset" href="#js-config-x-offset">#</a> `xOffset` &lt;[number](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#Number_type)&gt; X offset of capture, in pixels (default: `0`).
    * <a name="js-config-y-offset" href="#js-config-y-offset">#</a> `yOffset` &lt;[number](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#Number_type)&gt; Y offset of capture, in pixels (default: `0`).
    * <a name="js-config-width" href="#js-config-width">#</a> `width` &lt;[number](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#Number_type)&gt; Width of capture, in pixels.
    * <a name="js-config-height" href="#js-config-height">#</a> `height` &lt;[number](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#Number_type)&gt; Height of capture, in pixels.
    * <a name="js-config-transparent-background" href="#js-config-transparent-background">#</a> `transparentBackground` &lt;[boolean](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#Boolean_type)&gt; Allows background to be transparent if there is no background styling (for pngs).
    * <a name="js-config-even-width" href="#js-config-even-width">#</a> `evenWidth` &lt;[boolean](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#Boolean_type)&gt; Rounds width up to the nearest even number.
    * <a name="js-config-even-height" href="#js-config-even-height">#</a> `evenHeight` &lt;[boolean](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#Boolean_type)&gt; Rounds height up to the nearest even number.
    * <a name="js-config-left" href="#js-config-left">#</a> `left` &lt;[number](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#Number_type)&gt; Left edge of capture, in pixels. Equivalent to `config.xOffset`.
    * <a name="js-config-right" href="#js-config-right">#</a> `right` &lt;[number](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#Number_type)&gt; Right edge of capture, in pixels. Ignored if `width` is specified.
    * <a name="js-config-top" href="#js-config-top">#</a> `top` &lt;[number](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#Number_type)&gt; Top edge of capture, in pixels. Equivalent to `config.yOffset`.
    * <a name="js-config-bottom" href="#js-config-bottom">#</a> `bottom` &lt;[number](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#Number_type)&gt; Bottom edge of capture, in pixels. Ignored if `height` is specified.
    * <a name="js-config-load-delay" href="#js-config-load-delay">#</a> `loadDelay` &lt;[number](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#Number_type)&gt; Wait `config.loadDelay` real seconds after loading (default: `0`).
    * <a name="js-config-quiet" href="#js-config-quiet">#</a> `quiet` &lt;[boolean](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#Boolean_type)&gt; Suppress console logging.
    * <a name="js-config-log-to-std-err" href="#js-config-log-to-std-err">#</a> `logToStdErr` &lt;[boolean](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#Boolean_type)&gt; Log to stderr instead of stdout. Doesn't do anything if `config.quiet` is set to true.
    * <a name="js-config-frame-processor" href="#js-config-frame-processor">#</a> `frameProcessor` &lt;[function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function)([Buffer](https://nodejs.org/api/buffer.html#buffer_class_buffer), [number](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#Number_type), [number](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#Number_type))&gt; A function that will be called after capturing each frame. If `config.outputDirectory` and `config.outputPattern` aren't specified, enabling this suppresses automatic file output. After capturing each frame, `config.frameProcessor` is called with three arguments, and if it returns a promise, capture will be paused until the promise resolves:
        * `screenshotData` &lt;[Buffer](https://nodejs.org/api/buffer.html#buffer_class_buffer)&gt; a buffer of the screenshot data.
        * `frameNumber` &lt;[number](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#Number_type)&gt; The current frame number (1 based).
        * `totalFrames` &lt;[number](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#Number_type)&gt; The total number of frames.

* <a name="js-api-return" href="#js-api-return">#</a> returns: &lt;[Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)&gt; resolves after all the frames have been captured.

## <a name="how-it-works" href="#how-it-works">#</a> How it works
**timesnap** uses puppeteer's `page.evaluateOnNewDocument` feature to automatically overwrite a page's native time-handling JavaScript functions (`new Date().getTime()`, `Date.now`, `performance.now`, `requestAnimationFrame`, `setTimeout`, `setInterval`, `cancelAnimationFrame`, `cancelTimeout`, and `cancelInterval`) to custom ones that use a virtual timeline, allowing for any computation to complete before taking a screenshot.

This work was inspired by [a talk by Noah Veltman](https://github.com/veltman/d3-unconf), who described manually altering a document's `Date.now` and `performance.now` functions and using `puppeteer` to change time and take snapshots. 
