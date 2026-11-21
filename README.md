# VeriCred - Verify Without Revealing

VeriCred is an end-to-end academic credential prototype built from the FET'S LUCK pitch deck. A university signs a private credential, the student proves a CGPA lower bound without disclosing the CGPA, and a verifier checks the issuer and current status against an EVM registry.

The app also includes a land-document verification workflow based on the deck's signed credential, privacy, and selective-disclosure architecture.

It also includes private medical document storage and consent-based sharing. OCR detects likely sensitive lines in the browser, creates a redacted raster copy, and requires the owner to inspect/approve it before saving or sharing. Recipients get a time-limited, revocable link to only the redacted copy; the original stays encrypted in the owner's archive.

The website includes a persistent light/dark theme toggle and follows the device's preferred theme on first visit.

## What is included

- Responsive University, Student Wallet, Verifier, and Public Ledger views.
- `IssuerRegistry.sol` for authorized university wallets.
- `CredentialRegistry.sol` for credential fingerprint registration and issuer-controlled revocation.
- EIP-191 issuer signatures bound to the credential commitment, degree metadata, issue time, and chain.
- A local Schnorr OR proof over secp256k1 Pedersen commitments for the claim `CGPA >= threshold`.
- QR proof transfer, JSON proof download/import, and encrypted student wallet backup/restore.
- A local Node archive API. Student records are AES-256-GCM encrypted in the browser before the server writes them. The password is not sent to the API; the public EVM never receives personal details or grades.
- Hardhat local EVM startup and deployment scripts.
- Land owner, authorized human reviewer, and verifier workspaces for private property document review.
- Browser OCR for PDFs and photographs, encrypted local archive, authority-signed property credentials, QR sharing, and current on-chain status checks.
- Medical report and prescription upload, local OCR-assisted redaction, encrypted owner archive, explicit expiry, and sender-issued redacted-copy access links that can be revoked.

## Requirements

- Node.js 20 or later and npm.
- MetaMask (or another EIP-1193 browser wallet) for issuing and revoking credentials.

## Run the complete local stack

From this project directory in PowerShell:

```powershell
npm install
npm run start:local
```

Open [http://localhost:5173](http://localhost:5173). Keep the terminal open while using the app. The command starts:

- Vite UI at `http://localhost:5173`
- Encrypted archive API at `http://localhost:3001`
- Hardhat JSON-RPC chain at `http://127.0.0.1:8545` (chain ID `31337`)

The local startup compiles and deploys both contracts and writes their addresses to the ignored generated file `public/chain-config.json`.

### Connect the local issuer account

The Hardhat node prints several development accounts and their private keys in the terminal. In MetaMask:

1. Add a network with RPC URL `http://127.0.0.1:8545`, chain ID `31337`, currency symbol `ETH`.
2. Import the private key for Hardhat account #1 (the demo university issuer).
3. Connect it in VeriCred. The header should show `authorized issuer`.
4. In the University tab, issue a credential. Use a unique 12+ character passphrase when prompted; this encrypts the archive copy in the local database.
5. Switch to Student Wallet, set the requirement, and create a proof/QR. The student's score and blinding secret remain inside the local wallet.
6. Switch to Verifier and scan the QR, import the downloaded proof JSON, or paste the proof payload. The verifier checks the issuer signature, proof, issuer allow-list, and live revocation status.
7. Use `Revoke on-chain` in the University tab, then verify the same proof again to see it rejected.

Hardhat development keys are public test keys. Never use them on a public network or with real funds.

### Land document verification

1. In MetaMask, import Hardhat account #2 (the local demo land authority) and connect it when using the Authority tab. The local startup deploys this account as an authorized issuer.
2. Open **Land documents** and select **Property owner**. Upload a PDF or photo for a property, choose the document type, and enter an archive password. OCR runs in the browser and suggests owner, deed/registration number, plot/survey number, khatian, and area fields. Review every field; OCR can be wrong.
3. Select the claims a verifier may see, then submit. The file and private fields are encrypted in the browser before they reach the loopback archive API. The authority wallet reviews the pending submission, sets the recorded ownership/mutation status, and signs or rejects it.
4. A verified owner can show or download the QR certificate. A buyer, bank, or lawyer can scan it or import its payload in the **Verifier** tab. The verifier checks the signature, authorized issuer, chain, current registry status, and latest archive status, then sees only signed claims the owner selected for disclosure.

The pitch deck proposes government land-record API checking where available. No official API, credentials, or jurisdiction-specific schema were supplied with this project, so the app routes new submissions to authorized human review and does not claim government-record matching. The local demo authority account is only a development issuer. Real deployment needs a real authority identity and onboarding, production authentication and authorization, secure key management, an approved official integration or operational review process, and legal/security review. Do not use this prototype for real property transactions.

PDF/image OCR is performed in the browser using Tesseract.js and PDF.js. The first OCR run may need internet access to retrieve Tesseract's worker and English model; the uploaded document itself is processed locally and encrypted before archive upload.

### Medical records and consent sharing

1. Open **Private medical records**, choose a report type, upload a PDF/photo, and set the archive expiry date. After the date, the local API deletes the encrypted record and associated grants.
2. Select **Detect & redact sensitive details**. OCR flags lines that appear to contain patient identity/contact details and paints them over in a raster copy. Inspect every preview page and confirm the review before saving. Detection is heuristic and can miss sensitive information or cover useful medical content; the sender is responsible for checking the copy. The original is encrypted separately and is never included in a share grant.
3. Choose **Create share link**, enter the record archive passphrase, and set an access expiry no later than the document cutoff. The app creates a random decryption key and puts it in the URL fragment (not the API request). Share the link only with the intended recipient. Revoke any active grant from the saved-record list.
4. A recipient opens the link while the app/API is running to view or download the redacted page images. The API denies access after expiry or revocation. A grant is a bearer link: anyone it is forwarded to can access its redacted copy while active. Revocation blocks future downloads but cannot erase a copy the recipient already downloaded.

The local archive stores encrypted medical content and redacted-share envelopes; metadata is limited to an opaque ID, creation date, and expiry date for server enforcement. Expired records and grants are purged from this local archive. The local demo is bound to `127.0.0.1`, so a recipient on another device cannot reach it until the app is deployed with a properly secured public backend. Do not upload real medical records to this prototype. OCR redaction is not a substitute for manual redaction or a privacy/security review.

### Student wallet handoff

The local browser wallet contains private student data. To move it between browsers, choose **Export student wallet** and set a strong password; import the resulting encrypted JSON in the other browser. The **Find encrypted archive** action can also retrieve a record from the local API when the student has the archive passphrase.

## Public testnet deployment

Copy `.env.example` to `.env` and set the RPC endpoint and deployment key. Do not commit `.env` or private keys. Deploy on Sepolia with:

```powershell
npm run deploy:sepolia
```

`PRIVATE_KEY` owns the issuer registry. `ISSUER_PRIVATE_KEY` is the university wallet registered at deployment; `LAND_AUTHORITY_PRIVATE_KEY` is the land authority wallet. Keys may be shared for a single-operator prototype. Deployment writes public addresses and issuer details to `public/chain-config.json`. The browser wallet must be configured for that same network. Deploying these demo contracts to a public testnet does not create a government verification integration or production identity system.

The public RPC fallback is for convenience and may have rate limits; configure your own RPC URL for sustained use.

## Privacy and security boundaries

- On-chain data: random fingerprint, issuer address, and issued/revoked status timestamps only.
- Student-private data: identity, roll number, CGPA, and proof blinding value. These stay in browser wallet storage and encrypted backups/archive envelopes.
- Proof public data: the signed credential commitment, degree metadata, threshold, and zero-knowledge proof. The exact CGPA is not included.
- This is a hackathon prototype. The custom range proof has not received an independent cryptographic audit and must not be used for real student records or high-stakes decisions until it is reviewed and replaced/approved by qualified cryptographers.
- Local archive files live under `.data/` and are excluded from Git. Back them up only as encrypted data and keep passphrases separate.
- Land archive API is bound to localhost and stores encrypted document envelopes plus non-sensitive workflow/status metadata. The local demo has no production account system, role provisioning, government API connection, or independent security audit. Never expose it to the public internet or upload real identity/property documents.
- Property QR payloads contain the authority signature, selected disclosures, document types, and credential reference needed for verification. Full documents and unselected owner fields are not written to the blockchain or included in the QR.

## Useful commands

```powershell
npm run build             # Build the web client
npm run compile:contracts # Compile the Solidity contracts
npm run chain             # Run only the local EVM
npm run api               # Run only the encrypted archive API
```
