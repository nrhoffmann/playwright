# class: Page
* extends: [EventEmitter](https://nodejs.org/api/events.html#events_class_eventemitter)

Page provides methods to interact with a single tab in a [Browser], or an [extension background
page](https://developer.chrome.com/extensions/background_pages) in Chromium. One [Browser] instance might have multiple
[Page] instances.

This example creates a page, navigates it to a URL, and then saves a screenshot:

```js
const { webkit } = require('playwright');  // Or 'chromium' or 'firefox'.

(async () => {
  const browser = await webkit.launch();
  const context = await browser.newContext();
  const page = await context.newPage();
  await page.goto('https://example.com');
  await page.screenshot({path: 'screenshot.png'});
  await browser.close();
})();
```

The Page class emits various events (described below) which can be handled using any of Node's native
[`EventEmitter`](https://nodejs.org/api/events.html#events_class_eventemitter) methods, such as `on`, `once` or
`removeListener`.

This example logs a message for a single page `load` event:

```js
page.once('load', () => console.log('Page loaded!'));
```

To unsubscribe from events use the `removeListener` method:

```js
function logRequest(interceptedRequest) {
  console.log('A request was made:', interceptedRequest.url());
}
page.on('request', logRequest);
// Sometime later...
page.removeListener('request', logRequest);
```

## event: Page.close

Emitted when the page closes.

## event: Page.console
- type: <[ConsoleMessage]>

Emitted when JavaScript within the page calls one of console API methods, e.g. `console.log` or `console.dir`. Also
emitted if the page throws an error or a warning.

The arguments passed into `console.log` appear as arguments on the event handler.

An example of handling `console` event:

```js
page.on('console', msg => {
  for (let i = 0; i < msg.args().length; ++i)
    console.log(`${i}: ${msg.args()[i]}`);
});
page.evaluate(() => console.log('hello', 5, {foo: 'bar'}));
```

## event: Page.crash

Emitted when the page crashes. Browser pages might crash if they try to allocate too much memory. When the page crashes,
ongoing and subsequent operations will throw.

The most common way to deal with crashes is to catch an exception:

```js
try {
  // Crash might happen during a click.
  await page.click('button');
  // Or while waiting for an event.
  await page.waitForEvent('popup');
} catch (e) {
  // When the page crashes, exception message contains 'crash'.
}
```

However, when manually listening to events, it might be useful to avoid stalling when the page crashes. In this case,
handling `crash` event helps:

```js
await new Promise((resolve, reject) => {
  page.on('requestfinished', async request => {
    if (await someProcessing(request))
      resolve(request);
  });
  page.on('crash', error => reject(error));
});
```

## event: Page.dialog
- type: <[Dialog]>

Emitted when a JavaScript dialog appears, such as `alert`, `prompt`, `confirm` or `beforeunload`. Playwright can respond
to the dialog via [`method: Dialog.accept`] or [`method: Dialog.dismiss`] methods.

## event: Page.domcontentloaded

Emitted when the JavaScript [`DOMContentLoaded`](https://developer.mozilla.org/en-US/docs/Web/Events/DOMContentLoaded)
event is dispatched.

## event: Page.download
- type: <[Download]>

Emitted when attachment download started. User can access basic file operations on downloaded content via the passed
[Download] instance.

> **NOTE** Browser context **must** be created with the `acceptDownloads` set to `true` when user needs access to the
downloaded content. If `acceptDownloads` is not set or set to `false`, download events are emitted, but the actual
download is not performed and user has no access to the downloaded files.

## event: Page.filechooser
- type: <[FileChooser]>

Emitted when a file chooser is supposed to appear, such as after clicking the  `<input type=file>`. Playwright can
respond to it via setting the input files using [`method: FileChooser.setFiles`] that can be uploaded after that.

```js
page.on('filechooser', async (fileChooser) => {
  await fileChooser.setFiles('/tmp/myfile.pdf');
});
```

## event: Page.frameattached
- type: <[Frame]>

Emitted when a frame is attached.

## event: Page.framedetached
- type: <[Frame]>

Emitted when a frame is detached.

## event: Page.framenavigated
- type: <[Frame]>

Emitted when a frame is navigated to a new url.

## event: Page.load

Emitted when the JavaScript [`load`](https://developer.mozilla.org/en-US/docs/Web/Events/load) event is dispatched.

## event: Page.pageerror
- type: <[Error]>

Emitted when an uncaught exception happens within the page.

## event: Page.popup
- type: <[Page]>

Emitted when the page opens a new tab or window. This event is emitted in addition to the [`event:
BrowserContext.page`], but only for popups relevant to this page.

The earliest moment that page is available is when it has navigated to the initial url. For example, when opening a
popup with `window.open('http://example.com')`, this event will fire when the network request to "http://example.com" is
done and its response has started loading in the popup.

```js
const [popup] = await Promise.all([
  page.waitForEvent('popup'),
  page.evaluate(() => window.open('https://example.com')),
]);
console.log(await popup.evaluate('location.href'));
```

> **NOTE** Use [`method: Page.waitForLoadState`] to wait until the page gets to a particular state (you should not
need it in most cases).

## event: Page.request
- type: <[Request]>

Emitted when a page issues a request. The [request] object is read-only. In order to intercept and mutate requests, see
[`method: Page.route`] or [`method: BrowserContext.route`].

## event: Page.requestfailed
- type: <[Request]>

Emitted when a request fails, for example by timing out.

> **NOTE** HTTP Error responses, such as 404 or 503, are still successful responses from HTTP standpoint, so request
will complete with [`event: Page.requestfinished`] event and not with [`event: Page.requestfailed`].

## event: Page.requestfinished
- type: <[Request]>

Emitted when a request finishes successfully after downloading the response body. For a successful response, the
sequence of events is `request`, `response` and `requestfinished`.

## event: Page.response
- type: <[Response]>

Emitted when [response] status and headers are received for a request. For a successful response, the sequence of events
is `request`, `response` and `requestfinished`.

## event: Page.websocket
- type: <[WebSocket]>

Emitted when <[WebSocket]> request is sent.

## event: Page.worker
- type: <[Worker]>

Emitted when a dedicated [WebWorker](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API) is spawned by the
page.

## async method: Page.$
- returns: <[null]|[ElementHandle]>

The method finds an element matching the specified selector within the page. If no elements match the selector, the
return value resolves to `null`.

Shortcut for main frame's [`method: Frame.$`].

### param: Page.$.selector = %%-query-selector-%%

## async method: Page.$$
- returns: <[Array]<[ElementHandle]>>

The method finds all elements matching the specified selector within the page. If no elements match the selector, the
return value resolves to `[]`.

Shortcut for main frame's [`method: Frame.$$`].

### param: Page.$$.selector = %%-query-selector-%%

## async method: Page.$eval
- returns: <[Serializable]>

The method finds an element matching the specified selector within the page and passes it as a first argument to
[`param: pageFunction`]. If no elements match the selector, the method throws an error. Returns the value of [`param:
pageFunction`].

If [`param: pageFunction`] returns a [Promise], then [`method: Page.$eval`] would wait for the promise to resolve and return its
value.

Examples:

```js
const searchValue = await page.$eval('#search', el => el.value);
const preloadHref = await page.$eval('link[rel=preload]', el => el.href);
const html = await page.$eval('.main-container', (e, suffix) => e.outerHTML + suffix, 'hello');
```

Shortcut for main frame's [`method: Frame.$eval`].

### param: Page.$eval.selector = %%-query-selector-%%

### param: Page.$eval.pageFunction
- `pageFunction` <[function]\([Element]\)>

Function to be evaluated in browser context

### param: Page.$eval.arg
- `arg` <[EvaluationArgument]>

Optional argument to pass to [`param: pageFunction`]

## async method: Page.$$eval
- returns: <[Serializable]>

The method finds all elements matching the specified selector within the page and passes an array of matched elements as
a first argument to [`param: pageFunction`]. Returns the result of [`param: pageFunction`] invocation.

If [`param: pageFunction`] returns a [Promise], then [`method: Page.$$eval`] would wait for the promise to resolve and return
its value.

Examples:

```js
const divsCounts = await page.$$eval('div', (divs, min) => divs.length >= min, 10);
```

### param: Page.$$eval.selector = %%-query-selector-%%

### param: Page.$$eval.pageFunction
- `pageFunction` <[function]\([Array]<[Element]>\)>

Function to be evaluated in browser context

### param: Page.$$eval.arg
- `arg` <[EvaluationArgument]>

Optional argument to pass to [`param: pageFunction`]

## property: Page.accessibility
- type: <[Accessibility]>

## async method: Page.addInitScript

Adds a script which would be evaluated in one of the following scenarios:
* Whenever the page is navigated.
* Whenever the child frame is attached or navigated. In this case, the script is evaluated in the context of the newly attached frame.

The script is evaluated after the document was created but before any of its scripts were run. This is useful to amend
the JavaScript environment, e.g. to seed `Math.random`.

An example of overriding `Math.random` before the page loads:

```js
// preload.js
Math.random = () => 42;

// In your playwright script, assuming the preload.js file is in same directory
const preloadFile = fs.readFileSync('./preload.js', 'utf8');
await page.addInitScript(preloadFile);
```

> **NOTE** The order of evaluation of multiple scripts installed via [`method: BrowserContext.addInitScript`] and
[`method: Page.addInitScript`] is not defined.

### param: Page.addInitScript.script
* langs: js
- `script` <[function]|[string]|[Object]>
  - `path` <[path]> Path to the JavaScript file. If `path` is a relative path, then it is resolved relative to the current working directory. Optional.
  - `content` <[string]> Raw script content. Optional.

Script to be evaluated in the page.

### param: Page.addInitScript.arg
* langs: js
- `arg` <[Serializable]>

Optional argument to pass to [`param: script`] (only supported when passing a function).

## async method: Page.addScriptTag
- returns: <[ElementHandle]>

Adds a `<script>` tag into the page with the desired url or content. Returns the added tag when the script's onload
fires or when the script content was injected into frame.

Shortcut for main frame's [`method: Frame.addScriptTag`].

### param: Page.addScriptTag.params
- `params` <[Object]>
  - `url` <[string]> URL of a script to be added. Optional.
  - `path` <[path]> Path to the JavaScript file to be injected into frame. If `path` is a relative path, then it is resolved relative to the current working directory. Optional.
  - `content` <[string]> Raw JavaScript content to be injected into frame. Optional.
  - `type` <[string]> Script type. Use 'module' in order to load a Javascript ES6 module. See [script](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/script) for more details. Optional.

## async method: Page.addStyleTag
- returns: <[ElementHandle]>

Adds a `<link rel="stylesheet">` tag into the page with the desired url or a `<style type="text/css">` tag with the
content. Returns the added tag when the stylesheet's onload fires or when the CSS content was injected into frame.

Shortcut for main frame's [`method: Frame.addStyleTag`].

### param: Page.addStyleTag.params
- `params` <[Object]>
  - `url` <[string]> URL of the `<link>` tag. Optional.
  - `path` <[path]> Path to the CSS file to be injected into frame. If `path` is a relative path, then it is resolved relative to the current working directory. Optional.
  - `content` <[string]> Raw CSS content to be injected into frame. Optional.

## async method: Page.bringToFront

Brings page to front (activates tab).

## async method: Page.check

This method checks an element matching [`param: selector`] by performing the following steps:
1. Find an element match matching [`param: selector`]. If there is none, wait until a matching element is attached to the DOM.
1. Ensure that matched element is a checkbox or a radio input. If not, this method rejects. If the element is already checked, this method returns immediately.
1. Wait for [actionability](./actionability.md) checks on the matched element, unless [`option: force`] option is set. If the element is detached during the checks, the whole action is retried.
1. Scroll the element into view if needed.
1. Use [`property: Page.mouse`] to click in the center of the element.
1. Wait for initiated navigations to either succeed or fail, unless [`option: noWaitAfter`] option is set.
1. Ensure that the element is now checked. If not, this method rejects.

When all steps combined have not finished during the specified [`option: timeout`], this method rejects with a
[TimeoutError]. Passing zero timeout disables this.

Shortcut for main frame's [`method: Frame.check`].

### param: Page.check.selector = %%-input-selector-%%

### option: Page.check.force = %%-input-force-%%

### option: Page.check.noWaitAfter = %%-input-no-wait-after-%%

### option: Page.check.timeout = %%-input-timeout-%%

## async method: Page.click

This method clicks an element matching [`param: selector`] by performing the following steps:
1. Find an element match matching [`param: selector`]. If there is none, wait until a matching element is attached to the DOM.
1. Wait for [actionability](./actionability.md) checks on the matched element, unless [`option: force`] option is set. If the element is detached during the checks, the whole action is retried.
1. Scroll the element into view if needed.
1. Use [`property: Page.mouse`] to click in the center of the element, or the specified [`option: position`].
1. Wait for initiated navigations to either succeed or fail, unless [`option: noWaitAfter`] option is set.

When all steps combined have not finished during the specified [`option: timeout`], this method rejects with a
[TimeoutError]. Passing zero timeout disables this.

Shortcut for main frame's [`method: Frame.click`].

### param: Page.click.selector = %%-input-selector-%%

### option: Page.click.button = %%-input-button-%%

### option: Page.click.clickCount = %%-input-click-count-%%

### option: Page.click.delay = %%-input-down-up-delay-%%

### option: Page.click.position = %%-input-position-%%

### option: Page.click.modifiers = %%-input-modifiers-%%

### option: Page.click.force = %%-input-force-%%

### option: Page.click.noWaitAfter = %%-input-no-wait-after-%%

### option: Page.click.timeout = %%-input-timeout-%%

## async method: Page.close

If [`option: runBeforeUnload`] is `false`, does not run any unload handlers and waits for the page to be closed. If
[`option: runBeforeUnload`] is `true` the method will run unload handlers, but will **not** wait for the page to
close.

By default, `page.close()` **does not** run `beforeunload` handlers.

> **NOTE** if [`option: runBeforeUnload`] is passed as true, a `beforeunload` dialog might be summoned
> and should be handled manually via [`event: Page.dialog`] event.

### option: Page.close.runBeforeUnload
- `runBeforeUnload` <[boolean]>

Defaults to `false`. Whether to run the [before
unload](https://developer.mozilla.org/en-US/docs/Web/Events/beforeunload) page handlers.

## async method: Page.content
- returns: <[string]>

Gets the full HTML contents of the page, including the doctype.

## method: Page.context
- returns: <[BrowserContext]>

Get the browser context that the page belongs to.

## property: Page.coverage
* langs: js
- type: <[null]|[ChromiumCoverage]>

Browser-specific Coverage implementation, only available for Chromium atm. See
[ChromiumCoverage](#class-chromiumcoverage) for more details.

## async method: Page.dblclick

This method double clicks an element matching [`param: selector`] by performing the following steps:
1. Find an element match matching [`param: selector`]. If there is none, wait until a matching element is attached to the DOM.
1. Wait for [actionability](./actionability.md) checks on the matched element, unless [`option: force`] option is set. If the element is detached during the checks, the whole action is retried.
1. Scroll the element into view if needed.
1. Use [`property: Page.mouse`] to double click in the center of the element, or the specified [`option: position`].
1. Wait for initiated navigations to either succeed or fail, unless [`option: noWaitAfter`] option is set. Note that if the first click of the `dblclick()` triggers a navigation event, this method will reject.

When all steps combined have not finished during the specified [`option: timeout`], this method rejects with a
[TimeoutError]. Passing zero timeout disables this.

> **NOTE** `page.dblclick()` dispatches two `click` events and a single `dblclick` event.

Shortcut for main frame's [`method: Frame.dblclick`].

### param: Page.dblclick.selector = %%-input-selector-%%

### option: Page.dblclick.button = %%-input-button-%%

### option: Page.dblclick.delay = %%-input-down-up-delay-%%

### option: Page.dblclick.position = %%-input-position-%%

### option: Page.dblclick.modifiers = %%-input-modifiers-%%

### option: Page.dblclick.force = %%-input-force-%%

### option: Page.dblclick.noWaitAfter = %%-input-no-wait-after-%%

### option: Page.dblclick.timeout = %%-input-timeout-%%

## async method: Page.dispatchEvent

The snippet below dispatches the `click` event on the element. Regardless of the visibility state of the elment, `click`
is dispatched. This is equivalend to calling
[element.click()](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/click).

```js
await page.dispatchEvent('button#submit', 'click');
```

Under the hood, it creates an instance of an event based on the given [`param: type`], initializes it with [`param:
eventInit`] properties and dispatches it on the element. Events are `composed`, `cancelable` and bubble by default.

Since [`param: eventInit`] is event-specific, please refer to the events documentation for the lists of initial
properties:
* [DragEvent](https://developer.mozilla.org/en-US/docs/Web/API/DragEvent/DragEvent)
* [FocusEvent](https://developer.mozilla.org/en-US/docs/Web/API/FocusEvent/FocusEvent)
* [KeyboardEvent](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/KeyboardEvent)
* [MouseEvent](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/MouseEvent)
* [PointerEvent](https://developer.mozilla.org/en-US/docs/Web/API/PointerEvent/PointerEvent)
* [TouchEvent](https://developer.mozilla.org/en-US/docs/Web/API/TouchEvent/TouchEvent)
* [Event](https://developer.mozilla.org/en-US/docs/Web/API/Event/Event)

You can also specify `JSHandle` as the property value if you want live objects to be passed into the event:

```js
// Note you can only create DataTransfer in Chromium and Firefox
const dataTransfer = await page.evaluateHandle(() => new DataTransfer());
await page.dispatchEvent('#source', 'dragstart', { dataTransfer });
```

### param: Page.dispatchEvent.selector = %%-input-selector-%%

### param: Page.dispatchEvent.type
- `type` <[string]>

DOM event type: `"click"`, `"dragstart"`, etc.

### param: Page.dispatchEvent.eventInit
- `eventInit` <[EvaluationArgument]>

Optional event-specific initialization properties.

### option: Page.dispatchEvent.timeout = %%-input-timeout-%%

## async method: Page.emulateMedia

```js
await page.evaluate(() => matchMedia('screen').matches);
// → true
await page.evaluate(() => matchMedia('print').matches);
// → false

await page.emulateMedia({ media: 'print' });
await page.evaluate(() => matchMedia('screen').matches);
// → false
await page.evaluate(() => matchMedia('print').matches);
// → true

await page.emulateMedia({});
await page.evaluate(() => matchMedia('screen').matches);
// → true
await page.evaluate(() => matchMedia('print').matches);
// → false
```

```js
await page.emulateMedia({ colorScheme: 'dark' });
await page.evaluate(() => matchMedia('(prefers-color-scheme: dark)').matches);
// → true
await page.evaluate(() => matchMedia('(prefers-color-scheme: light)').matches);
// → false
await page.evaluate(() => matchMedia('(prefers-color-scheme: no-preference)').matches);
// → false
```

### param: Page.emulateMedia.params
- `params` <[Object]>
  - `media` <[null]|"screen"|"print"> Changes the CSS media type of the page. The only allowed values are `'screen'`, `'print'` and `null`. Passing `null` disables CSS media emulation. Omitting `media` or passing `undefined` does not change the emulated value. Optional.
  - `colorScheme` <[null]|"light"|"dark"|"no-preference"> Emulates `'prefers-colors-scheme'` media feature, supported values are `'light'`, `'dark'`, `'no-preference'`. Passing `null` disables color scheme emulation. Omitting `colorScheme` or passing `undefined` does not change the emulated value. Optional.

## async method: Page.evaluate
- returns: <[Serializable]>

Returns the value of the [`param: pageFunction`] invocation.

If the function passed to the `page.evaluate` returns a [Promise], then `page.evaluate` would wait for the promise to
resolve and return its value.

If the function passed to the `page.evaluate` returns a non-[Serializable] value, then `page.evaluate` resolves to
`undefined`. DevTools Protocol also supports transferring some additional values that are not serializable by `JSON`:
`-0`, `NaN`, `Infinity`, `-Infinity`, and bigint literals.

Passing argument to [`param: pageFunction`]:

```js
const result = await page.evaluate(([x, y]) => {
  return Promise.resolve(x * y);
}, [7, 8]);
console.log(result); // prints "56"
```

A string can also be passed in instead of a function:

```js
console.log(await page.evaluate('1 + 2')); // prints "3"
const x = 10;
console.log(await page.evaluate(`1 + ${x}`)); // prints "11"
```

[ElementHandle] instances can be passed as an argument to the `page.evaluate`:

```js
const bodyHandle = await page.$('body');
const html = await page.evaluate(([body, suffix]) => body.innerHTML + suffix, [bodyHandle, 'hello']);
await bodyHandle.dispose();
```

Shortcut for main frame's [`method: Frame.evaluate`].

### param: Page.evaluate.pageFunction
- `pageFunction` <[function]|[string]>

Function to be evaluated in the page context

### param: Page.evaluate.arg
- `arg` <[EvaluationArgument]>

Optional argument to pass to [`param: pageFunction`]

## async method: Page.evaluateHandle
- returns: <[JSHandle]>

Returns the value of the [`param: pageFunction`] invocation as in-page object (JSHandle).

The only difference between `page.evaluate` and `page.evaluateHandle` is that `page.evaluateHandle` returns in-page
object (JSHandle).

If the function passed to the `page.evaluateHandle` returns a [Promise], then `page.evaluateHandle` would wait for the
promise to resolve and return its value.

A string can also be passed in instead of a function:

```js
const aHandle = await page.evaluateHandle('document'); // Handle for the 'document'
```

[JSHandle] instances can be passed as an argument to the `page.evaluateHandle`:

```js
const aHandle = await page.evaluateHandle(() => document.body);
const resultHandle = await page.evaluateHandle(body => body.innerHTML, aHandle);
console.log(await resultHandle.jsonValue());
await resultHandle.dispose();
```

### param: Page.evaluateHandle.pageFunction
- `pageFunction` <[function]|[string]>

Function to be evaluated in the page context

### param: Page.evaluateHandle.arg
- `arg` <[EvaluationArgument]>

Optional argument to pass to [`param: pageFunction`]

## async method: Page.exposeBinding

The method adds a function called [`param: name`] on the `window` object of every frame in this page. When called, the
function executes [`param: callback`] and returns a [Promise] which resolves to the return value of [`param:
callback`]. If the [`param: callback`] returns a [Promise], it will be awaited.

The first argument of the [`param: callback`] function contains information about the caller: `{
browserContext: BrowserContext, page: Page, frame: Frame }`.

See [`method: BrowserContext.exposeBinding`] for the context-wide version.

> **NOTE** Functions installed via `page.exposeBinding` survive navigations.

An example of exposing page URL to all frames in a page:

```js
const { webkit } = require('playwright');  // Or 'chromium' or 'firefox'.

(async () => {
  const browser = await webkit.launch({ headless: false });
  const context = await browser.newContext();
  const page = await context.newPage();
  await page.exposeBinding('pageURL', ({ page }) => page.url());
  await page.setContent(`
    <script>
      async function onClick() {
        document.querySelector('div').textContent = await window.pageURL();
      }
    </script>
    <button onclick="onClick()">Click me</button>
    <div></div>
  `);
  await page.click('button');
})();
```

An example of passing an element handle:

```js
await page.exposeBinding('clicked', async (source, element) => {
  console.log(await element.textContent());
}, { handle: true });
await page.setContent(`
  <script>
    document.addEventListener('click', event => window.clicked(event.target));
  </script>
  <div>Click me</div>
  <div>Or click me</div>
`);
```

### param: Page.exposeBinding.name
- `name` <[string]>

Name of the function on the window object.

### param: Page.exposeBinding.callback
- `callback` <[function]>

Callback function that will be called in the Playwright's context.

### option: Page.exposeBinding.handle
- `handle` <[boolean]>

Whether to pass the argument as a handle, instead of passing by value. When passing a handle, only one argument is
supported. When passing by value, multiple arguments are supported.

## async method: Page.exposeFunction

The method adds a function called [`param: name`] on the `window` object of every frame in the page. When called, the
function executes [`param: callback`] and returns a [Promise] which resolves to the return value of [`param:
callback`].

If the [`param: callback`] returns a [Promise], it will be awaited.

See [`method: BrowserContext.exposeFunction`] for context-wide exposed function.

> **NOTE** Functions installed via `page.exposeFunction` survive navigations.

An example of adding an `md5` function to the page:

```js
const { webkit } = require('playwright');  // Or 'chromium' or 'firefox'.
const crypto = require('crypto');

(async () => {
  const browser = await webkit.launch({ headless: false });
  const page = await browser.newPage();
  await page.exposeFunction('md5', text => crypto.createHash('md5').update(text).digest('hex'));
  await page.setContent(`
    <script>
      async function onClick() {
        document.querySelector('div').textContent = await window.md5('PLAYWRIGHT');
      }
    </script>
    <button onclick="onClick()">Click me</button>
    <div></div>
  `);
  await page.click('button');
})();
```

An example of adding a `window.readfile` function to the page:

```js
const { chromium } = require('playwright');  // Or 'firefox' or 'webkit'.
const fs = require('fs');

(async () => {
  const browser = await chromium.launch();
  const page = await browser.newPage();
  page.on('console', msg => console.log(msg.text()));
  await page.exposeFunction('readfile', async filePath => {
    return new Promise((resolve, reject) => {
      fs.readFile(filePath, 'utf8', (err, text) => {
        if (err)
          reject(err);
        else
          resolve(text);
      });
    });
  });
  await page.evaluate(async () => {
    // use window.readfile to read contents of a file
    const content = await window.readfile('/etc/hosts');
    console.log(content);
  });
  await browser.close();
})();
```

### param: Page.exposeFunction.name
- `name` <[string]>

Name of the function on the window object

### param: Page.exposeFunction.callback
- `callback` <[function]>

Callback function which will be called in Playwright's context.

## async method: Page.fill

This method waits for an element matching [`param: selector`], waits for [actionability](./actionability.md) checks,
focuses the element, fills it and triggers an `input` event after filling. If the element matching [`param: selector`]
is not an `<input>`, `<textarea>` or `[contenteditable]` element, this method throws an error. Note that you can pass an
empty string to clear the input field.

To send fine-grained keyboard events, use [`method: Page.type`].

Shortcut for main frame's [`method: Frame.fill`]

### param: Page.fill.selector = %%-input-selector-%%

### param: Page.fill.value
- `value` <[string]>

Value to fill for the `<input>`, `<textarea>` or `[contenteditable]` element.

### option: Page.fill.noWaitAfter = %%-input-no-wait-after-%%

### option: Page.fill.timeout = %%-input-timeout-%%

## async method: Page.focus

This method fetches an element with [`param: selector`] and focuses it. If there's no element matching [`param:
selector`], the method waits until a matching element appears in the DOM.

Shortcut for main frame's [`method: Frame.focus`].

### param: Page.focus.selector = %%-input-selector-%%

### option: Page.focus.timeout = %%-input-timeout-%%

## method: Page.frame
- returns: <[null]|[Frame]>

Returns frame matching the specified criteria. Either `name` or `url` must be specified.

```js
const frame = page.frame('frame-name');
```

```js
const frame = page.frame({ url: /.*domain.*/ });
```

### param: Page.frame.frameSelector
- `frameSelector` <[string]|[Object]>
  - `name` <[string]> Frame name specified in the `iframe`'s `name` attribute. Optional.
  - `url` <[string]|[RegExp]|[function]\([URL]\):[boolean]> A glob pattern, regex pattern or predicate receiving frame's `url` as a [URL] object. Optional.

Frame name or other frame lookup options.

## method: Page.frames
- returns: <[Array]<[Frame]>>

An array of all frames attached to the page.

## async method: Page.getAttribute
- returns: <[null]|[string]>

Returns element attribute value.

### param: Page.getAttribute.selector = %%-input-selector-%%

### param: Page.getAttribute.name
- `name` <[string]>

Attribute name to get the value for.

### option: Page.getAttribute.timeout = %%-input-timeout-%%

## async method: Page.goBack
- returns: <[null]|[Response]>

Returns the main resource response. In case of multiple redirects, the navigation will resolve with the response of the
last redirect. If can not go back, returns `null`.

Navigate to the previous page in history.

### option: Page.goBack.timeout = %%-navigation-timeout-%%

### option: Page.goBack.waitUntil = %%-navigation-wait-until-%%

## async method: Page.goForward
- returns: <[null]|[Response]>

Returns the main resource response. In case of multiple redirects, the navigation will resolve with the response of the
last redirect. If can not go forward, returns `null`.

Navigate to the next page in history.

### option: Page.goForward.timeout = %%-navigation-timeout-%%

### option: Page.goForward.waitUntil = %%-navigation-wait-until-%%

## async method: Page.goto
- returns: <[null]|[Response]>

Returns the main resource response. In case of multiple redirects, the navigation will resolve with the response of the
last redirect.

`page.goto` will throw an error if:
* there's an SSL error (e.g. in case of self-signed certificates).
* target URL is invalid.
* the [`option: timeout`] is exceeded during navigation.
* the remote server does not respond or is unreachable.
* the main resource failed to load.

`page.goto` will not throw an error when any valid HTTP status code is returned by the remote server, including 404 "Not
Found" and 500 "Internal Server Error".  The status code for such responses can be retrieved by calling [`method:
Response.status`].

> **NOTE** `page.goto` either throws an error or returns a main resource response. The only exceptions are navigation to
`about:blank` or navigation to the same URL with a different hash, which would succeed and return `null`.
> **NOTE** Headless mode doesn't support navigation to a PDF document. See the [upstream
issue](https://bugs.chromium.org/p/chromium/issues/detail?id=761295).

Shortcut for main frame's [`method: Frame.goto`]

### param: Page.goto.url
- `url` <[string]>

URL to navigate page to. The url should include scheme, e.g. `https://`.

### option: Page.goto.timeout = %%-navigation-timeout-%%

### option: Page.goto.waitUntil = %%-navigation-wait-until-%%

### option: Page.goto.referer
- `referer` <[string]>

Referer header value. If provided it will take preference over the referer header value set by [`method:
Page.setExtraHTTPHeaders`].

## async method: Page.hover

This method hovers over an element matching [`param: selector`] by performing the following steps:
1. Find an element match matching [`param: selector`]. If there is none, wait until a matching element is attached to the DOM.
1. Wait for [actionability](./actionability.md) checks on the matched element, unless [`option: force`] option is set. If the element is detached during the checks, the whole action is retried.
1. Scroll the element into view if needed.
1. Use [`property: Page.mouse`] to hover over the center of the element, or the specified [`option: position`].
1. Wait for initiated navigations to either succeed or fail, unless `noWaitAfter` option is set.

When all steps combined have not finished during the specified [`option: timeout`], this method rejects with a
[TimeoutError]. Passing zero timeout disables this.

Shortcut for main frame's [`method: Frame.hover`].

### param: Page.hover.selector = %%-input-selector-%%

### option: Page.hover.position = %%-input-position-%%

### option: Page.hover.modifiers = %%-input-modifiers-%%

### option: Page.hover.force = %%-input-force-%%

### option: Page.hover.timeout = %%-input-timeout-%%

## async method: Page.innerHTML
- returns: <[string]>

Returns `element.innerHTML`.

### param: Page.innerHTML.selector = %%-input-selector-%%

### option: Page.innerHTML.timeout = %%-input-timeout-%%

## async method: Page.innerText
- returns: <[string]>

Returns `element.innerText`.

### param: Page.innerText.selector = %%-input-selector-%%

### option: Page.innerText.timeout = %%-input-timeout-%%

## method: Page.isClosed
- returns: <[boolean]>

Indicates that the page has been closed.

## async method: Page.isDisabled
- returns: <[boolean]>

Returns whether the element is disabled, the opposite of [enabled](./actionability.md#enabled).

### param: Page.isDisabled.selector = %%-input-selector-%%

### option: Page.isDisabled.timeout = %%-input-timeout-%%

## async method: Page.isEditable
- returns: <[boolean]>

Returns whether the element is [editable](./actionability.md#editable).

### param: Page.isEditable.selector = %%-input-selector-%%

### option: Page.isEditable.timeout = %%-input-timeout-%%

## async method: Page.isEnabled
- returns: <[boolean]>

Returns whether the element is [enabled](./actionability.md#enabled).

### param: Page.isEnabled.selector = %%-input-selector-%%

### option: Page.isEnabled.timeout = %%-input-timeout-%%

## async method: Page.isHidden
- returns: <[boolean]>

Returns whether the element is hidden, the opposite of [visible](./actionability.md#visible).

### param: Page.isHidden.selector = %%-input-selector-%%

### option: Page.isHidden.timeout = %%-input-timeout-%%

## async method: Page.isVisible
- returns: <[boolean]>

Returns whether the element is [visible](./actionability.md#visible).

### param: Page.isVisible.selector = %%-input-selector-%%

### option: Page.isVisible.timeout = %%-input-timeout-%%

## property: Page.keyboard
- type: <[Keyboard]>

## method: Page.mainFrame
- returns: <[Frame]>

The page's main frame. Page is guaranteed to have a main frame which persists during navigations.

## property: Page.mouse
- type: <[Mouse]>

## async method: Page.opener
- returns: <[null]|[Page]>

Returns the opener for popup pages and `null` for others. If the opener has been closed already the returns `null`.

## async method: Page.pdf
- returns: <[Buffer]>

Returns the PDF buffer.

> **NOTE** Generating a pdf is currently only supported in Chromium headless.

`page.pdf()` generates a pdf of the page with `print` css media. To generate a pdf with `screen` media, call [`method:
Page.emulateMedia`] before calling `page.pdf()`:

> **NOTE** By default, `page.pdf()` generates a pdf with modified colors for printing. Use the
[`-webkit-print-color-adjust`](https://developer.mozilla.org/en-US/docs/Web/CSS/-webkit-print-color-adjust) property to
force rendering of exact colors.

```js
// Generates a PDF with 'screen' media type.
await page.emulateMedia({media: 'screen'});
await page.pdf({path: 'page.pdf'});
```

The [`option: width`], [`option: height`], and [`option: margin`] options accept values labeled with units.
Unlabeled values are treated as pixels.

A few examples:
* `page.pdf({width: 100})` - prints with width set to 100 pixels
* `page.pdf({width: '100px'})` - prints with width set to 100 pixels
* `page.pdf({width: '10cm'})` - prints with width set to 10 centimeters.

All possible units are:
* `px` - pixel
* `in` - inch
* `cm` - centimeter
* `mm` - millimeter

The [`option: format`] options are:
* `Letter`: 8.5in x 11in
* `Legal`: 8.5in x 14in
* `Tabloid`: 11in x 17in
* `Ledger`: 17in x 11in
* `A0`: 33.1in x 46.8in
* `A1`: 23.4in x 33.1in
* `A2`: 16.54in x 23.4in
* `A3`: 11.7in x 16.54in
* `A4`: 8.27in x 11.7in
* `A5`: 5.83in x 8.27in
* `A6`: 4.13in x 5.83in

> **NOTE** [`option: headerTemplate`] and [`option: footerTemplate`] markup have the following limitations:
> 1. Script tags inside templates are not evaluated.
> 2. Page styles are not visible inside templates.

### option: Page.pdf.path
- `path` <[path]>

The file path to save the PDF to. If [`option: path`] is a relative path, then it is resolved relative to the current
working directory. If no path is provided, the PDF won't be saved to the disk.

### option: Page.pdf.scale
- `scale` <[float]>

Scale of the webpage rendering. Defaults to `1`. Scale amount must be between 0.1 and 2.

### option: Page.pdf.displayHeaderFooter
- `displayHeaderFooter` <[boolean]>

Display header and footer. Defaults to `false`.

### option: Page.pdf.headerTemplate
- `headerTemplate` <[string]>

HTML template for the print header. Should be valid HTML markup with following classes used to inject printing values
into them:
  * `'date'` formatted print date
  * `'title'` document title
  * `'url'` document location
  * `'pageNumber'` current page number
  * `'totalPages'` total pages in the document

### option: Page.pdf.footerTemplate
- `footerTemplate` <[string]>

HTML template for the print footer. Should use the same format as the [`option: headerTemplate`].

### option: Page.pdf.printBackground
- `printBackground` <[boolean]>

Print background graphics. Defaults to `false`.

### option: Page.pdf.landscape
- `landscape` <[boolean]>

Paper orientation. Defaults to `false`.

### option: Page.pdf.pageRanges
- `pageRanges` <[string]>

Paper ranges to print, e.g., '1-5, 8, 11-13'. Defaults to the empty string, which means print all pages.

### option: Page.pdf.format
- `format` <[string]>

Paper format. If set, takes priority over [`option: width`] or [`option: height`] options. Defaults to 'Letter'.

### option: Page.pdf.width
- `width` <[string]|[float]>

Paper width, accepts values labeled with units.

### option: Page.pdf.height
- `height` <[string]|[float]>

Paper height, accepts values labeled with units.

### option: Page.pdf.margin
- `margin` <[Object]>
  - `top` <[string]|[float]> Top margin, accepts values labeled with units. Defaults to `0`.
  - `right` <[string]|[float]> Right margin, accepts values labeled with units. Defaults to `0`.
  - `bottom` <[string]|[float]> Bottom margin, accepts values labeled with units. Defaults to `0`.
  - `left` <[string]|[float]> Left margin, accepts values labeled with units. Defaults to `0`.

Paper margins, defaults to none.

### option: Page.pdf.preferCSSPageSize
- `preferCSSPageSize` <[boolean]>

Give any CSS `@page` size declared in the page priority over what is declared in [`option: width`] and [`option:
height`] or [`option: format`] options. Defaults to `false`, which will scale the content to fit the paper size.

## async method: Page.press

Focuses the element, and then uses [`method: Keyboard.down`] and [`method: Keyboard.up`].

[`param: key`] can specify the intended
[keyboardEvent.key](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/key) value or a single character to
generate the text for. A superset of the [`param: key`] values can be found
[here](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/key/Key_Values). Examples of the keys are:

`F1` - `F12`, `Digit0`- `Digit9`, `KeyA`- `KeyZ`, `Backquote`, `Minus`, `Equal`, `Backslash`, `Backspace`, `Tab`,
`Delete`, `Escape`, `ArrowDown`, `End`, `Enter`, `Home`, `Insert`, `PageDown`, `PageUp`, `ArrowRight`, `ArrowUp`, etc.

Following modification shortcuts are also supported: `Shift`, `Control`, `Alt`, `Meta`, `ShiftLeft`.

Holding down `Shift` will type the text that corresponds to the [`param: key`] in the upper case.

If [`param: key`] is a single character, it is case-sensitive, so the values `a` and `A` will generate different
respective texts.

Shortcuts such as `key: "Control+o"` or `key: "Control+Shift+T"` are supported as well. When speficied with the
modifier, modifier is pressed and being held while the subsequent key is being pressed.

```js
const page = await browser.newPage();
await page.goto('https://keycode.info');
await page.press('body', 'A');
await page.screenshot({ path: 'A.png' });
await page.press('body', 'ArrowLeft');
await page.screenshot({ path: 'ArrowLeft.png' });
await page.press('body', 'Shift+O');
await page.screenshot({ path: 'O.png' });
await browser.close();
```

### param: Page.press.selector = %%-input-selector-%%

### param: Page.press.key
- `key` <[string]>

Name of the key to press or a character to generate, such as `ArrowLeft` or `a`.

### option: Page.press.delay
- `delay` <[float]>

Time to wait between `keydown` and `keyup` in milliseconds. Defaults to 0.

### option: Page.press.noWaitAfter = %%-input-no-wait-after-%%

### option: Page.press.timeout = %%-input-timeout-%%

## async method: Page.reload
- returns: <[null]|[Response]>

Returns the main resource response. In case of multiple redirects, the navigation will resolve with the response of the
last redirect.

### option: Page.reload.timeout = %%-navigation-timeout-%%

### option: Page.reload.waitUntil = %%-navigation-wait-until-%%

## async method: Page.route

Routing provides the capability to modify network requests that are made by a page.

Once routing is enabled, every request matching the url pattern will stall unless it's continued, fulfilled or aborted.

> **NOTE** The handler will only be called for the first url if the response is a redirect.

An example of a naïve handler that aborts all image requests:

```js
const page = await browser.newPage();
await page.route('**/*.{png,jpg,jpeg}', route => route.abort());
await page.goto('https://example.com');
await browser.close();
```

or the same snippet using a regex pattern instead:

```js
const page = await browser.newPage();
await page.route(/(\.png$)|(\.jpg$)/, route => route.abort());
await page.goto('https://example.com');
await browser.close();
```

Page routes take precedence over browser context routes (set up with [`method: BrowserContext.route`]) when request
matches both handlers.

> **NOTE** Enabling routing disables http cache.

### param: Page.route.url
- `url` <[string]|[RegExp]|[function]\([URL]\):[boolean]>

A glob pattern, regex pattern or predicate receiving [URL] to match while routing.

### param: Page.route.handler
- `handler` <[function]\([Route], [Request]\)>

handler function to route the request.

## async method: Page.screenshot
- returns: <[Buffer]>

Returns the buffer with the captured screenshot.

> **NOTE** Screenshots take at least 1/6 second on Chromium OS X and Chromium Windows. See https://crbug.com/741689 for
discussion.

### option: Page.screenshot.path
- `path` <[path]>

The file path to save the image to. The screenshot type will be inferred from file extension. If [`option: path`] is a
relative path, then it is resolved relative to the current working directory. If no path is provided, the image won't be
saved to the disk.

### option: Page.screenshot.type
- `type` <"png"|"jpeg">

Specify screenshot type, defaults to `png`.

### option: Page.screenshot.quality
- `quality` <[int]>

The quality of the image, between 0-100. Not applicable to `png` images.

### option: Page.screenshot.fullPage
- `fullPage` <[boolean]>

When true, takes a screenshot of the full scrollable page, instead of the currently visible viewport. Defaults to
`false`.

### option: Page.screenshot.clip
- `clip` <[Object]>
  - `x` <[float]> x-coordinate of top-left corner of clip area
  - `y` <[float]> y-coordinate of top-left corner of clip area
  - `width` <[float]> width of clipping area
  - `height` <[float]> height of clipping area

An object which specifies clipping of the resulting image. Should have the following fields:

### option: Page.screenshot.omitBackground
- `omitBackground` <[boolean]>

Hides default white background and allows capturing screenshots with transparency. Not applicable to `jpeg` images.
Defaults to `false`.

### option: Page.screenshot.timeout = %%-input-timeout-%%

## async method: Page.selectOption
- returns: <[Array]<[string]>>

Returns the array of option values that have been successfully selected.

Triggers a `change` and `input` event once all the provided options have been selected. If there's no `<select>` element
matching [`param: selector`], the method throws an error.

```js
// single selection matching the value
page.selectOption('select#colors', 'blue');

// single selection matching both the value and the label
page.selectOption('select#colors', { label: 'Blue' });

// multiple selection
page.selectOption('select#colors', ['red', 'green', 'blue']);

```

Shortcut for main frame's [`method: Frame.selectOption`]

### param: Page.selectOption.selector = %%-input-selector-%%
### param: Page.selectOption.values = %%-select-options-values-%%
### option: Page.selectOption.noWaitAfter = %%-input-no-wait-after-%%
### option: Page.selectOption.timeout = %%-input-timeout-%%

## async method: Page.setContent

### param: Page.setContent.html
- `html` <[string]>

HTML markup to assign to the page.

### option: Page.setContent.timeout = %%-navigation-timeout-%%

### option: Page.setContent.waitUntil = %%-navigation-wait-until-%%

## method: Page.setDefaultNavigationTimeout

This setting will change the default maximum navigation time for the following methods and related shortcuts:
* [`method: Page.goBack`]
* [`method: Page.goForward`]
* [`method: Page.goto`]
* [`method: Page.reload`]
* [`method: Page.setContent`]
* [`method: Page.waitForNavigation`]

> **NOTE** [`method: Page.setDefaultNavigationTimeout`] takes priority over [`method: Page.setDefaultTimeout`],
[`method: BrowserContext.setDefaultTimeout`] and [`method: BrowserContext.setDefaultNavigationTimeout`].

### param: Page.setDefaultNavigationTimeout.timeout
- `timeout` <[float]>

Maximum navigation time in milliseconds

## method: Page.setDefaultTimeout

This setting will change the default maximum time for all the methods accepting [`param: timeout`] option.

> **NOTE** [`method: Page.setDefaultNavigationTimeout`] takes priority over [`method: Page.setDefaultTimeout`].

### param: Page.setDefaultTimeout.timeout
- `timeout` <[float]>

Maximum time in milliseconds

## async method: Page.setExtraHTTPHeaders

The extra HTTP headers will be sent with every request the page initiates.

> **NOTE** page.setExtraHTTPHeaders does not guarantee the order of headers in the outgoing requests.

### param: Page.setExtraHTTPHeaders.headers
- `headers` <[Object]<[string], [string]>>

An object containing additional HTTP headers to be sent with every request. All header values must be strings.

## async method: Page.setInputFiles

This method expects [`param: selector`] to point to an [input
element](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input).

Sets the value of the file input to these file paths or files. If some of the `filePaths` are relative paths, then they
are resolved relative to the the current working directory. For empty array, clears the selected files.

### param: Page.setInputFiles.selector = %%-input-selector-%%

### param: Page.setInputFiles.files = %%-input-files-%%

### option: Page.setInputFiles.noWaitAfter = %%-input-no-wait-after-%%

### option: Page.setInputFiles.timeout = %%-input-timeout-%%

## async method: Page.setViewportSize

In the case of multiple pages in a single browser, each page can have its own viewport size. However, [`method:
Browser.newContext`] allows to set viewport size (and more) for all pages in the context at once.

`page.setViewportSize` will resize the page. A lot of websites don't expect phones to change size, so you should set the
viewport size before navigating to the page.

```js
const page = await browser.newPage();
await page.setViewportSize({
  width: 640,
  height: 480,
});
await page.goto('https://example.com');
```

### param: Page.setViewportSize.viewportSize
- `viewportSize` <[Object]>
  - `width` <[int]> page width in pixels. **required**
  - `height` <[int]> page height in pixels. **required**

## async method: Page.tap

This method taps an element matching [`param: selector`] by performing the following steps:
1. Find an element match matching [`param: selector`]. If there is none, wait until a matching element is attached to the DOM.
1. Wait for [actionability](./actionability.md) checks on the matched element, unless [`option: force`] option is set. If the element is detached during the checks, the whole action is retried.
1. Scroll the element into view if needed.
1. Use [`property: Page.touchscreen`] to tap the center of the element, or the specified [`option: position`].
1. Wait for initiated navigations to either succeed or fail, unless [`option: noWaitAfter`] option is set.

When all steps combined have not finished during the specified [`option: timeout`], this method rejects with a
[TimeoutError]. Passing zero timeout disables this.

> **NOTE** `page.tap()` requires that the `hasTouch` option of the browser context be set to true.

Shortcut for main frame's [`method: Frame.tap`].

### param: Page.tap.selector = %%-input-selector-%%

### option: Page.tap.position = %%-input-position-%%

### option: Page.tap.modifiers = %%-input-modifiers-%%

### option: Page.tap.noWaitAfter = %%-input-no-wait-after-%%

### option: Page.tap.force = %%-input-force-%%

### option: Page.tap.timeout = %%-input-timeout-%%

## async method: Page.textContent
- returns: <[null]|[string]>

Returns `element.textContent`.

### param: Page.textContent.selector = %%-input-selector-%%

### option: Page.textContent.timeout = %%-input-timeout-%%

## async method: Page.title
- returns: <[string]>

Returns the page's title. Shortcut for main frame's [`method: Frame.title`].

## property: Page.touchscreen
- type: <[Touchscreen]>

## async method: Page.type

Sends a `keydown`, `keypress`/`input`, and `keyup` event for each character in the text. `page.type` can be used to send
fine-grained keyboard events. To fill values in form fields, use [`method: Page.fill`].

To press a special key, like `Control` or `ArrowDown`, use [`method: Keyboard.press`].

```js
await page.type('#mytextarea', 'Hello'); // Types instantly
await page.type('#mytextarea', 'World', {delay: 100}); // Types slower, like a user
```

Shortcut for main frame's [`method: Frame.type`].

### param: Page.type.selector = %%-input-selector-%%

### param: Page.type.text
- `text` <[string]>

A text to type into a focused element.

### option: Page.type.delay
- `delay` <[float]>

Time to wait between key presses in milliseconds. Defaults to 0.

### option: Page.type.noWaitAfter = %%-input-no-wait-after-%%

### option: Page.type.timeout = %%-input-timeout-%%

## async method: Page.uncheck

This method unchecks an element matching [`param: selector`] by performing the following steps:
1. Find an element match matching [`param: selector`]. If there is none, wait until a matching element is attached to the DOM.
1. Ensure that matched element is a checkbox or a radio input. If not, this method rejects. If the element is already unchecked, this method returns immediately.
1. Wait for [actionability](./actionability.md) checks on the matched element, unless [`option: force`] option is set. If the element is detached during the checks, the whole action is retried.
1. Scroll the element into view if needed.
1. Use [`property: Page.mouse`] to click in the center of the element.
1. Wait for initiated navigations to either succeed or fail, unless [`option: noWaitAfter`] option is set.
1. Ensure that the element is now unchecked. If not, this method rejects.

When all steps combined have not finished during the specified [`option: timeout`], this method rejects with a
[TimeoutError]. Passing zero timeout disables this.

Shortcut for main frame's [`method: Frame.uncheck`].

### param: Page.uncheck.selector = %%-input-selector-%%

### option: Page.uncheck.force = %%-input-force-%%

### option: Page.uncheck.noWaitAfter = %%-input-no-wait-after-%%

### option: Page.uncheck.timeout = %%-input-timeout-%%

## async method: Page.unroute

Removes a route created with [`method: Page.route`]. When [`param: handler`] is not specified, removes all routes
for the [`param: url`].

### param: Page.unroute.url
- `url` <[string]|[RegExp]|[function]\([URL]\):[boolean]>

A glob pattern, regex pattern or predicate receiving [URL] to match while routing.

### param: Page.unroute.handler
- `handler` <[function]\([Route], [Request]\)>

Optional handler function to route the request.

## method: Page.url
- returns: <[string]>

Shortcut for main frame's [`method: Frame.url`].

## method: Page.video
- returns: <[null]|[Video]>

Video object associated with this page.

## method: Page.viewportSize
- returns: <[null]|[Object]>
  - `width` <[int]> page width in pixels.
  - `height` <[int]> page height in pixels.

## async method: Page.waitForEvent
- returns: <[any]>

Returns the event data value.

Waits for event to fire and passes its value into the predicate function. Returns when the predicate returns truthy
value. Will throw an error if the page is closed before the event is fired.

### param: Page.waitForEvent.event
- `event` <[string]>

Event name, same one would pass into `page.on(event)`.

### param: Page.waitForEvent.optionsOrPredicate
- `optionsOrPredicate` <[Function]|[Object]>
  - `predicate` <[Function]> receives the event data and resolves to truthy value when the waiting should resolve.
  - `timeout` <[float]> maximum time to wait for in milliseconds. Defaults to `30000` (30 seconds). Pass `0` to disable timeout. The default value can be changed by using the [`method: BrowserContext.setDefaultTimeout`].

Either a predicate that receives an event or an options object. Optional.

## async method: Page.waitForFunction
- returns: <[JSHandle]>

Returns when the [`param: pageFunction`] returns a truthy value. It resolves to a JSHandle of the truthy value.

The `waitForFunction` can be used to observe viewport size change:

```js
const { webkit } = require('playwright');  // Or 'chromium' or 'firefox'.

(async () => {
  const browser = await webkit.launch();
  const page = await browser.newPage();
  const watchDog = page.waitForFunction('window.innerWidth < 100');
  await page.setViewportSize({width: 50, height: 50});
  await watchDog;
  await browser.close();
})();
```

To pass an argument to the predicate of `page.waitForFunction` function:

```js
const selector = '.foo';
await page.waitForFunction(selector => !!document.querySelector(selector), selector);
```

Shortcut for main frame's [`method: Frame.waitForFunction`].

### param: Page.waitForFunction.pageFunction
- `pageFunction` <[function]|[string]>

Function to be evaluated in browser context

### param: Page.waitForFunction.arg
- `arg` <[EvaluationArgument]>

Optional argument to pass to [`param: pageFunction`]

### option: Page.waitForFunction.polling
- `polling` <[float]|"raf">

If [`option: polling`] is `'raf'`, then [`param: pageFunction`] is constantly executed in `requestAnimationFrame`
callback. If [`option: polling`] is a number, then it is treated as an interval in milliseconds at which the function
would be executed. Defaults to `raf`.

### option: Page.waitForFunction.timeout = %%-wait-for-timeout-%%

## async method: Page.waitForLoadState

Returns when the required load state has been reached.

This resolves when the page reaches a required load state, `load` by default. The navigation must have been committed
when this method is called. If current document has already reached the required state, resolves immediately.

```js
await page.click('button'); // Click triggers navigation.
await page.waitForLoadState(); // The promise resolves after 'load' event.
```

```js
const [popup] = await Promise.all([
  page.waitForEvent('popup'),
  page.click('button'), // Click triggers a popup.
])
await popup.waitForLoadState('domcontentloaded'); // The promise resolves after 'domcontentloaded' event.
console.log(await popup.title()); // Popup is ready to use.
```

Shortcut for main frame's [`method: Frame.waitForLoadState`].

### param: Page.waitForLoadState.state
- `state` <"load"|"domcontentloaded"|"networkidle">

Optional load state to wait for, defaults to `load`. If the state has been already reached while loading current document, the
method resolves immediately. Can be one of:
  * `'load'` - wait for the `load` event to be fired.
  * `'domcontentloaded'` - wait for the `DOMContentLoaded` event to be fired.
  * `'networkidle'` - wait until there are no network connections for at least `500` ms.

### option: Page.waitForLoadState.timeout = %%-navigation-timeout-%%

## async method: Page.waitForNavigation
- returns: <[null]|[Response]>

Returns the main resource response. In case of multiple redirects, the navigation will resolve with the response of the
last redirect. In case of navigation to a different anchor or navigation due to History API usage, the navigation will
resolve with `null`.

This resolves when the page navigates to a new URL or reloads. It is useful for when you run code which will indirectly
cause the page to navigate. e.g. The click target has an `onclick` handler that triggers navigation from a `setTimeout`.
Consider this example:

```js
const [response] = await Promise.all([
  page.waitForNavigation(), // The promise resolves after navigation has finished
  page.click('a.delayed-navigation'), // Clicking the link will indirectly cause a navigation
]);
```

**NOTE** Usage of the [History API](https://developer.mozilla.org/en-US/docs/Web/API/History_API) to change the URL is
considered a navigation.

Shortcut for main frame's [`method: Frame.waitForNavigation`].

### option: Page.waitForNavigation.timeout = %%-navigation-timeout-%%

### option: Page.waitForNavigation.url
- `url` <[string]|[RegExp]|[function]\([URL]\):[boolean]>

A glob pattern, regex pattern or predicate receiving [URL] to match while waiting for the navigation.

### option: Page.waitForNavigation.waitUntil = %%-navigation-wait-until-%%

## async method: Page.waitForRequest
- returns: <[Request]>

Waits for the matching request and returns it.

```js
const firstRequest = await page.waitForRequest('http://example.com/resource');
const finalRequest = await page.waitForRequest(request => request.url() === 'http://example.com' && request.method() === 'GET');
return firstRequest.url();
```

```js
await page.waitForRequest(request => request.url().searchParams.get('foo') === 'bar' && request.url().searchParams.get('foo2') === 'bar2');
```

### param: Page.waitForRequest.urlOrPredicate
- `urlOrPredicate` <[string]|[RegExp]|[function]\([Request]\):[boolean]>

Request URL string, regex or predicate receiving [Request] object.

### option: Page.waitForRequest.timeout
- `timeout` <[float]>

Maximum wait time in milliseconds, defaults to 30 seconds, pass `0` to disable the timeout. The default value can be
changed by using the [`method: Page.setDefaultTimeout`] method.

## async method: Page.waitForResponse
- returns: <[Response]>

Returns the matched response.

```js
const firstResponse = await page.waitForResponse('https://example.com/resource');
const finalResponse = await page.waitForResponse(response => response.url() === 'https://example.com' && response.status() === 200);
return finalResponse.ok();
```

### param: Page.waitForResponse.urlOrPredicate
- `urlOrPredicate` <[string]|[RegExp]|[function]\([Response]\):[boolean]>

Request URL string, regex or predicate receiving [Response] object.

### option: Page.waitForResponse.timeout
- `timeout` <[float]>

Maximum wait time in milliseconds, defaults to 30 seconds, pass `0` to disable the timeout. The default value can be
changed by using the [`method: BrowserContext.setDefaultTimeout`] or [`method: Page.setDefaultTimeout`] methods.

## async method: Page.waitForSelector
- returns: <[null]|[ElementHandle]>

Returns when element specified by selector satisfies [`option: state`] option. Returns `null` if waiting for `hidden`
or `detached`.

Wait for the [`param: selector`] to satisfy [`option: state`] option (either appear/disappear from dom, or become
visible/hidden). If at the moment of calling the method [`param: selector`] already satisfies the condition, the
method will return immediately. If the selector doesn't satisfy the condition for the [`option: timeout`]
milliseconds, the function will throw.

This method works across navigations:

```js
const { chromium } = require('playwright');  // Or 'firefox' or 'webkit'.

(async () => {
  const browser = await chromium.launch();
  const page = await browser.newPage();
  let currentURL;
  page
    .waitForSelector('img')
    .then(() => console.log('First URL with image: ' + currentURL));
  for (currentURL of ['https://example.com', 'https://google.com', 'https://bbc.com']) {
    await page.goto(currentURL);
  }
  await browser.close();
})();
```

### param: Page.waitForSelector.selector = %%-query-selector-%%

### option: Page.waitForSelector.state = %%-wait-for-selector-state-%%

### option: Page.waitForSelector.timeout = %%-input-timeout-%%

## async method: Page.waitForTimeout

Waits for the given [`param: timeout`] in milliseconds.

Note that `page.waitForTimeout()` should only be used for debugging. Tests using the timer in production are going to be
flaky. Use signals such as network events, selectors becoming visible and others instead.

```js
// wait for 1 second
await page.waitForTimeout(1000);
```

Shortcut for main frame's [`method: Frame.waitForTimeout`].

### param: Page.waitForTimeout.timeout
- `timeout` <[float]>

A timeout to wait for

## method: Page.workers
- returns: <[Array]<[Worker]>>

This method returns all of the dedicated [WebWorkers](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API)
associated with the page.

> **NOTE** This does not contain ServiceWorkers