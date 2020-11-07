### Dev-Mode
Needed:
  - NodeJS >= 7.6
  - running Instance started as `node g e k k o --ui`

*Side-Note: I'm running on Windows 10 - so here it's working. Linux sometimes gave me headaches while installing dependencies...*

To install, follow these steps:

1. Install Quasar-CLI via npm `npm install -g @quasar/cli`, to make sure, everything is alright, follow the official installation guide to Quasar https://quasar.dev/quasar-cli/installation .
2. Clone this repo to some location on your hardline (should work from any directory in DEV-Mode).
3. Fire up your g e k k o in UI-Mode.
4. CD into cloned repository and run `npm install`.
5. Start the UI with `quasar dev`

*Note: If it complains about missing modules, make sure, vue-cli and quasar-cli are properly installed.*

### Compile for replacing the original g e k k o-UI
1. Clone this repo.
2. Run `npm install` + ```quasar build``` from this repo.
2. In your g e k k o-folder zip up the folder `web/vue` as backup.
3. Place everything from repo's `dist/spa-mat` into the `web/vue` folder. (index.html must be there)
4. Modify the first line in `web/routes/baseConfig.js` so that it looks like this `var UIconfig = require('../vue/statics/UiConfig');`
5. Modify the first line in `web/server.js` so that it looks like this `const config = require('./vue/statics/UiConfig');`
6. Modify ~line 87 in web -> server.js:
replace
```
app
  .use(cors())
  .use(serve(WEBROOT + 'vue/dist'))
  .use(bodyParser())
  .use(require('koa-logger')())
  .use(router.routes())
  .use(router.allowedMethods());
```
with
```
app
  .use(cors())
  .use(serve(WEBROOT + 'vue'))
  .use(bodyParser())
  .use(require('koa-logger')())
  .use(router.routes())
  .use(router.allowedMethods());
```
7. Start g e k k o with UI command (`node g e k k o --ui`).
8. Enjoy!

***If you changed your default connection or database-settings, please edit the file ***`<g e k k o-quasar-ui-folder>/src/statics/UiConfig.js`*** accordingly.***


npx quasar dev
