# MedChain — Blockchain-Based Medical Records

A student/demo full-stack project using **Solidity + Hardhat + React + Node.js + IPFS (Pinata)**. It demonstrates patient-controlled permissions, encrypted record uploads, blockchain references, and audit events.

> **Safety:** This is an educational prototype, not a certified healthcare system. Use fictional data only. Do not upload real patient records or use this for clinical decisions. A public blockchain is not an appropriate place for personal medical information. This project stores encrypted payloads on IPFS and only a CID/hash and timestamps on-chain, but CIDs and wallet addresses may still reveal metadata.

## Features

- MetaMask wallet connection.
- Patient wallet registration.
- Patient grants/revokes a provider wallet's smart-contract access.
- Browser-side AES-GCM encryption with a password-derived key (PBKDF2-SHA256).
- Upload encrypted JSON payload to IPFS via Pinata's API.
- Store IPFS CID and integrity hash in a Solidity smart contract.
- Load record references and decrypt in the browser when the correct password is supplied.
- On-chain events for registration, consent, record addition, and record-view requests.
- Responsive dashboard UI.

## Requirements

- Node.js 20+ (includes `fetch`)
- npm
- VS Code
- MetaMask browser extension
- Pinata account/API JWT for actual IPFS uploads (optional until you add a record)

## 1. Install dependencies

Open this project folder in VS Code. In its terminal run:

```bash
npm run install:all
```

## 2. Configure IPFS (Pinata)

1. Create a Pinata account and generate a JWT with pinning permission.
2. Copy `.env.example` to `.env` in the project root.
3. Put the token in `.env`:

```env
PINATA_JWT=your_actual_pinata_jwt
PORT=5000
```

Never commit `.env` or expose your JWT in frontend code. The server only accepts encrypted payloads.

## 3. Start a local blockchain

Open **Terminal 1** in the root folder:

```bash
npx hardhat node
```

Keep it running. Hardhat prints test accounts and private keys. They are local test-only accounts; never use them on a real network.

## 4. Deploy the smart contract

Open **Terminal 2** in the root folder:

```bash
npm run compile
npm run deploy:local
```

Deployment writes the ABI/address into `client/src/contractConfig.json`.

## 5. Configure MetaMask for Hardhat

In MetaMask, add a network manually:

- Network name: Hardhat Local
- RPC URL: `http://127.0.0.1:8545`
- Chain ID: `31337`
- Currency symbol: `ETH`

Import one of the test accounts printed by `npx hardhat node` using its private key. These keys are public test keys and must never hold real assets. Reload the web page after deployment.

## 6. Run the website and API

Open **Terminal 3** in the root folder:

```bash
npm run dev
```

- React website: the Vite URL printed in terminal (normally `http://localhost:5173`)
- API health check: `http://localhost:5000/api/health`

If you prefer separate terminals:

```bash
npm run dev --prefix server
npm run dev --prefix client
```

## 7. Test the demo

1. Connect MetaMask to the local Hardhat network.
2. Click **Register patient** in the Add encrypted medical record panel and confirm the transaction in MetaMask. Alternatively, you can register from the Hardhat console:

```bash
npx hardhat console --network localhost
```

```js
const c = await ethers.getContractAt("MedicalRecords", "PASTE_DEPLOYED_ADDRESS_HERE");
await (await c.registerPatient()).wait();
```

3. Return to the website and add fictional record information with an encryption password of at least 10 characters.
4. The encrypted payload uploads to IPFS; its CID and hash are saved on-chain.
5. Use **Record vault** to load records, then view/decrypt with the password.
6. In **Patient permissions**, grant/revoke a provider test wallet address. The provider must connect their wallet to read records; use the patient's encryption password separately to decrypt a record.



## Project structure

```text
Blockchain-Medical-Records/
├── contracts/MedicalRecords.sol
├── scripts/deploy.js
├── server/server.js
├── client/
│   ├── index.html
│   └── src/
│       ├── App.jsx
│       ├── styles.css
│       ├── main.jsx
│       └── contractConfig.json
├── .env.example
├── hardhat.config.js
└── package.json
```

## Design summary

- **Frontend:** React UI, MetaMask connection, encryption/decryption in Web Crypto API, and contract calls via ethers.js.
- **Backend:** Express endpoint forwards already-encrypted JSON to Pinata. It does not intentionally accept plaintext medical notes.
- **IPFS:** Stores the encrypted payload. A CID is a locator, not an access-control mechanism; encryption is essential.
- **Smart contract:** Patient owns their record list and controls which provider wallets can call read functions. Events create a verifiable audit trail.

## Known demo limitations / next improvements

- Add a visible register button (the registration handler is already implemented).
- Build an audit-event timeline by querying contract logs.
- Use a reviewed key-management scheme for provider-specific key sharing; do not share one password casually.
- Add authentication, rate limiting, server-side request validation, secure key backup/recovery, threat modelling, and professional security review before any real-world deployment.
- The example stores record type/title/date/notes inside the encrypted payload; do not put identifying information in on-chain fields or IPFS filenames.
- Record metadata reads are `view` calls; opening a record submits a separate `logRecordAccess` transaction for the audit event. This costs gas on a real network.
