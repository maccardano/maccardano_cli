# MACCARDANO CLI — Installation & User Guide

## Download

Go to **[Releases](../../releases/latest)** and download:
- **Linux**: `maccardano-cli`
- **Windows**: `maccardano-cli.exe`

---

## Required Tools

You need **one or more** of these depending on your wallet type:

### cardano-address (REQUIRED for seed phrase import)
```bash
# Linux
wget https://github.com/IntersectMBO/cardano-addresses/releases/download/3.12.0/cardano-addresses-3.12.0-linux64.tar.gz
tar xf cardano-addresses-3.12.0-linux64.tar.gz
sudo mv bin/cardano-address /usr/local/bin/
```
Windows: download from the same link, extract, add to PATH.

### cardano-hw-cli (REQUIRED for Ledger hardware wallet)
```bash
# Linux
wget https://github.com/vacuumlabs/cardano-hw-cli/releases/latest/download/cardano-hw-cli-linux
chmod +x cardano-hw-cli-linux
sudo mv cardano-hw-cli-linux /usr/local/bin/cardano-hw-cli
```
Windows: download `.exe` from the releases page, add to PATH.

### Blockfrost API Key (REQUIRED — free)
1. Go to **blockfrost.io** → Sign up free
2. Create a project → select **Mainnet**
3. Copy your API key (starts with `mainnetXXXXXX`)
4. You enter it on first run — saved automatically

---

## Linux Setup

```bash
# Make executable
chmod +x maccardano-cli

# Run
./maccardano-cli
```

---

## Windows Setup

Double-click `maccardano-cli.exe` or run from Command Prompt:
```bat
maccardano-cli.exe
```

---

## First Run

On first launch:
1. Enter your **Blockfrost mainnet API key**
2. The key is saved — you only enter it once

---

## How to Use

### Load Your Multisig Wallet

```
Main Menu → Load / Open Multi-Sig Wallet
→ Find my wallets (enter YOUR signer address)
```

Enter **your personal wallet address** (the one you use for signing, NOT the multisig address).
The CLI scans your transaction history and automatically finds all multisig wallets where you are a signer.

---

### Import Your Signing Key

```
Main Menu → Key Management
```

**Option A — From seed phrase (Typhon, Eternl, Lace):**
```
Key Management → Derive key from seed phrase
```
- Requires `cardano-address` installed
- Enter your 15 or 24-word seed phrase (visible for accuracy)
- Paste your wallet address to verify — must show ✅ MATCH
- Saves `payment.skey` + `payment.xprv`

**Option B — Ledger hardware wallet:**
```
Key Management → Get key hash from Ledger device
```
- Connect Ledger, open Cardano app
- Default path: `1852H/1815H/0H/0/0`
- For second Ledger: connect it and run again

**Option C — Tangem card:**
- Open **Tangem Cardano Signer** app on Android
- Tap "List Card Wallets" — copy the key hash
- Enter it when prompted in the CLI

---

### Create a Send Proposal

```
Wallet Actions → Create Send Proposal
```

1. Enter recipient address
2. Enter amount in ADA
3. Confirm
4. Share the **Proposal ID** (8 characters, e.g. `084953c9`) with all co-signers

**Note:** ~0.3 ADA network fee paid from YOUR wallet. 1 ADA service fee included in the multisig transaction itself.

---

### Sign a Proposal (each co-signer)

```
Wallet Actions → Sign Proposal
→ Enter Proposal ID
→ Choose: skey / Ledger / Tangem
```

**For Ledger:** Connect your device, open Cardano app, press Enter when prompted. Sign on device screen.

**For Tangem:** A QR code appears in terminal. Scan with Tangem Cardano Signer app → sign with card → paste the JSON response back.

**Important:** Each signer runs the CLI on their own PC. The signature is saved as a `.json` file in `~/.maccardano_wallets/sigs/`.

When prompted for a label, use a clear name: `typhon`, `ledger1`, `ledger2`, `tangem`

---

### Execute the Proposal

Once all required signatures are collected:

```
Wallet Actions → Execute Proposal
→ Enter Proposal ID
```

The CLI automatically finds all saved signature files.
If a co-signer is on a different PC, they send you their `.json` file — place it in `~/.maccardano_wallets/sigs/` before executing.

Confirm submission → transaction is sent to mainnet.

---

## Fees

| Action | Cost |
|--------|------|
| Proposal creation | ~0.3 ADA (network fee from your wallet) |
| Each execution | 1 ADA service fee (automatic, included in TX) |
| Signing | Free |

---

## File Locations

| File | Location |
|------|----------|
| Config (API key) | `~/.maccardano_cli.json` |
| Saved wallets | `~/.maccardano_wallets/` |
| Signature files | `~/.maccardano_wallets/sigs/` |
| Signing keys | wherever you chose during import |

**Windows paths:** replace `~` with `C:\Users\YourName\`

---

## Troubleshooting

**"No UTxOs in proposer wallet"**
When loading your skey, it shows a derived address. If it doesn't match your actual wallet, paste your real wallet address when prompted.

**"OutsideValidityIntervalUTxO"**
The proposal expired (24-hour limit). Create a new proposal.

**"ScriptWitnessNotValidatingUTXOW"**
Not enough valid signatures. Make sure all required signers have signed.

**Two Ledgers show same key hash**
Both devices may use the same seed phrase. Each Ledger needs a unique seed for a unique key.

**Proposal not visible on maccardano.online**
The CLI uses a different metadata format. This is a backup tool — proposals created via CLI are not visible on the website and vice versa.

---

## Security

- Your seed phrase is NEVER stored
- All signing happens locally on your device
- The CLI communicates only with Blockfrost (blockchain API)
- Service fee address is protected and cannot be modified

---

*MACCARDANO CLI — Backup tool for [maccardano.online](https://maccardano.online)*
*Use this when the website is offline to manage your multisig wallets.*
