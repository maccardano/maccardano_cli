# MACCARDANO CLI — Installation Guide

**Linux & Windows (via WSL)**

---

## Download

Go to **[Releases](../../releases/latest)** and download:

- **Linux**: `maccardano-cli`
- **Windows**: `maccardano-cli` (same Linux binary, runs inside WSL — see Windows setup below)

---

## Windows Setup (WSL)

Windows users run the CLI inside **WSL (Windows Subsystem for Linux)** — a full Linux environment built into Windows 10/11. All commands are identical to Linux once WSL is installed.

### Install WSL

Open **PowerShell as Administrator** and run:

```powershell
wsl --install
```

This installs WSL 2 with Ubuntu automatically. Restart your PC when prompted.

After restart, Ubuntu opens and asks you to create a username and password. Choose anything — this is your Linux username inside WSL.

**Verify WSL is working:**
```powershell
wsl --list --verbose
```
You should see Ubuntu with STATE: Running.

### Open WSL terminal

From now on, all CLI commands run inside WSL. Open it anytime by:
- Pressing `Win + R` → type `wsl` → Enter
- Or searching "Ubuntu" in the Start menu

### Copy the binary into WSL

After downloading `maccardano-cli`, copy it into WSL:

```bash
# Inside WSL terminal
# Your Windows Downloads folder is accessible at:
cp /mnt/c/Users/YourWindowsUsername/Downloads/maccardano-cli ~/maccardano-cli
chmod +x ~/maccardano-cli
```

Replace `YourWindowsUsername` with your actual Windows username.

---

## Linux Setup

```bash
# Make executable
chmod +x maccardano-cli

# Run
./maccardano-cli
```

---

## Required Tools

Install these depending on your wallet type. All commands work the same on Linux and WSL.

### cardano-address (required for seed phrase import)

```bash
# Linux / WSL
wget https://github.com/IntersectMBO/cardano-addresses/releases/download/3.12.0/cardano-addresses-3.12.0-linux64.tar.gz
tar xf cardano-addresses-3.12.0-linux64.tar.gz
sudo mv bin/cardano-address /usr/local/bin/
cardano-address --version
```

### cardano-hw-cli (required for Ledger hardware wallet)

```bash
# Linux / WSL
wget https://github.com/vacuumlabs/cardano-hw-cli/releases/latest/download/cardano-hw-cli_1.16.0_linux-x64.tar.gz
tar xf cardano-hw-cli_1.16.0_linux-x64.tar.gz
sudo mv cardano-hw-cli /usr/local/bin/
cardano-hw-cli version
```

> **WSL + Ledger note:** Ledger USB access from WSL requires an extra step. See [Ledger on WSL](#ledger-on-wsl) below.

### Blockfrost API Key (required — free)

1. Go to **[blockfrost.io](https://blockfrost.io)** → Sign up free
2. Create a project → select **Mainnet**
3. Copy your API key (starts with `mainnet...`)
4. Enter it on first run — saved automatically to `~/.maccardano_cli.json`

---

## First Run

```bash
./maccardano-cli
```

On first launch you will be asked for your Blockfrost API key. Enter it once — saved permanently.

---

## How to Use

### Step 1 — Load Your Multisig Wallet

```
Main Menu → Load / Open Multi-Sig Wallet
→ Find my wallets (enter YOUR signer address)
```

Enter your **personal wallet address** (the one you sign with — NOT the multisig address).
The CLI scans your transaction history and finds all multisig wallets where you are a signer automatically.

---

### Step 2 — Import Your Signing Key

```
Main Menu → Key Management
```

**From seed phrase (Typhon, Eternl, Lace):**
```
Key Management → Derive key from seed phrase
```
- Requires `cardano-address` installed
- Enter your 15 or 24-word seed phrase
- Paste your wallet address to verify — must show ✅ MATCH
- Saves `payment.skey` + `payment.xprv` wherever you choose

**From Ledger hardware wallet:**
```
Key Management → Get key hash from Ledger device
```
- Connect Ledger, open Cardano app
- Default derivation path: `1852H/1815H/0H/0/0`
- For a second Ledger device: connect it and run again — always re-reads the connected device

**From Tangem card:**
- Open **Tangem Cardano Signer** app on Android
- Tap "List Card Wallets" → copy the key hash
- Enter it when prompted in the CLI

---

### Step 3 — Create a Send Proposal

```
Wallet Actions → Create Send Proposal
```

1. Enter recipient address
2. Enter amount in ADA
3. Confirm
4. Note the **Proposal ID** (8 characters, e.g. `084953c9`)
5. Share this ID with all co-signers

**Cost:** ~0.3 ADA network fee from YOUR wallet. The 1 ADA service fee is included inside the multisig transaction itself.

---

### Step 4 — Sign the Proposal (each co-signer)

```
Wallet Actions → Sign Proposal
→ Enter Proposal ID
→ Choose: skey / Ledger / Tangem
```

**Software key (.skey):**
Choose option 1, enter path to your `.skey` file.

**Ledger:**
Choose option 2. Connect your Ledger, open Cardano app, press Enter. Confirm on device screen.
Each time you sign with Ledger the CLI re-reads the connected device — swap devices freely.

**Tangem:**

Download: https://github.com/maccardano/maccardano_android/releases/

Choose option 3. A QR code appears in the terminal.
Scan it with **Tangem Cardano Signer** on Android → sign with card tap → paste the returned JSON back.

When saving the signature file, use a clear label:
```
Label for this signature file: typhon
Label for this signature file: ledger1
Label for this signature file: ledger2
Label for this signature file: tangem
```

Signature files are saved to `~/.maccardano_wallets/sigs/`

---

### Step 5 — Execute the Proposal

Once all required signatures are collected:

```
Wallet Actions → Execute Proposal
→ Enter Proposal ID
```

The CLI automatically finds all saved `.json` signature files for that proposal.

If a co-signer is on a **different PC**, they send you their `.json` file. Place it in:
```
~/.maccardano_wallets/sigs/
```
Then run Execute — it will find it automatically.

Confirm submission → transaction sent to mainnet.

---

## View Proposals

```
Wallet Actions → View On-Chain Proposals
```

Shows all proposals with:
- **Active** — TTL not expired, can still be signed and executed
- **Expired** — 24-hour window passed, needs a new proposal

Columns: ID, Type, Amount, Signatures collected, Recipient address

---

## Ledger on WSL

Ledger USB access requires forwarding the USB device from Windows to WSL.

**Install usbipd-win on Windows:**

Download from: https://github.com/dorssel/usbipd-win/releases

Install it, then in **PowerShell as Administrator:**

```powershell
# List USB devices
usbipd list

# Find your Ledger (shows as "Ledger" or similar)
# Note the BUSID (e.g. 2-3)

# Attach Ledger to WSL
usbipd attach --wsl --busid 2-3
```

Then inside WSL:

```bash
# Install USB tools
sudo apt install libusb-1.0-0-dev udev

# Add udev rule for Ledger
echo 'SUBSYSTEM=="usb", ATTR{idVendor}=="2c97", MODE="0660", GROUP="plugdev"' | sudo tee /etc/udev/rules.d/20-ledger.rules
sudo udevadm control --reload-rules
sudo usermod -aG plugdev $USER

# Verify Ledger is visible
lsusb | grep -i ledger
```

Re-run `usbipd attach` each time you reconnect the Ledger.

---

## File Locations

| File | Location |
|------|----------|
| Config (API key) | `~/.maccardano_cli.json` |
| Saved wallets | `~/.maccardano_wallets/` |
| Signature files | `~/.maccardano_wallets/sigs/` |
| Derived signing keys | wherever you chose during import |

**WSL file location from Windows Explorer:**
```
\\wsl$\Ubuntu\home\YourLinuxUsername\.maccardano_wallets\
```

---

## Troubleshooting

**"No UTxOs in proposer wallet"**
When loading your `.skey`, the CLI shows the derived address. If it does not match your actual wallet, paste your real wallet address when prompted.

**"OutsideValidityIntervalUTxO"**
The proposal expired — 24-hour limit passed. Create a new proposal.

**"ScriptWitnessNotValidatingUTXOW"**
Not enough valid signatures. All required signers must sign before executing.

**Two Ledgers show same key hash**
Both devices use the same seed phrase. Each Ledger needs a unique seed for a unique signing key.

**Proposal not visible on maccardano.online**
The CLI uses a different metadata format from the website. This is expected — the CLI is a standalone backup tool. Proposals created via CLI are not shown on the website and vice versa.

**WSL: permission denied running binary**
```bash
chmod +x ~/maccardano-cli
```

**WSL: cardano-address not found**
Make sure you moved it to `/usr/local/bin/` and it has execute permission:
```bash
sudo chmod +x /usr/local/bin/cardano-address
```

---

## Fees

| Action | Cost |
|--------|------|
| Proposal creation | ~0.3 ADA network fee (from your personal wallet) |
| Proposal execution | 1 ADA service fee (automatic, inside the multisig TX) |
| Signing | Free — no transaction needed |
| Key import | Free |

---

## Security

- Your seed phrase is **never stored** anywhere
- All signing happens **locally** on your device
- The CLI connects only to **Blockfrost** (official Cardano API)
- Service fee address is protected inside the compiled binary and cannot be modified
- Signature files (`.json`) contain only witness data — not your private key

---

*MACCARDANO CLI — Backup tool for [maccardano.online](https://maccardano.online)*
*Use this when the website is offline to manage your multisig wallets independently.*
