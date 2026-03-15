# SatsPay

> **Send Bitcoin to any phone number. Claim it from anywhere.**

SatsPay is a Bitcoin-native remittance and payments protocol built on the [Stacks blockchain](https://stacks.co). It lets anyone send sBTC (Bitcoin secured by Stacks) to a phone number — no wallet required on the receiving end. Recipients get an SMS with a claim link and can collect their funds by connecting a wallet or cashing out to a local bank account.

Built in Lagos. Built for Africa. Built on Bitcoin.

---

## The Problem

Sending money across borders — or even across town — in Nigeria and across Africa is painful. Traditional remittance corridors are slow, expensive (fees of 5–10%), and require both parties to have bank accounts. Crypto alternatives exist but demand technical knowledge most people don't have: setting up wallets, copying addresses, understanding gas fees.

SatsPay fixes this. The sender needs a Bitcoin wallet. The recipient needs only a phone number and an SMS.

---

## How It Works

```
Sender                          SatsPay Protocol                     Recipient
  │                                     │                                 │
  ├─── enters phone number + amount ───►│                                 │
  │                                     ├─── locks sBTC in escrow ───────►│
  │                                     ├─── sends SMS claim link ────────►│
  │                                     │                                 │
  │                            Recipient opens link                       │
  │                                     │◄── connects wallet or ──────────┤
  │                                     │    enters bank account          │
  │                                     │                                 │
  │                                     ├─── releases sBTC to wallet ─────►│
  │                                     │    OR initiates NGN bank transfer►│
```

1. **Sender** opens SatsPay, connects their Leather or Xverse wallet, enters a Nigerian (or any) phone number and an sBTC amount
2. **Protocol** locks the sBTC in the on-chain escrow contract and sends the recipient an SMS with a unique claim link
3. **Recipient** opens the link, connects a wallet to claim sBTC directly — or enters their bank account number to receive Nigerian Naira (NGN) via Flutterwave/Paystack
4. If unclaimed after 30 days, the escrow automatically refunds the sender

---

## Repository Structure

This GitHub organization contains the following repositories:

| Repository | Description |
|---|---|
| [`satspay-contracts`](https://github.com/satspay/satspay-contracts) | Clarity smart contracts (escrow, registry, sBTC interface) |
| [`satspay-app`](https://github.com/satspay/satspay-app) | Next.js web application — send flow, claim flow, business dashboard |
| [`satspay-api`](https://github.com/satspay/satspay-api) | Backend API — transfer engine, claim manager, FX oracle, offramp connector |
| [`satspay-docs`](https://github.com/satspay/satspay-docs) | Protocol documentation, API references, integration guides |

---

## Tech Stack

### Blockchain
- **Stacks** — Layer for Bitcoin smart contracts
- **sBTC** — SIP-010 fungible token, 1:1 backed by Bitcoin
- **Clarity** — Smart contract language (decidable, no reentrancy bugs)
- **Hiro API** — Stacks node RPC for broadcasting and querying transactions

### Frontend
- **Next.js 14** — App router, server components
- **Stacks.js** — Wallet connection (Leather, Xverse), transaction signing
- **Tailwind CSS** — Styling
- **shadcn/ui** — Component library

### Backend
- **Node.js** — API runtime
- **PostgreSQL** — Users, transactions, claim records
- **Prisma** — ORM
- **Termii / Africa's Talking** — SMS gateway (Nigerian-optimized)
- **Flutterwave / Paystack** — NGN offramp (bank transfers)
- **CoinGecko / Pyth** — Live sBTC → NGN FX rates

---

## Smart Contracts Overview

SatsPay is powered by three Clarity contracts deployed on the Stacks blockchain:

### 1. `satspay-escrow`
The core of the protocol. Holds sBTC on behalf of recipients who haven't claimed yet. Enforces a 30-day expiry with automatic refunds. Only releases funds to a verified claim.

### 2. `satspay-registry`
Maps hashed phone numbers to Stacks wallet addresses. Once a recipient claims and registers their wallet, future sends to their phone number bypass escrow and go directly — making subsequent transfers instant.

### 3. `satspay-sbtc-interface`
A thin wrapper around the official sBTC SIP-010 contract. Abstracts token transfers so the escrow contract doesn't need to be updated if the underlying sBTC contract address ever changes.

See [`satspay-contracts`](https://github.com/satspay/satspay-contracts) for full contract documentation.

---

## Protocol Flow (Technical)

```
1. Sender calls satspay-escrow::send-to-phone(phone-hash, amount, expiry-blocks)
   └── transfers sBTC from sender → escrow contract
   └── stores claim record: { phone-hash, amount, sender, expiry }
   └── emits event: transfer-initiated

2. Backend detects event via Hiro API webhook
   └── generates UUID claim token
   └── sends SMS to recipient via Termii

3. Recipient opens claim link → connects wallet
   └── backend verifies claim token
   └── calls satspay-escrow::claim(claim-id, recipient-address)
   └── escrow releases sBTC → recipient wallet

4. Simultaneously (or later):
   └── satspay-registry::register(phone-hash, recipient-address) is called
   └── future sends to this phone skip escrow entirely

5. If unclaimed after expiry-blocks:
   └── satspay-escrow::reclaim(claim-id) can be called by original sender
   └── sBTC returns to sender
```

---

## Getting Started

### Prerequisites

- Node.js 18+
- [Clarinet](https://github.com/hirosystems/clarinet) (Clarity development environment)
- A Stacks testnet wallet (Leather or Xverse browser extension)
- PostgreSQL 15+

### Clone all repositories

```bash
# Clone the contracts
git clone https://github.com/satspay/satspay-contracts
cd satspay-contracts && clarinet check

# Clone the app
git clone https://github.com/satspay/satspay-app
cd satspay-app && npm install && npm run dev

# Clone the API
git clone https://github.com/satspay/satspay-api
cd satspay-api && npm install && cp .env.example .env
```

### Environment Variables (API)

```env
# Stacks
STACKS_API_URL=https://api.hiro.so
STACKS_NETWORK=testnet
CONTRACT_DEPLOYER_ADDRESS=ST...
ESCROW_CONTRACT_NAME=satspay-escrow
REGISTRY_CONTRACT_NAME=satspay-registry

# Database
DATABASE_URL=postgresql://user:password@localhost:5432/satspay

# SMS
TERMII_API_KEY=your_termii_key
TERMII_SENDER_ID=SatsPay

# FX
COINGECKO_API_KEY=your_key

# Offramp (v2)
FLUTTERWAVE_SECRET_KEY=your_key
PAYSTACK_SECRET_KEY=your_key

# App
NEXT_PUBLIC_APP_URL=http://localhost:3000
JWT_SECRET=your_secret
```

---

## Roadmap

### Hackathon MVP (v0.1)
- [x] Escrow Clarity contract
- [x] Registry Clarity contract
- [ ] Send flow (Next.js + Stacks.js)
- [ ] Claim page (wallet connect + onchain claim)
- [ ] SMS notification (Termii)
- [ ] Testnet deployment

### v0.2 — Post-hackathon
- [ ] Bank cashout via Flutterwave/Paystack (NGN offramp)
- [ ] Business dashboard (CSV payroll upload)
- [ ] Live FX rate display (sBTC → NGN)
- [ ] Transaction history
- [ ] Mainnet deployment

### v0.3 — Growth
- [ ] WhatsApp notifications
- [ ] Multi-country support (GHS, KES, ZAR)
- [ ] Direct send (bypass escrow for registered wallets)
- [ ] Mobile app (React Native)
- [ ] Business API (programmatic payroll integration)

### v1.0 — Protocol
- [ ] Undercollateralized credit scoring (onchain history)
- [ ] sBTC savings vaults
- [ ] Multi-sig business accounts
- [ ] Open protocol — third-party integrations

---

## Security

- Phone numbers are **never stored in plaintext** — only SHA-256 hashes are stored onchain
- Claim tokens are single-use UUIDs with 48-hour expiry
- All contract calls require explicit wallet signing — no custodial key management
- Escrow contract has been written to avoid Clarity reentrancy patterns
- 30-day fund expiry prevents permanent loss of unclaimed sBTC

### Reporting Vulnerabilities

Please do not open public issues for security vulnerabilities. Email **security@satspay.xyz** with details and we will respond within 48 hours.

---

## Contributing

SatsPay is open source and welcomes contributions. See [CONTRIBUTING.md](./CONTRIBUTING.md) for guidelines.

1. Fork the relevant repository
2. Create a feature branch (`git checkout -b feature/your-feature`)
3. Commit your changes
4. Open a pull request against `main`

---

## License

MIT License — see [LICENSE](./LICENSE) for details.

---

## Built With ❤️ in Lagos

SatsPay was started as a hackathon project for [Buidl Battle 2](https://dorahacks.io/hackathon/buidlbattle2/buidl) on Stacks and continues as a full product. The mission is simple: make Bitcoin the easiest way to send money across Africa.

> *"The next billion Bitcoin users won't come from San Francisco. They'll come from Lagos, Nairobi, Accra, and Johannesburg — and they'll arrive via their phone number."*

---

**Links:** [Website](https://satspay.xyz) · [Twitter](https://twitter.com/satspay) · [Discord](https://discord.gg/satspay) · [Docs](https://docs.satspay.xyz)
