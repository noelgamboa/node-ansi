node-ansi
=========

ANSI graphics parser for node.js.  Output to plain text, HTML, binary, animated GIF, or whatever you want.

####Installation

GIF output functionality uses [gifencoder](https://github.com/eugeneware/gifencoder) and [node-canvas](https://github.com/Automattic/node-canvas), which in turn rely on [Cairo](http://cairographics.org/).  Ensure that Cairo and its dependencies are installed before proceeding.

I'll add this to NPM eventually.  In the meantime, clone this repository to a subdirectory of your application's *node_modules* directory, cd to its path, and type:

```sh
npm install
```

####Usage

```js
var ANSI = require('./ansi.js'),
	fs = require('fs');

// Create a new ANSI object
var a = new ANSI();

// Load an ANSI graphic from a file
a.fromFile("./gnome.ans");

// Write the plain-text version of the graphic to a file
fs.writeFileSync("gnome.txt", a.plainText, { 'encoding' : 'binary' });

// Write the HTML version of the graphic to a file
fs.writeFileSync(
	"gnome.html",
	"<html><body>" + a.HTML + "</body></html>",
	{ 'encoding' : 'binary' }
);

// Write the CGA binary version of the graphic to a file
fs.writeFileSync("gnome.bin", a.binary.data);
// Report the width of the binary graphic, required by ANSI editors
console.log("Binary graphic width: %d", a.binary.width);

// Save the animated GIF version of the graphic to a file
a.toGIF({ 'filename' : "gnome.gif", 'loop' : false });
```

#### The ANSI object

#####Methods

- **fromFile("/path/to/file.ans")**
	- Loads and parses an ANSI graphic from a file
- **fromString(ansiString)**
	- Loads and parses an ANSI graphic from a string
- **toGIF(options)**
	- Converts the loaded ANSI graphic to an animated GIF, and saves it to a file
	- *options* is an object with the following properties:
		- *filename* (string) (required)
			- The path and filename to save the GIF to
		- *loop* (boolean) (default: false)
			- Whether or not the GIF should loop infinitely
		- *delay* (number) (default: 40)
			- Time between frames, in milliseconds
		- *charactersPerFrame* (number) (default: 10)
			- How many new characters appear in each frame of the GIF
				- If you set this to 1, the GIF will show the ANSI being drawn one character at a time, which is nice, however ...
				- If you set this to 1, it will take a long time to generate the GIF
				- Adjusting this number has a noticeable impact on filesize
				- You'll probably want to adjust *delay* along with this value
		- *quality* (number) (default: 20)
			- The image quality, on a scale of 1 to 20, where 1 is best and 20 is worst
				- I haven't noticed a visible difference between 1 and 20
				- GIFs generate a lot faster when the quality is set to the lowest (yet highest-numbered) value
- **toPNG(filename)**
	- Converts the loaded ANSI graphic to a PNG, and saves it to a file
	- *filename* is the path to the file to save in

#####Properties

- **data**
	- An array of objects representing each explicitly drawn character in the graphic
	- Elements in this array appear in the sequence that they were parsed from the file
		- The parser handles cursor-positioning sequences
		- The parser handles clear-screen and clear-to-EOL sequences
		- Characters will not necessarily be in left-to-right, top-to-bottom sequence
			- This is useful for handling animated ANSIs
	- The elements in this array are objects of the following format:

```js
{	cursor : {
		x : *number*,			// X-coordinate of character
		y : *number*			// Y-coordinate of character
	},
	graphics : {
		bright : *boolean*,		// Bold/bright foreground colour
		blink : *boolean*,		// Blinking
		foreground : *number*,	// Foreground colour (30 - 47)
		background : *number*	// Background colour (40 - 47)
	},
	chr : *string*				// The character itself
}
```
- **matrix**
	- An object representing every character-cell in the graphic, from top to bottom, left to right
	- The object takes the following format:

```js
{	0 : { 		// Line 0 of the graphic
		0 : {	// Column 1 of line 0
			graphics : {
				bright : *boolean*,
				blink : *boolean*,
				foreground : *number*,
				background : *number*
			},
			chr : *string*
		}
		...
	}
	...
}
```

- **plainText**
	- A string representation of the graphic with all colour/bright/blink attributes removed, with line-endings in place
		- (ie. the ANSI graphic converted to boring text.)

- **binary**
	- An object with the following properties:
		- **width** (number)
			- The width of the binary graphic
		- **data** (Buffer)
			- A buffer of [ chr, attr, chr, attr, ... ] uint8s

- **HTML**
	- An HTML &lt;pre&gt; block containing the graphic, with colorized regions in styled &lt;span&gt; elements, and characters encoded as HTML entities as required
		- Opening and closing &lt;html&gt; &lt;head&gt; and &lt;body&gt; tags are not included in this string