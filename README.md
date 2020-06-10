# Enable `window` and `document`

The goal of this package is to work as a quick-and-dirty one-liner that will allow Node to `require` and otherwise execute traditional browser code without throwing errors.  Simply set the `window` and `document` globals with:

```
require('enable-window-document');
```

No variable assignment required, just call it!

## Use cases
The specific need for this functionality came from the `web-widgets` package, which generates widget trees using DOM operations like `document.createElement()`.  The Node runtime cannot build out this widget tree by default, as it does not have access to the `window` and `document` variables, so it throws a `ReferenceError`.

By importing this package (which depends on JSDOM), we can expose the `window` and `document` globals to the whole project, meaning we can write all of our browser-optimized (and DOM-heavy) code in a file like `browser.js`, but still use that same code for server-side rendering in Node with `require('browser.js')`. 

 In Node, the widget tree is built out in the virtual DOM and then exported as flat HTML with `outerHTML`.  In the browser, the DOM is manipulated directly on-the-fly (i.e. with `Node.appendChild`).  

**Additionally, we can Closure Compile our browser code before depending on it in Node, meaning builds are as small and performant as possible. No webpack, no extra polyfills.**

## Example
Won't work:

```
console.log(document.createElement('a'));

>   ReferenceError: document is not defined
```

Works like a charm:
```
require('enable-window-document');
console.log(document.createElement('a'));

>   HTMLAnchorElement {Symbol(impl): HTMLAnchorElementImpl}
```

## Implementation
This package simply creates a blank JSDOM with four lines of code, and stores the global `window` and `document` variables, which point to the empty DOM:

```
let JSDOM = require('jsdom'),
    DOM = new JSDOM.JSDOM(`<html><body></body></html>`);

global.window = DOM.window,
global.document = window.document;
```

## Digressions
It should really take *zero* lines to run browser-compatible JS in Node, but one line will do for now. The important part is that instead of writing everything for Node and then using `browserify` and other tools to polyfill it for the browser, we can write strict code for the browser and force Node to interpret Javascript in the same way the browser does.  

Why the Node runtime does not expose `window` and `document` OOTB is a valid question, as whatever rationale exists behind that decision cannot be more concrete than the principle that code which can execute in a browser should be able to execute in the Node runtime. Javascript output from one environment should === the same Javascript run in another. [It's supposed to be a lingua franca!](https://i.imgur.com/TwkD81I.jpeg)