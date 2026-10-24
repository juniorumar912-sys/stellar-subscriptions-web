# Stellar Subscriptions — Web

The web app for Stellar Subscriptions: recurring payments on Stellar. Merchants publish plans and charge on schedule. Subscribers authorize a capped subscription, and can pause or cancel it at any time.

The app never promises more than the contracts enforce:

- A merchant can never charge more than the cap the subscriber chose.
- A merchant can never charge more often than once per billing interval.
- Cancelling is final. After it, no further charge can go through.

If a screen says otherwise, that is a bug.

## Screenshots

_Screenshots are not captured yet. Add them under `docs/screenshots/` and link them here._

## How it works

The app has **no backend**. It is a Next.js front end that talks to three Soroban smart contracts directly through a Stellar RPC node:

| Contract     | Role                                                    |
| ------------ | ------------------------------------------------------- |
| Subscription | Holds each subscriber's authorization, cap and status. Enforces the cap and the schedule. |
| Plan         | Stores the merchant plans (amount, interval, merchant). |
| Registry     | Aggregate platform stats shown on the home page.        |

- **Reads** are simulated read-only calls through the RPC node. Nothing is signed and nothing is spent. A funded account is used only as the simulation source.
- **Writes** (subscribe, pause, resume, cancel) are built in the browser, simulated, signed by the subscriber's [Freighter](https://www.freighter.app/) wallet, submitted to the RPC node, and then confirmed on-ledger.

The contract code, its tests and its deployment records live in the contracts repository: [stellar-sub/stellar-subscriptions-contracts](https://github.com/stellar-sub/stellar-subscriptions-contracts). The HTTP API and documentation live in [stellar-sub/stellar-subscriptions-api-docs](https://github.com/stellar-sub/stellar-subscriptions-api-docs).

### Testnet deployment

The app uses this Testnet deployment by default. The ids are the ones recorded in the contracts repository's `DEPLOYMENTS.md`:

| Contract     | Testnet contract id                                        |
| ------------ | ---------------------------------------------------------- |
| Subscription | `CAW5H4BH5VXWR23DFUVMY45TL7JSMTDZTH3E7LMMXW5LZ7FMDL33KXMM` |
| Plan         | `CDKFRKPA2EIYMW2GX2FY4ZFQ2VAFS5UZD5BHAZLCIGRA7N24N2DFAYN3` |
| Registry     | `CB7HBXFNBLUSBY3SMIYUPVTE3JZTISYRMZB7XXPUC3ZPLIDNPTKUX645` |

Mainnet is **not deployed yet**. With no Mainnet ids configured, the app shows a "not deployed on Mainnet" state rather than failing.

## Quick start

Requirements:

- Node.js 20 or newer
- npm
- The Freighter browser extension, set to Testnet, if you want to subscribe or charge from the UI

```sh
git clone https://github.com/stellar-sub/stellar-subscriptions-web.git
cd stellar-subscriptions-web
npm install
cp .env.example .env.local
npm run dev
```

Open <http://localhost:3000>. The app opens on Testnet. Connect Freighter to see your own subscriptions.

To try the subscribe flow you need a funded Testnet account. Fund one with [Friendbot](https://laboratory.stellar.org/#account-creator?network=test) and switch Freighter to Testnet.

## Environment variables

Copy `.env.example` to `.env.local`. Every variable is read at build time and is exposed to the browser, so do not put secrets in them.

| Variable                                       | Required | Purpose                                                                 |
| ---------------------------------------------- | -------- | ----------------------------------------------------------------------- |
| `NEXT_PUBLIC_DEFAULT_NETWORK`                  | No       | `testnet` (default) or `mainnet`. The network shown before a wallet connects. |
| `NEXT_PUBLIC_TESTNET_RPC_URL`                  | No       | Testnet Stellar RPC URL. Defaults to `https://soroban-testnet.stellar.org`. |
| `NEXT_PUBLIC_MAINNET_RPC_URL`                  | No       | Mainnet Stellar RPC URL. Defaults to `https://mainnet.sorobanrpc.com`. |
| `NEXT_PUBLIC_TESTNET_SUBSCRIPTION_CONTRACT`    | No       | Testnet Subscription contract id. Defaults to the deployment above.      |
| `NEXT_PUBLIC_TESTNET_PLAN_CONTRACT`            | No       | Testnet Plan contract id. Defaults to the deployment above.              |
| `NEXT_PUBLIC_TESTNET_REGISTRY_CONTRACT`        | No       | Testnet Registry contract id. Defaults to the deployment above.          |
| `NEXT_PUBLIC_MAINNET_SUBSCRIPTION_CONTRACT`    | No       | Mainnet Subscription contract id. Leave empty until Mainnet is deployed. |
| `NEXT_PUBLIC_MAINNET_PLAN_CONTRACT`            | No       | Mainnet Plan contract id. Leave empty until Mainnet is deployed.         |
| `NEXT_PUBLIC_MAINNET_REGISTRY_CONTRACT`        | No       | Mainnet Registry contract id. Leave empty until Mainnet is deployed.     |
| `NEXT_PUBLIC_TESTNET_READ_ACCOUNT`             | No       | Funded Testnet account used as the source of read-only simulations. Defaults to a built-in account. |
| `NEXT_PUBLIC_MAINNET_READ_ACCOUNT`             | No       | Funded Mainnet account used as the source of read-only simulations. Required to read from Mainnet. |
| `NEXT_PUBLIC_GITHUB_ORG`                       | No       | GitHub organization linked from the footer. Defaults to `stellar-sub`.   |

Mainnet reads need both `NEXT_PUBLIC_MAINNET_READ_ACCOUNT` and the three Mainnet contract ids.

## Scripts

| Command             | What it does                                  |
| ------------------- | --------------------------------------------- |
| `npm run dev`       | Start the development server on port 3000.    |
| `npm run build`     | Create a production build.                    |
| `npm run start`     | Serve the production build.                   |
| `npm run typecheck` | Type-check with `tsc --noEmit`.               |
| `npm run lint`      | Lint with ESLint (Next.js config).            |
| `npm test`          | Run the Vitest suite once.                    |
| `npm run test:watch`| Run Vitest in watch mode.                     |

## Contributing

See [CONTRIBUTING.md](./CONTRIBUTING.md) for how to report issues and how to open a pull request.

Before opening a pull request, run `npm run typecheck`, `npm run lint`, `npm test` and `npm run build`.
