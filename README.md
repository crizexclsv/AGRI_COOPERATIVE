# A.G.R.I. Cooperative
### Alliance for Growth in Rural Industries — Tokenized rice farmer payments on Stellar

> Eliminating 30-day payment delays for 75 rice farmers in Nueva Ecija, Central Luzon, Philippines through transparent, on-chain cooperative payouts powered by Soroban smart contracts.

---

## Problem

A 75-member rice farmer cooperative in Nueva Ecija, Central Luzon, Philippines loses **₱150,000/month (~$2,700 USD)** in harvest revenue due to:

- **30-day delayed payments** from rice mill middlemen after delivery
- **No transparent profit-sharing records** — farmers cannot audit or dispute their share
- **No banking access** — most members are unbanked and cannot receive wire transfers
- **Forced debt cycles** — farmers take high-interest informal loans to survive between payouts

The middleman controls the money. Farmers have no visibility and no recourse.

---

## Solution

A Soroban smart contract deployed on the Stellar network that:

1. **Registers** all cooperative members on-chain (wallet + identity)
2. **Logs** each farmer's harvest delivery (kg delivered per batch)
3. **Calculates** proportional USDC shares automatically based on contribution
4. **Disburses** payments directly to each farmer's Stellar wallet — instantly, with no middleman
5. **Records** a cumulative, auditable payment history per farmer for full transparency

Farmers receive USDC the moment the mill pays the cooperative. No waiting. No disputes. No middlemen.

---

## Stellar Features Used

| Feature | Usage |
|---|---|
| **Soroban Smart Contracts** | Core logic — delivery logging, share calculation, payment disbursement |
| **USDC Transfers** | Stablecoin payouts to farmer wallets (pegged to PHP via anchor) |
| **XLM** | Gas fees for contract execution |
| **Custom Token (AGRI)** | Optional cooperative membership token for governance |
| **Trustlines** | Farmer wallets establish trustline to USDC before receiving payment |
| **Clawback / Compliance** | Admin can flag inactive members and prevent erroneous payouts |

---

## Vision and Purpose

Smallholder farmers are the backbone of Philippine agriculture yet they are the last to be paid and the first to be exploited. A.G.R.I. Cooperative proves that blockchain is not just for crypto traders — it is infrastructure for rural economic justice.

By anchoring cooperative payments to Stellar's fast, low-cost network, every harvest becomes an on-chain record. Every payout becomes auditable. Every farmer becomes their own bank.

The long-term vision is to expand this model to rice cooperatives across Central Luzon, then to corn, sugarcane, and vegetable cooperatives across the Philippines and Southeast Asia — giving millions of smallholder farmers the financial transparency and immediacy they have always deserved.

---

## MVP Transaction Flow

```
Admin logs delivery (farmer wallet + kg + batch ID)
        ↓
Contract stores delivery record on-chain
        ↓
Admin calls disburse_payments (batch ID + total USDC from mill)
        ↓
Contract calculates each farmer's proportional share
        ↓
USDC transferred directly to each farmer wallet
        ↓
Payment event emitted → cumulative record updated
```

**Demo runtime: under 2 minutes.**

---

## Suggested MVP Timeline

| Week | Milestone |
|---|---|
| Week 1 | Soroban contract — `register_farmer`, `log_delivery`, `calculate_shares` |
| Week 2 | Soroban contract — `disburse_payments`, `get_payment_record`, full test suite |
| Week 3 | Frontend — admin dashboard (log delivery, trigger payout) + farmer view (payment history) |
| Week 4 | Testnet deploy, anchor integration (Coins.ph / GCash), demo polish |

---

## Prerequisites

Make sure the following are installed before building or testing:

**Rust toolchain**
```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
rustup target add wasm32-unknown-unknown
```

**Soroban CLI** (v21.x or later)
```bash
cargo install --locked soroban-cli --features opt
```

Verify your versions:
```bash
rustc --version       # rustc 1.78.0 or later
soroban --version     # soroban 21.x or later
```

---

## Build

Compile the contract to a Wasm binary optimized for deployment:

```bash
soroban contract build
```

The compiled output will be at:
```
target/wasm32-unknown-unknown/release/agri_cooperative.wasm
```

---

## Test

Run the full test suite (5 tests):

```bash
cargo test
```

Expected output:
```
running 5 tests
test tests::test_happy_path_full_disbursement       ... ok
test tests::test_unauthorized_disbursement_rejected ... ok
test tests::test_payment_record_stored_correctly    ... ok
test tests::test_duplicate_farmer_registration_rejected ... ok
test tests::test_cumulative_payment_across_batches  ... ok

test result: ok. 5 passed; 0 failed
```

---

## Deploy to Testnet

**Step 1 — Configure your testnet identity**
```bash
soroban keys generate --global my-admin --network testnet
soroban keys fund my-admin --network testnet
```

**Step 2 — Deploy the contract**
```bash
soroban contract deploy \
  --wasm target/wasm32-unknown-unknown/release/agri_cooperative.wasm \
  --source my-admin \
  --network testnet
```

Copy the returned Contract ID — you will need it for all CLI invocations below.

**Step 3 — Initialize the contract**
```bash
soroban contract invoke \
  --id <CONTRACT_ID> \
  --source my-admin \
  --network testnet \
  -- initialize \
  --admin <ADMIN_WALLET_ADDRESS> \
  --token_id <USDC_TOKEN_CONTRACT_ADDRESS>
```

---

## Sample CLI Invocations

Replace `<CONTRACT_ID>`, `<ADMIN_ADDRESS>`, and `<FARMER_ADDRESS>` with your actual values.

**Register a farmer**
```bash
soroban contract invoke \
  --id <CONTRACT_ID> \
  --source my-admin \
  --network testnet \
  -- register_farmer \
  --caller <ADMIN_ADDRESS> \
  --wallet GBZX4364PEPQTDICMIQDZ56K4T75QZCR4NBEYKO6PDRJAHZKGUOJPCR \
  --name "JuanDelaCruz"
```

**Log a harvest delivery**
```bash
soroban contract invoke \
  --id <CONTRACT_ID> \
  --source my-admin \
  --network testnet \
  -- log_delivery \
  --caller <ADMIN_ADDRESS> \
  --farmer GBZX4364PEPQTDICMIQDZ56K4T75QZCR4NBEYKO6PDRJAHZKGUOJPCR \
  --kg 300 \
  --batch_id 1
```

**Disburse payments for a batch**
```bash
soroban contract invoke \
  --id <CONTRACT_ID> \
  --source my-admin \
  --network testnet \
  -- disburse_payments \
  --caller <ADMIN_ADDRESS> \
  --batch_id 1 \
  --total_usdc 50000000000
```
> `50000000000` = 5,000 USDC in stroops (7 decimal places)

**Check a farmer's cumulative payment record**
```bash
soroban contract invoke \
  --id <CONTRACT_ID> \
  --source my-admin \
  --network testnet \
  -- get_payment_record \
  --farmer GBZX4364PEPQTDICMIQDZ56K4T75QZCR4NBEYKO6PDRJAHZKGUOJPCR
```


## License

MIT License

Copyright (c) 2026 A.G.R.I. Cooperative

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

##CONTRACT
https://stellar.expert/explorer/testnet/tx/2eacd3587b7a5f4cb127e5adcafbd6a26d57e77e238f4e5242c004521cdaf3bc

https://lab.stellar.org/r/testnet/contract/CBVARIHEJFR3NPP5K67WMDPHU7PBLRRHUXJQVHZUXJ3AUT74A3SDYVE2
