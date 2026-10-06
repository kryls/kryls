<div align="center">

<img src="k.png" width="110" alt="KRYLS Logo" /> 

<h1>
<span style="color:#00f0ff;">KRYLS</span>
<span style="color:#bd00ff;">•</span>
Liquid-Glass Web3 Freelancing MVP
</h1>

<p style="max-width:820px; opacity:0.92; line-height:1.7;">
A Web3 freelancing MVP featuring a neon <b>Liquid-Glass</b> UI, project marketplace, chat-based collaboration, and an on-chain escrow flow on Sepolia (with Kleros / XMTP-ready escrow contract design).
</p>

<p>
  <img alt="Solidity" src="https://img.shields.io/badge/Solidity-0.8.19-0b1220?style=for-the-badge&logo=solidity&logoColor=white"/>
  <img alt="Node" src="https://img.shields.io/badge/Node.js-Express-0b1220?style=for-the-badge&logo=node.js&logoColor=00f0ff"/>
  <img alt="Ethers" src="https://img.shields.io/badge/Ethers.js-5.7.2-0b1220?style=for-the-badge&logo=ethereum&logoColor=bd00ff"/>
  <img alt="Network" src="https://img.shields.io/badge/Sepolia-Escrow-0b1220?style=for-the-badge&logo=ethereum&logoColor=00f0ff"/>
  <img alt="Swap" src="https://img.shields.io/badge/Polygon-ParaSwap-0b1220?style=for-the-badge&logo=polygon&logoColor=00ff99"/>
</p>

</div>

---

## What is KRYLS?

KRYLS is a Web3 freelancing platform MVP built with plain HTML/CSS/JS on the frontend and a lightweight Node/Express backend using JSON files as a database.  
The platform supports wallet-based login, role selection (client / freelancer), posting and browsing projects, applying to projects, in-app chat, and an escrow approval flow that creates an on-chain escrow on Sepolia.  
A dedicated Solidity escrow contract (`KrylsEscrowProduction`) is included, designed for non-custodial payments with milestone support, dispute hooks, and optional XMTP features.

---

## Theme (Liquid Glass + Neon)

The UI design is based on a neon cyan + purple palette and glassmorphism components (blurred translucent panels, glowing borders).  
Core theme variables (e.g. `--primary: #00f0ff`, `--neon-purple: #bd00ff`) and the glass backdrop styles are defined in `style.css`.  
Premium mode applies a purple-forward theme by toggling `body.premium`, which swaps the main accent from cyan to purple.

---

## Pages

- **Home** (`index.html` + `script.js`)  
  Wallet connect, terms modal, role selection, and optional platform stats from the deployed contract.

- **Dashboard** (`dashboard.html` + `dashboard.js`)  
  Profile editing (username/bio/socials), role switching, project creation wizard (including milestone mode), premium purchase flow, and project management for both roles.

- **Market** (`market.html` + `market.js`)  
  Browse open projects, search/filter by tags, and submit applications (requests).

- **Chat** (`chat.html` + `chat.js`)  
  Conversation list + messages, request approval (creates escrow on-chain), terms negotiation, delivery submission/acceptance, and dispute workflow (MVP off-chain; contract-ready on-chain).

- **Swap** (`swap.html` + `swap.js`)  
  A token swap UI **powered by ParaSwap on Polygon**.

---

## High-level Flow

1. User connects a wallet and a server profile is created/loaded.
2. User selects a role: `client` or `freelancer`.
3. Client posts a project; backend stores it (JSON DB) and validates allowed tokens for Sepolia.
4. Freelancer applies to an open project (creates a request).
5. In Chat, the client approves a request:
   - frontend checks token approval and registration
   - calls `createProject(...)` on the Sepolia escrow contract
   - backend marks the project as “In Progress” and stores tx + on-chain id (if available)
6. During the project:
   - either side can propose new terms
   - freelancer can submit delivery
   - owner can accept delivery → project completes and chat auto-deletes after 24 hours (cleanup job)

---

## Networks & On-chain Contracts

### Sepolia Escrow

Frontend uses this escrow contract address:
- `0xFd2F7895c9D851288e8Afb3e86fb9Bd4A9153BBb`

Supported ERC-20 tokens (Sepolia):
- USDT: `0xaA8E23Fb1079EA71e0a56F48a2aA51851D8433D0`
- USDC: `0x1c7D4B196Cb0C7B01d743Fbc6116a902379C7238`
- DAI:  `0xFF34B3d4Aee8ddCd6F9AFFFB6Fe49bD371b8a357`

> The backend also enforces this token allowlist when posting projects.

### Polygon Swap

The Swap page is designed for Polygon (`chainId = 137`) and uses ParaSwap API endpoints (`apiv5.paraswap.io`) to quote/build swap transactions.

---

## Backend (Node/Express)

The backend (`server.js`) serves the static site and exposes a REST API under `/api`.  
It stores data in `db/*.json` files (profiles/projects/requests/messages/logs/terms/deliveries/disputes/oracles).  
Presence (“online/offline”) is tracked using an in-memory TTL map.

### Premium (one-time)

Premium is activated through `POST /api/premium/activate`.  
In the dashboard, the premium purchase flow sends an ERC-20 transfer (default: 30 units) and then calls the activation endpoint.  
Premium also enables “Pin project” with a cooldown of **1 pin / 24h**, and longer project lifetime (15 days vs 7 days).

> Important: `PREMIUMTREASURY` in `dashboard.js` is currently a placeholder. Replace it before using real funds.

### Auto Cleanup

A scheduled cleanup deletes chats/requests/terms/deliveries/disputes 24 hours after a project is completed.

---

## Admin Panel

Admin panel routes are protected with Basic Auth and are served at:
- `/admin`

Environment variables:
- `ADMINUSER` (default: `admin`)
- `ADMINPASS` (default: `kryls12345`)

Admin features include:
- view and edit users (role/username/bio/socials/premium)
- view projects, requests, messages, and logs

---

## Smart Contract: `KrylsEscrowProduction` (Solidity)

The escrow contract is designed as a non-custodial system for digital services with:
- full-payment and milestone payment types
- token allowlist (USDT/USDC/DAI pre-allowed)
- client registration fee (token decimals based)
- optional dispute lifecycle using Kleros ERC-792 (createDispute + `rule(...)` callback)
- configurable arbitrator allowlist and timelocked admin updates
- optional XMTP toggles and on-chain message event patterns

This repo includes the full contract source in `krylsEscrowPro.sol`.

---

## API (Quick Map)

Base URL: `/api`

### Auth & Profile
- `POST /api/login`
- `POST /api/set-role`
- `GET  /api/profile/:wallet`
- `POST /api/update-profile` (alias: `POST /api/profile/update`)

### Presence
- `POST /api/presence/ping`
- `GET  /api/presence/status?wallets=0x..,0x..`

### Projects
- `POST /api/post-project`
- `GET  /api/all-open-projects`
- `GET  /api/project/:id`
- `GET  /api/client-projects/:wallet`
- `GET  /api/freelancer-active/:wallet`
- `POST /api/cancel-project`

### Requests
- `POST /api/request-project`
- `GET  /api/requests/incoming/:wallet`
- `GET  /api/requests/mine/:wallet`
- `POST /api/requests/confirm`
- `POST /api/requests/reject`
- `POST /api/requests/hide`

### Terms / Delivery / Disputes (MVP Off-chain)
- `GET  /api/terms/pending/:projectId`
- `POST /api/terms/propose`
- `POST /api/terms/respond`
- `GET  /api/delivery/latest/:projectId`
- `POST /api/delivery/submit`
- `POST /api/delivery/accept`
- `POST /api/delivery/dispute`
- `GET  /api/dispute/latest/:projectId`
- `POST /api/dispute/resolve`
- `GET  /api/oracles`

### Chat
- `POST /api/send-message`
- `GET  /api/messages/:wallet`

### Admin (Protected)
- `GET  /api/admin/users`
- `POST /api/admin/users/update`
- `GET  /api/admin/projects`
- `GET  /api/admin/requests`
- `GET  /api/admin/messages?limit=...`
- `GET  /api/admin/logs?limit=...`

---

## Local Setup

### Prerequisites
- Node.js (recommended 18+)
- A browser wallet (MetaMask/Trust Wallet) for Sepolia/Polygon actions

### Install & Run
```bash
npm init -y
npm i express cors
node server.js
