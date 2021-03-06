# convert-rich-text

Convert a [rich-text](https://github.com/ottypes/rich-text) document (i.e. insert-only delta) into HTML.

[![Build Status](https://travis-ci.org/thomsbg/convert-rich-text.svg?branch=master)](https://travis-ci.org/thomsbg/convert-rich-text)

## Install

```bash
$ npm install [--save] convert-rich-text
```

## Usage

```javascript
var convert = require('convert-rich-text');
var html = convert(delta, formats, options); // last argument optional
```

Specify an object of format names and config values, using the same conventions
as QuillJS:

```javascript
var convert = require('convert-rich-text');
var delta = {ops: [
  {insert: 'Hello, World!'},
  {insert: '\n', attributes: {firstheader: true}},
  {insert: 'This is a '},
  {insert: 'demo', attributes: {bold: true}},
  {insert: ' of convert-rich-text\n'},
  {insert: 1, attributes: {image: 'monkey.png'}
]};
var formats = {
  firstheader: { type: 'line', tag: 'H1' },
  bold: { tag: 'EM' },
  image: { tag: 'IMG', attribute: 'src' }
};
var options = {
  blockTag: 'P',
  inlineTag: 'SPAN'
};
var html = convert(delta, formats, options);
console.log(html);
```

Result:

```html
<h1>Hello, World!</h1>
<p>This is a <b>demo</b> of convert-rich-text</p>
<p><img src="monkey.png"></p>
```

## Formats

The following options are supported for configuring a format (adapted from QuillJS):

`type: 'line'` -- make this format apply only to the line as a whole (via attributes to newline characters).

`tag: 'B'` -- wrap the applicable text in that tag

`parentTag: 'UL', tag: 'LI'` -- used for line formats to create multi-level tag structures; consecutive lines with the same `parentTag` will share the parent.

`attribute: 'href'` -- set an attribute using the given name and the value from the delta

`class: 'cursor-'` -- add a class with the given prefix, e.g. convert({ ops: [{ insert: 'hello', attributes: { cursor: 1234 } }] }, { cursor: { class: 'cursor-' })` => `<span class="cursor-1234">hello</span>`

`style: 'fontSize'` -- set an inline style using the given name and the value from the delta

`add: function(node, value[, dom])` -- a hook for custom behavior, runs after logic for other options. e.g.

```javascript
convert(delta, {
  // wrap in a span, and set data attributes,
  // e.g. `{insert: 'hello', { data: { foo: 'bar' } } }` => `<span data-foo="bar">hello</span>`
  data: { tag: 'span', add: function(node, data) {
    Object.keys(data).forEach(function(key) {
      node.dataset[key] = data[key];
    });
    return node;
  } },
  // repeat the line N times
  // e.g. `{insert: 'hello\n', { times: 3 } }` => `<p>hello</p><p>hello</p><p>hello</p>`
  repeat: { type: 'line', add: function(node, value) {
    var clone = node;
    for (var i = 1, n = parseInt(value); i < n; i++) {
      clone = node.cloneNode(true);
      node.parentNode.appendChild(clone);
    }
    return clone;
  } }
});
```

## Options

Each line of rich-text is wrapped with a block element (default `<div>`).

attribute-, class- and style-based formats wrap text with an inline element if there is no other tag to work on (default `<span>`).

You can change these tags with the `blockTag` and `inlineTag` options:

```javascript
convert(delta, formats, { blockTag: 'FIGURE', inlineTag: 'INS' });
```

## Changelog

- `2.0.2` [Relax jsdom and node version requirements]
- `2.0.0` [Server-side support via jsdom](https://github.com/thomsbg/convert-rich-text/pull/2), node version locked to <=0.12
- `1.2.1` Beginning of changelog

## Development

Run `npm start` to spin up a static web server and watchify.
Open http://localhost:8080/test in a browser to run and debug tests.

## Credit

Thank you [@kbjr](https://github.com/kbjr) for https://github.com/UmbraEngineering/quilljs-renderer on which this project is forked.
