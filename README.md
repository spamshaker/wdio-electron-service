# WDIO Electron Service

## WebdriverIO service for testing Electron applications

Handles electron-specific browser capabilities, chromedriver execution and session management.

## Installation

```bash
npm i -D wdio-electron-service
```

```bash
yarn i -D wdio-electron-service
```

```bash
pnpm i -D wdio-electron-service
```

`wdio-electron-service` installs chromedriver via [`electron-chromedriver`](https://github.com/electron/chromedriver), so you shouldn't need to install your own version.

Instructions on how to install `WebdriverIO` can be found [here.](https://webdriver.io/docs/gettingstarted)

## Example Configuration

To use the service you need to add `electron` to your services array, followed by a configuration object:

```js
// wdio.conf.js
const { join } = require('path');
const fs = require('fs');

const packageJson = JSON.parse(fs.readFileSync('./package.json'));
const {
  build: { productName },
} = packageJson;

const config = {
  outputDir: 'all-logs',
  // ...
  services: [
    [
      'electron',
      {
        appPath: join(__dirname, 'dist'),
        appName: productName,
        appArgs: ['foo', 'bar=baz'],
        chromedriver: {
          port: 9519,
          logFileName: 'wdio-chromedriver.log',
        },
      },
    ],
  ],
  // ...
};

module.exports = { config };
```

### API Configuration

If you wish to use the Electron APIs then you will need to import the preload and main scripts. At the top of your preload:

```ts
import 'wdio-electron-service/preload';
```

And at the top of your main index file (app entry point):

```ts
import 'wdio-electron-service/main';
```

The APIs should now be available in tests. Currently available APIs: [`app`](https://www.electronjs.org/docs/latest/api/app), [`mainProcess`](https://www.electronjs.org/docs/latest/api/process), [`browserWindow`](https://www.electronjs.org/docs/latest/api/browser-window).

```ts
const appName = await browser.electronApp('getName');
```

### Custom Electron API

You can also implement a custom API if you wish. To do this you will need to define a handler in your main process:

```ts
import { ipcMain } from 'electron';

ipcMain.handle('wdio-electron', () => {
  // access some Electron or Node things on the main process
  return 'such api';
});
```

The custom API can then be called in a spec file:

```ts
const someValue = await browser.electronAPI('wow'); // default
const someValue = await browser.myCustomAPI('wow'); // configured using `customApiBrowserCommand`
```

### Example

See [wdio-electron-service-example](https://github.com/goosewobbler/wdio-electron-service-example) for an example of "real-world" usage in testing a minimal electron app.

## Configuration

### `appPath`: _`string`_

The path to the built app for testing. In a typical electron project this will be where `electron-builder` is configured to output, e.g. `dist` by default. Required to be used with `appName` as both are needed in order to generate a path to the Electron binary.

### `appName`: _`string`_

The name of the built app for testing. Required to be used with `appPath` as both are needed in order to generate a path to the Electron binary.

It needs to match the name of the install directory used by `electron-builder`; this value is derived from your `electron-builder` configuration and will be either the `name` property (from `package.json`) or the `productName` property (from `electron-builder` config). You can find more information regarding this in the `electron-builder` [documentation](https://www.electron.build/configuration/configuration#configuration).

### `binaryPath`: _`string`_

The path to the electron binary of the app for testing. The path generated by using `appPath` and `appName` is tied to `electron-builder` output, if you are implementing something custom then you can use this.

### `appArgs`: _`string[]`_

An array of string arguments to be passed through to the app on execution of the test run.

### `customApiBrowserCommand`: _`string`_

#### default `electronAPI`

The browser command used to access the custom Electron API.

### `newSessionPerTest`: _`boolean`_

#### default `true`

By default the browser session is reloaded after each test to avoid state bleed. If you wish to manage the state of your app across tests manually then you can set this to false.

## Chromedriver configuration

This service wraps the [`wdio-chromedriver-service`](https://github.com/webdriverio-community/wdio-chromedriver-service), you can configure the following options which will be passed through to that service:

### `chromedriver.port`: _`number`_

#### default `9515`

`wdio-chromedriver-service` option. The port on which chromedriver should run.

### `chromedriver.path`: _`string`_

#### default `/`

`wdio-chromedriver-service` option. The path on which chromedriver should run.

### `chromedriver.protocol`: _`string`_

#### default `http`

`wdio-chromedriver-service` option. The protocol chromedriver should use.

### `chromedriver.hostname`: _`string`_

#### default `localhost`

`wdio-chromedriver-service` option. The hostname chromedriver should use.

### `chromedriver.outputDir`: _`string`_

#### default defined by [`config.outputDir`](https://webdriver.io/docs/options/#outputdir)

`wdio-chromedriver-service` option. The path where the output log of the chromedriver server should be stored. If not specified, the WDIO `outputDir` config property is used and chromedriver logs are written to the same directory as the WDIO logs.

### `chromedriver.logFileName`: _`string`_

#### default `wdio-chromedriver.log`

`wdio-chromedriver-service` option. The name of the output log file to be written in the `outputDir`.
