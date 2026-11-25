# Split

**Split** is a simple open-source application for splitting common expenses. Every split lives at its own unique URL, so sharing it is as easy as sending a link. All data is encrypted in the browser before it leaves your device.

**Try it:** https://split-one.mayfleet.com

> This is the first version of Split, built on Vue 2. The current one is [Split](https://github.com/andrew-markin/split), available at [split.mayfleet.com](https://split.mayfleet.com).

## Features

- **Participants with patrons.** A participant can pay for another one, e.g. for a family member, and can have a payment comment, e.g. a bank account or a payment service to send money to.
- **Categories with participation rates.** Expenses are split per category, and every participant takes a share in a category in quarters of a full one, so everyone pays only for what they share.
- **Expenses.** An amount, a payer and an optional date, description and category. The payer of the previous expense is picked for the next one.
- **Settlements.** Who transfers how much to whom is calculated to the cent, with the largest debts paired with the largest credits to keep the transfers few, and can be copied as text, e.g. to send it to a chat.
- **Sharing by link.** Anyone with the link can view and edit the split in real time.
- **End-to-end encryption.** The server only ever sees encrypted data and never sees the key.
- **Offline-friendly.** Changes are kept in the browser and synced with the server once the connection is back. A dot next to the title marks changes that aren't on the server yet.
- **Clone a split** to start a new one from its copy, or **start a split from scratch**.
- Light and dark themes, English and Russian interface.

## How it works

1. When you open the app, a random 256-bit key is generated and placed in the URL fragment (`https://split-one.mayfleet.com/#/<key>`). Browsers never send the URL fragment to the server.
2. The split is serialized to JSON, compressed with deflate and encrypted with AES (via [crypto-js](https://github.com/brix/crypto-js)) using that key.
3. The server stores and relays the encrypted data under a *reference*: an HMAC-SHA256 of the key. This lets the server identify a split without learning the key.
4. Concurrent edits from multiple clients are merged on the client side. Each participant, category, participation, expense and preference carries a change timestamp, removals are remembered and win over edits, and the server keeps a version number for optimistic concurrency.

The sync backend is [Blobs](https://github.com/andrew-markin/blobs) (formerly Appository Sync), a small self-hosted Socket.IO server that stores opaque versioned blobs.

> **Note:** whoever has the link has full access to the split. Treat the link as a password.

## Tech stack

Vue 2.7, Vuetify 2, Vuex 3, Vue Router 3, vue-i18n 8, Socket.IO client, built with Vue CLI 5 (webpack 5).

### Dependencies and security

Vue 2, Vuetify 2 and Vue CLI are end-of-life, so they and their ecosystems get no major updates. [Dependabot](.github/dependabot.yml) proposes minor and patch updates only, and `overrides` in [`package.json`](./package.json) replace a few dependencies of the toolchain, all used at build time only, with patched versions:

| Override                                          | Instead of | Reason                                     |
| ------------------------------------------------- | ---------- | ------------------------------------------ |
| `webpack-dev-server` 5                            | 4          | Source code exposure on a malicious website |
| `serialize-javascript` 7.1                        | 6          | Cross-site scripting and code execution    |
| `postcss` 8 under `@vue/component-compiler-utils` | 7          | Reading local files through source maps    |
| `uuid` 11 under `sockjs`                          | 8          | Missing buffer bounds check                |
| `cross-spawn` 6 under `yorkie`                    | 5          | Regular expression denial of service       |

`npm run audit` still reports advisories in Vue 2 and Vuetify 2 themselves, which these versions will never get fixes for. None of them is reachable in Split:

- The regular expression denial of service in Vue is in its runtime template compiler, which isn't included in the bundle, as templates are compiled at build time.
- The Vuetify prototype pollution needs a malicious Vuetify preset, while the app configures Vuetify with a static object.
- The Vuetify cross-site scripting is in the title of the date picker, which the app shows without a title.

npm blocks install scripts of dependencies unless they are allowed in `allowScripts` in `package.json`. The only allowed one belongs to `@parcel/watcher` (a dependency of `sass`) and builds it from source only when asked to.

## Development

### Requirements

- Node.js 20.19 or newer
- A running [Blobs](https://github.com/andrew-markin/blobs) server (by default at `ws://localhost:3000`)

### Setup

```bash
npm install
```

Create `.env.development.local` and/or `.env.production.local` in the project root, using [`.env.example`](./.env.example) as a reference:

| Variable           | Description                                              |
| ------------------ | -------------------------------------------------------- |
| `PORT`             | Port of the development server                           |
| `VUE_APP_BACKEND`  | WebSocket URL of the Blobs server                        |
| `VUE_APP_TOKEN`    | Bucket token (JWT) issued by the Blobs server, see below |
| `VUE_APP_REF_SALT` | Salt for deriving split references from keys             |

To get a token, run `npm run token split` in a Blobs checkout with the same `TOKEN_SECRET` as the server. The token gives access to the `split` bucket only.

`VUE_APP_*` variables are embedded into the client bundle, so they are **not secrets**. They identify the application to the sync server. Data stays private because of the per-split encryption key, not because of these values.

> **Warning:** changing `VUE_APP_REF_SALT` changes every split reference, so all existing splits will appear empty.

### Scripts

| Command                    | Description                                    |
| -------------------------- | ---------------------------------------------- |
| `npm start`                | Start the dev server with hot reload           |
| `npm run build`            | Build for production into `dist/`              |
| `npm run lint`             | Lint and auto-fix the sources                  |
| `npm run audit`            | Audit runtime dependencies for vulnerabilities |
| `npm run vue-i18n-extract` | Report missing and unused translation keys     |

### Translations

Locale files live in [`src/locales`](./src/locales). To add a language, copy `en.json` to `<locale>.json`, translate the values and set `_LOCALE_TITLE` to the language's native name. It will show up in the menu automatically.

## Deployment

Every push and pull request is linted and built by the [CI workflow](.github/workflows/ci.yml), and pushes to `main` are deployed to GitHub Pages. The build takes the `VUE_APP_*` values from the repository secrets with the same names.

## License

Split is released under the [GNU General Public License v3.0](./LICENSE).
