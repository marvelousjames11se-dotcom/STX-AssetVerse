
# STX-AssetVerse — Real-World Asset Tokenization Smart Contract

**STC-AssetVerse** allows real-world assets (like real estate, art, commodities, etc.) to be **registered, tokenized, and governed** via proposals and votes.
It integrates key features for real-world deployment:

* ✅ **Asset registration** with metadata and valuation
* 🪙 **Semi-Fungible Token (SFT)** issuance per asset (e.g., 100,000 tokens per asset)
* 💰 **Dividend distribution and claims** for token holders
* ⚖️ **Governance via proposals and voting**
* 🧾 **KYC validation** for regulated participation
* 📈 **Oracle-based price feeds**
* 🕒 **Time-limited voting and asset expiry controls**

---

## 🧩 Core Concepts

| Concept         | Description                                                                                   |
| --------------- | --------------------------------------------------------------------------------------------- |
| **Asset**       | A real-world item represented on-chain with value and metadata.                               |
| **Token**       | Each asset is fractionalized into `tokens-per-asset` semi-fungible tokens (default: 100,000). |
| **KYC**         | Each user must be verified and approved to hold or interact with certain assets.              |
| **Proposal**    | A governance mechanism for token holders to vote on asset-related decisions.                  |
| **Dividend**    | Periodic distribution of profits to token holders based on share.                             |
| **Oracle Feed** | External price feed providing updated valuations for assets.                                  |

---

## ⚙️ Contract Constants

| Constant           | Description                                |
| ------------------ | ------------------------------------------ |
| `MAX-ASSET-VALUE`  | 1 trillion (maximum asset valuation)       |
| `MIN-ASSET-VALUE`  | 1 thousand (minimum asset valuation)       |
| `MAX-DURATION`     | ~1 day in blocks (144)                     |
| `MIN-DURATION`     | ~1 hour in blocks (12)                     |
| `MAX-KYC-LEVEL`    | Maximum KYC level allowed (5)              |
| `MAX-EXPIRY`       | ~1 year in blocks (52,560)                 |
| `tokens-per-asset` | 100,000 SFTs per asset                     |
| `contract-owner`   | Account that deployed or owns the contract |

---

## 🧠 Data Structures

### 📦 `assets`

Stores registered real-world assets.

| Field               | Type               | Description                      |
| ------------------- | ------------------ | -------------------------------- |
| `owner`             | `principal`        | Contract owner or issuer         |
| `metadata-uri`      | `string-ascii 256` | Metadata link (e.g. IPFS, HTTPS) |
| `asset-value`       | `uint`             | Asset’s total value              |
| `is-locked`         | `bool`             | Prevents transfer if locked      |
| `creation-height`   | `uint`             | Block when registered            |
| `last-price-update` | `uint`             | Last oracle update               |
| `total-dividends`   | `uint`             | Total dividends distributed      |

---

### 💰 `token-balances`

Tracks token ownership per asset.

| Field      | Type        | Description      |
| ---------- | ----------- | ---------------- |
| `owner`    | `principal` | Token holder     |
| `asset-id` | `uint`      | Associated asset |
| `balance`  | `uint`      | Token amount     |

---

### 👤 `kyc-status`

KYC compliance information.

| Field         | Type   | Description                     |
| ------------- | ------ | ------------------------------- |
| `is-approved` | `bool` | Whether address is KYC approved |
| `level`       | `uint` | KYC level (1–5)                 |
| `expiry`      | `uint` | Expiration block height         |

---

### 🗳️ `proposals`

Governance proposals per asset.

| Field           | Type               | Description                            |
| --------------- | ------------------ | -------------------------------------- |
| `title`         | `string-ascii 256` | Description/title of proposal          |
| `asset-id`      | `uint`             | Asset being governed                   |
| `start-height`  | `uint`             | Voting start                           |
| `end-height`    | `uint`             | Voting end                             |
| `executed`      | `bool`             | Whether the proposal has been executed |
| `votes-for`     | `uint`             | Votes in favor                         |
| `votes-against` | `uint`             | Votes against                          |
| `minimum-votes` | `uint`             | Quorum requirement                     |

---

### 🧾 `votes`

Tracks how each user votes.

| Field         | Type        | Description                   |
| ------------- | ----------- | ----------------------------- |
| `proposal-id` | `uint`      | ID of proposal                |
| `voter`       | `principal` | Address of voter              |
| `vote-amount` | `uint`      | Number of tokens used in vote |

---

### 💸 `dividend-claims`

Tracks last dividend claim per holder per asset.

| Field                 | Type   | Description                                     |
| --------------------- | ------ | ----------------------------------------------- |
| `last-claimed-amount` | `uint` | Dividends already claimed up to total-dividends |

---

### 📊 `price-feeds`

Oracle price data for assets.

| Field          | Type        | Description          |
| -------------- | ----------- | -------------------- |
| `price`        | `uint`      | Current price        |
| `decimals`     | `uint`      | Decimal precision    |
| `last-updated` | `uint`      | Block height updated |
| `oracle`       | `principal` | Oracle provider      |

---

## 🧩 Validation Functions

Private helper functions ensure clean input data:

| Function                             | Purpose                                         |
| ------------------------------------ | ----------------------------------------------- |
| `validate-asset-value(value)`        | Ensures value within allowed range              |
| `validate-duration(duration)`        | Ensures duration between `MIN` and `MAX`        |
| `validate-kyc-level(level)`          | Checks if ≤ `MAX-KYC-LEVEL`                     |
| `validate-expiry(expiry)`            | Ensures expiry is valid and within `MAX-EXPIRY` |
| `validate-minimum-votes(vote-count)` | Ensures valid quorum requirement                |
| `validate-metadata-uri(uri)`         | Checks that metadata URI is not empty           |

---

## 🧭 Public Functions

### `register-asset(metadata-uri, asset-value)`

Registers a new real-world asset.

* Only `contract-owner` can call.
* Mints `tokens-per-asset` to the owner.

**Returns:** `(ok asset-id)`

---

### `claim-dividends(asset-id)`

Claims unclaimed dividends for the caller.

**Calculation:**
`claimable = (balance * (total-dividends - last-claimed)) / tokens-per-asset`

**Returns:** `(ok true)` if successful.

---

### `create-proposal(asset-id, title, duration, minimum-votes)`

Creates a governance proposal.

* Caller must hold at least **10% of total tokens**.
* Proposal lasts for `duration` blocks.

**Returns:** `(ok proposal-id)`

---

### `vote(proposal-id, vote-for, amount)`

Casts a vote for or against a proposal.

* Cannot vote after proposal end.
* Cannot vote twice on same proposal.

**Returns:** `(ok true)`

---

## 🔍 Read-Only Functions

| Function                            | Description                    |
| ----------------------------------- | ------------------------------ |
| `get-asset-info(asset-id)`          | Returns asset details          |
| `get-balance(owner, asset-id)`      | Returns token balance          |
| `get-proposal(proposal-id)`         | Returns proposal data          |
| `get-vote(proposal-id, voter)`      | Returns voter’s vote           |
| `get-price-feed(asset-id)`          | Returns oracle feed data       |
| `get-last-claim(asset-id, claimer)` | Returns last claimed dividends |

---

## 🧭 Usage Workflow

1. **Contract Owner Registers an Asset**

   ```clarity
   (contract-call? .StacksTokenize register-asset "ipfs://metadata123" u5000000)
   ```

2. **Tokens Minted**

   * `100,000` tokens minted to owner.

3. **Investors Obtain Tokens**

   * Transfer or sale mechanism (not yet implemented).

4. **Dividends Declared**

   * Owner updates `total-dividends`.

5. **Investors Claim Dividends**

   ```clarity
   (contract-call? .StacksTokenize claim-dividends u1)
   ```

6. **Governance Proposal Created**

   ```clarity
   (contract-call? .StacksTokenize create-proposal u1 "Increase rent yield" u50 u1000)
   ```

7. **Token Holders Vote**

   ```clarity
   (contract-call? .StacksTokenize vote u1 true u100)
   ```

8. **Proposal Executed (future feature)**

---

## ⚠️ Error Codes

| Code                          | Meaning                                     |
| ----------------------------- | ------------------------------------------- |
| `err-owner-only` (100)        | Only contract owner can perform this action |
| `err-not-found` (101)         | Asset or record not found                   |
| `err-already-listed` (102)    | Asset already registered                    |
| `err-invalid-amount` (103)    | Invalid amount supplied                     |
| `err-not-authorized` (104)    | Caller lacks permissions                    |
| `err-kyc-required` (105)      | KYC verification required                   |
| `err-vote-exists` (106)       | Vote already recorded                       |
| `err-vote-ended` (107)        | Voting period expired                       |
| `err-price-expired` (108)     | Price feed outdated                         |
| `err-invalid-uri` (110)       | Invalid metadata URI                        |
| `err-invalid-value` (111)     | Invalid asset value                         |
| `err-invalid-duration` (112)  | Invalid proposal duration                   |
| `err-invalid-kyc-level` (113) | Invalid KYC level                           |
| `err-invalid-expiry` (114)    | Invalid expiry date                         |
| `err-invalid-votes` (115)     | Invalid vote threshold                      |
| `err-invalid-address` (116)   | Invalid principal address                   |
| `err-invalid-title` (117)     | Invalid proposal title                      |

---

## 🚀 Future Enhancements

* [ ] Add **token transfer logic** (to trade asset shares).
* [ ] Add **dividend deposit and distribution functions**.
* [ ] Implement **proposal execution logic**.
* [ ] Integrate **external price oracles** (e.g., Chainlink or Hiro).
* [ ] Add **KYC management interface**.
* [ ] Add **event logging** for better transparency.

---

## 📜 License

MIT License © 2025
Developed for decentralized real-world asset tokenization on **Stacks** using **Clarity**.

