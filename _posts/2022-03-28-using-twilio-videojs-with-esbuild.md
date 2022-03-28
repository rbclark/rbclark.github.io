---
title: Using twilio-video.js with esbuild
---

ESBuild is an extremely fast bundler for javascript. Unfortunately speed doesn't matter if your app doesn't work! While migrating from Webpacker to esbuild with a Rails 7 application I ran into an issue where esbuild would compile successfully, however the twilio-video.js library (and accompanying twilio-ruby gem) were not able to startup. The error was as follows:

```
Uncaught ReferenceError: process is not defined
...
```
This is happening due to a call to `require('twilio-video')` which expects `process.env` to be available globally.  There is a good writeup on the cause of this in an [issue on twilio-video](https://github.com/twilio/twilio-video.js/issues/1746). 

The trick to working around this issue with esbuild is to leverage the [@esbuild-plugins/node-modules-polyfill](https://www.npmjs.com/package/@esbuild-plugins/node-modules-polyfill) package. It provides a polyfill for built-in modules for the browser meaning things that would not normally be available inside a browser will still work. Below is how I included this using an esbuild configuration file.

**esbuild.config.mjs:**
```
import NodeModulesPolyfills from '@esbuild-plugins/node-modules-polyfill'
import esbuild from 'esbuild'

esbuild.build({
  entryPoints: ['app/javascript/application.js'],
  outdir: 'app/assets/builds',
  plugins: [NodeModulesPolyfills.default()]
}).catch(() => process.exit(1))
```

From there, I was able to execute esbuild using the command `node esbuild.config.mjs` and `twilio-video.js` was once again working successfully..
