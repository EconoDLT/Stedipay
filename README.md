# StediPay

**Stablecoin Payment Solution with Smart Swap Engine — for the Neobank Generation**

[![License: BUSL-1.1](https://img.shields.io/badge/License-BUSL--1.1-orange.svg)](LICENSE)
[![Copyright](https://img.shields.io/badge/%C2%A9%202024--2026-Hasret%20Ozan%20Sevim-blue.svg)](https://www.linkedin.com/in/hasret-ozan-sevim)
[![MiCA](https://img.shields.io/badge/MiCA-Design--Compliant-green)](https://eur-lex.europa.eu/eli/reg/2023/1114/oj)
[![TFR](https://img.shields.io/badge/TFR%202023%2F1113-Travel%20Rule%20Ready-blue)](https://eur-lex.europa.eu/eli/reg/2023/1113/oj)

> :warning: **Legal Notice**
> StediPay is a pilot portfolio project by Hasret Ozan Sevim — PhD candidate (Blockchain and Distributed Ledger Technology with Finance and Econometrics specialization, University of Camerino and Catholic University of Sacred Heart) and researcher specialising in blockchain interoperability, on-chain finance, and tokenized assets.

> **Portfolio / Pilot Notice:** All smart contracts are unaudited. Financial projections are hypothetical. Not legal advice.

> **Language Model / AI Assistance Notice:** This project is developed to contribute my personal portfolio and show my capabilities and skills. I utilized LLMs only to brainstorm the idea, optimize the steps of the project design and management, and optimize the code during the process. The gap that this project fills (a general and hypothetical framework of a stablecoin card with compliance to the Euro-zone countries and smart swap capabilities), the originality, the choice of mechanical and technical deployments, the choice of functions used in the smart contract deployment and the creation of the project files are specifically my own output. The statistics of the business projections are random and hypothetical.

> **Technical Skills Used For This Project:** Python, HTML, Streamlit, Pandas, NumPy, Plotly.

> **The Background Knowledge Required For This Project:** Blockhain fundamentals (on-chain transactions, wallet types, network stacks), smart contracts & token standards, EU financia regulation, DeFi protocols (AAVE, Spark, Uniswap, Curve, Chainlink DONs, CCIP, LayerZero etc.), Compliance & AML, financial product design (reserve management, yield farming, cashback models, liquidity buffers), legal analysis (ability to interpret regulatory texts and map them to technical implementation), economic modelling (APY calculations, risk management, spread analysis).

> **In case you're interested in researching, developing, deploying, or managing a similar product, you can contact me via: hasretozan.sevim@unicatt.it**

---

## Overview

StediPay is a modular stablecoin payment infrastructure layer for neobank card products. It enables consumers to pay with MiCA-authorised EMTs (EURC, EUROe, EURCV, EURQ) through a standard Visa/Mastercard card, settling on-chain in near real-time.

StediPay's defining feature is the **Smart Swap Engine** — a MiCA-compliant mechanism that, at the moment of every card payment, scans real-time prices of MiCA-authorised same-currency stablecoins across whitelisted DEX pools. If a price improvement exists (i.e. the pool temporarily misprices one EMT relative to another), the engine executes an atomic swap to capture the spread and credits the surplus to the user as **transaction cashback** — a better effective exchange rate, not interest.

This is legally structured as a **best-execution service** under MiCA Art. 78 (obligation to obtain the best possible result for clients when executing orders), not as a yield-bearing stablecoin product prohibited by MiCA Art. 40/50. The engine only interacts with MiCA-authorised EMTs from a user-configurable whitelist of credible issuers.

```
User pays with card
       ↓
BaaS Neobank Issuer (KYC/AML · PSD3/PSR licence · MiCA CASP)
       ↓
StediPay SDK (reserve check · Travel Rule assembly · compliance screening)
       ↓
Smart Swap Engine ← checks MiCA-authorised EMT pools in real time
       │    ├── EURC/EUROe/EURCV/EURQ price scan (whitelisted issuers only)
       │    ├── atomic swap if spread > 0.05% threshold (covers gas)
       │    └── cashback = spread surplus → user balance (best-execution credit)
       ↓
AI Agent (rules-bound · ERC-4337 · permission-scoped)
       ├── On-chain EMT settlement (Polygon PoS, ~2s)
       ├── Travel Rule data → beneficiary CASP (TFR Art. 14)
       ├── Potential Yield farming via ERC-4626 Vault
       ├── Cross-chain routing
       └── DORA ICT log + CARF reporting record
```

---

## Smart Swap Engine — Design & Legal Framework

### What it does

At the moment of every card payment, the Smart Swap Engine:

1. **Scans** real-time prices of all MiCA-authorised euro-pegged EMTs across whitelisted Uniswap v4 and Curve pools on Polygon PoS.
2. **Compares** the current market price of each EMT pair (e.g. EURC/EUROe, EURC/EURCV) against the theoretical parity (1:1 for same-currency EMTs).
3. **Executes** an atomic flash-swap if a spread above the minimum profitability threshold (0.05% — chosen to cover gas costs and slippage) exists.
4. **Credits** the net surplus to the user's StediPay balance as instant cashback — a price improvement on the transaction, not passive interest.
5. **Reverts** atomically if the swap would result in a loss — the user's payment is never at risk.

### MiCA-authorised EMT whitelist (user-configurable, April 2026)

| EMT                      | Issuer                    | MiCA Authorisation                      | Currency Peg |
| ------------------------ | ------------------------- | --------------------------------------- | ------------ |
| EURC                     | Circle Europe S.A.S.      | EMI licence, France (ACPR)              | EUR          |
| EUR CoinVertible (EURCV) | Société Générale–Forge    | Credit institution, France              | EUR          |
| EUROe                    | Membrane Finance Oy       | EMI licence, Finland (Finanssivalvonta) | EUR          |
| EURQ                     | Quantoz Payments B.V.     | EMI licence, Netherlands (DNB)          | EUR          |
| USDC                     | Circle Internet Financial | EMI licence, France (ACPR)              | USD          |

> Only MiCA-authorised EMTs from the ESMA registry are whitelisted. Non-compliant tokens (USDT, DAI, FRAX) are permanently excluded. ESMA has made clear that CASPs must restrict services on non-MiCA-compliant stablecoins (ESMA statement, 17 Jan 2025).

### Legal framing — why this is compliant

| Legal question                                     | Analysis                                                                                                                                                                                                                                                                                                        |
| -------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **MiCA Art. 40/50 — interest prohibition on EMTs** | The Smart Swap Engine generates a **price improvement on a transaction**, not interest paid for holding time. The gain arises from an active exchange execution, not from a balance duration. This is legally distinct from the interest prohibition.                                                           |
| **MiCA Art. 78 — best execution**                  | CASPs executing orders must obtain the best possible result for clients. The Smart Swap Engine is a best-execution mechanism — scanning available prices and routing to the most favourable EMT.                                                                                                                |
| **MiCA Title V — CASP exchange services**          | Executing EMT-to-EMT swaps is a regulated CASP activity. StediPay's BaaS partner holds CASP authorisation; StediPay operates as technical service provider.                                                                                                                                                     |
| **MiCA Art. 76 — market integrity**                | The engine only engages with natural price discrepancies in open liquidity pools. It does not manipulate prices or exploit inside information. All activity is transparent and on-chain.                                                                                                                        |
| **TFR 2023/1113 — Travel Rule**                    | EMT-to-EMT swaps within the same user's StediPay account are intra-account transfers (not CASP-to-CASP transfers), so TFR Travel Rule obligations do not apply to the swap itself. The outbound payment to the merchant's CASP does trigger TFR.                                                                |
| **PSD3/PSR**                                       | The swap is executed as part of a payment order — the BaaS partner's PSP licence covers the payment service. StediPay is a technical service provider.                                                                                                                                                          |
| **AI Act**                                         | The swap decision is made by the Smart Swap Engine within hard-coded permission boundaries (whitelisted tokens only, minimum spread threshold, atomic revert if loss). This is a deterministic rules-engine, not a high-risk AI decision — reducing AI Act Annex III exposure relative to the broader AI agent. |

### Risk controls

- **Whitelist-only:** only MiCA-authorised EMTs from verified issuers
- **Atomic execution:** swap reverts automatically if net result is negative — user's payment is protected at all times
- **Minimum spread threshold:** 0.05% — below this, no swap is attempted (gas exceeds gain)
- **Maximum swap size:** configurable per user (default: €500/transaction, €2,000/day)
- **Slippage protection:** maximum 0.1% slippage tolerance on Uniswap v4 concentrated liquidity pools
- **User control:** users select which EMT issuers they trust; engine only uses approved issuers
- **Opt-out:** Smart Swap Engine can be disabled per user — plain EURC settlement is always available
- **On-chain transparency:** every swap is logged on-chain with pool price, spread captured, and cashback amount

---

## Architecture

### Payment Flow (with Smart Swap Engine)

```
┌─────────────────────────────────────────────────────────────────────┐
│  USER LAYER       Card App / ERC-4337 Smart Account                 │
│                   → Configure trusted EMT issuer whitelist          │
├─────────────────────────────────────────────────────────────────────┤
│  NEOBANK LAYER    BaaS Issuer · Visa/MC · KYC/CDD · PSD3/PSR        │
│                   MiCA CASP authorisation                           │
├─────────────────────────────────────────────────────────────────────┤
│  STEDI PAY LAYER  SDK · Smart Swap Engine · AI Agent                │
│                   Travel Rule Module · Reserve Manager · DORA Log   │
│                   ┌── Smart Swap Engine ──────────────────────────┐ │
│                   │  scan → compare → swap → credit cashback      │ │
│                   │  MiCA-authorised EMTs only · atomic · opt-out │ │
│                   └───────────────────────────────────────────────┘ │
├─────────────────────────────────────────────────────────────────────┤
│  ON/OFF-RAMP      On-ramp liq. · Off-ramp liq. · Wallet/Custody    │
├─────────────────────────────────────────────────────────────────────┤
│  BLOCKCHAIN       Polygon PoS · Ethereum L1 · A Multi-Chain Bridge  │
│                          Uniswap v2 · Curve ·                       │
└─────────────────────────────────────────────────────────────────────┘
```

### Smart Contracts

| Contract                 | Network          | Description                                               |
| ------------------------ | ---------------- | --------------------------------------------------------- |
| `StediPayRegistry.sol`   | ETH L1 + Polygon | Protocol registry, EMT whitelist, CASP access control     |
| `StediPayAccount.sol`    | Polygon          | ERC-4337 smart account                                    |
| `StediPaySwapEngine.sol` | Polygon          | Smart Swap Engine — EMT price scan, atomic swap, cashback |
| `StediPayAgent.sol`      | Polygon          | Rules-bound AI agent module                               |
| `StediPayVault.sol`      | ETH L1           | ERC-4626 yield vault                                      |
| `StediPayBridge.sol`     | ETH L1 + Polygon | Two multi-chain bridges (primary and fallback)            |
| `StediPayTravelRule.sol` | Polygon          | TFR compliance registry — IVMS101 packet hash log         |

### StediPaySwapEngine.sol — simplified interface

```solidity
interface IStediPaySwapEngine {
    struct SwapParams {
        address tokenIn;        // User's current EMT (e.g. EURC)
        uint256 amountIn;       // Payment amount
        address[] whitelist;    // User-approved MiCA-authorised EMTs
        uint256 minSpreadBps;   // Minimum spread to trigger swap (default: 5 bps = 0.05%)
        uint256 maxSlippageBps; // Maximum slippage tolerance (default: 10 bps = 0.1%)
    }

    struct SwapResult {
        address tokenUsed;      // EMT actually used for settlement
        uint256 amountSettled;  // Amount sent to merchant
        uint256 cashback;       // Spread surplus credited to user (in EURC)
        bool swapExecuted;      // True if a profitable swap was found
        bytes32 poolId;         // Uniswap v4 pool used (for on-chain audit)
    }

    // Scan whitelist pools, execute best swap, return result
    // Reverts atomically if no profitable route found — falls back to tokenIn
    function executeSmartSwap(SwapParams calldata params)
        external returns (SwapResult memory result);

    // View: returns best available spread without executing
    function quoteBestSpread(address tokenIn, address[] calldata whitelist)
        external view returns (address bestToken, uint256 spreadBps);
}
```

---

## Regulatory Compliance (April 2026)

| Regulation                           | Status                        | StediPay Posture                                                                                                                                                     |
| ------------------------------------ | ----------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **MiCA** (EU 2023/1114)              | In force                      | EURC/EUROe/EURCV/EURQ as MiCA-authorised EMTs. Smart Swap Engine = best-execution service (Art. 78), not interest (Art. 40/50). CASP authorisation via BaaS partner. |
| **TFR / Travel Rule** (EU 2023/1113) | In force — zero threshold     | IVMS101 on all CASP-to-CASP transfers. Swap Engine intra-account swaps do not trigger TFR. Outbound merchant payment does.                                           |
| **PSD3 / PSR**                       | Agreed — ~late 2027           | BaaS partner holds PSP/EMI licence. StediPay = technical service provider (PSR Art. 2).                                                                              |
| **DORA** (EU 2022/2554)              | In force                      | Critical ICT TPSP. ICT risk register, DORA clauses for all sub-processors.                                                                                           |
| **AI Act** (EU 2024/1689)            | High-risk provisions Aug 2026 | Swap Engine = deterministic rules-engine (not high-risk AI). Broader AI agent: Annex III §5(b) conformity assessment in progress.                                    |
| **MiCA Art. 76 — market integrity**  | Design-compliant              | Engine uses open pool prices only; no price manipulation; all activity on-chain and auditable.                                                                       |
| **GDPR**                             | Ongoing                       | Privacy-by-design. TFR data sharing safeguards.                                                                                                                      |
| **CARF**                             | EU 2026–2027                  | Reporting hooks on all transactions and swaps.                                                                                                                       |

---

## Business Model

| Revenue stream          | Model                                         |
| ----------------------- | --------------------------------------------- |
| Interchange share       | 0.3% of payment volume                        |
| Smart Swap spread share | 20% of cashback spread captured (80% to user) |
| Yield spread            | 30% of DeFi yield on reserves (ERC-4626)      |
| SaaS API licence        | €2,000–€15,000/month per neobank partner      |
| User subscription       | €4.99/month premium tier                      |
| Merchant API            | Per-call pricing                              |

**Year 3 targets:** €6.1M ARR, 45,000 active cardholders, €150M payment volume, break-even Month 26.

---

## Roadmap

- [x] Phase 0: Repository, smart contract architecture, Streamlit demo, regulatory documentation
- [ ] Phase 1 (Q1 2027): Testnet deployment, Smart Swap Engine v1 on Amoy, TFR module, first neobank MOU
- [ ] Phase 2 (Q3 2027): Mainnet Polygon, card issuing live, AAVE yield, AI Act conformity assessment
- [ ] Phase 3 (Q4 2027): Multi-chain, 3 B2B partners, CASP authorisation application
- [ ] Phase 4 (2028+): EU EMI licence, Series A, 310K users, CARF exchanges

---

## Intellectual Property & Licence

**© 2024–2026 Hasret Ozan Sevim. All rights reserved.**

Licensed under the **Business Source License 1.1 (BUSL-1.1)**. Non-commercial/research use permitted. Commercial use requires a licence. See [LICENSE](LICENSE).

---

## Legal Disclaimer & Important Notices

> **This section is legally important. Please read it before using, referencing, or sharing any part of this project.**

### 1. Pilot Portfolio Project — Not a Real Product

StediPay is a **pilot portfolio project** created solely to demonstrate technical, regulatory, and business design skills by Hasret Ozan Sevim. It does not represent a live product, a commercial service, or an active business. Nothing in this repository, the associated website, the Streamlit demo, or the business booklet constitutes an offer, solicitation, or invitation to invest, purchase, or engage in any commercial activity of any kind.

### 2. All Data and Projections Are Strictly Hypothetical

All statistics, financial projections, KPIs, revenue figures, user numbers, APY estimates, payment volumes, transaction volumes, spread estimates, and technical performance metrics are **strictly hypothetical, illustrative, and for portfolio purposes only**. They have no basis in real trading data, real user behaviour, or real financial results. They must not be relied upon for any financial, investment, or business decision.

### 3. No Affiliation, Partnership, or Endorsement — Whatsoever

StediPay has **no partnership, affiliation, endorsement, sponsorship, approval, communication, or commercial relationship of any kind** with any company, protocol, product, or brand mentioned anywhere in this project. This includes, without limitation:

All brand names, product names, and trademarks are the **exclusive property of their respective owners**. They are referenced in this project solely for descriptive and illustrative purposes — to explain what a hypothetical real-world implementation _could_ use — under the doctrine of nominative fair use. No association, approval, endorsement, or sponsorship by any of these parties is implied or should be inferred.

### 4. No Financial or Legal Advice

Nothing in this project constitutes financial advice, investment advice, legal advice, tax advice, or any other regulated professional advice. The regulatory analysis contained in this project represents the author's personal understanding of EU law as of April 2026 and is provided for informational and portfolio purposes only. It does not constitute a formal legal opinion and must not be relied upon as such. Always consult qualified legal and financial professionals before making any decisions.

### 5. Smart Contracts — Unaudited, Not for Production

All smart contracts in this repository are **unaudited prototype code**. They have not undergone a professional security audit and **must not be deployed on any mainnet or used in any production environment under any circumstances**. The author accepts no responsibility for any loss, damage, or liability arising from any use of any code in this repository.

### 6. No Liability

To the maximum extent permitted by applicable law, **Hasret Ozan Sevim accepts no legal or financial liability of any kind** — direct, indirect, incidental, special, or consequential — arising from or related to this project, its content, its code, or any reliance placed upon it by any person or entity. Your use of anything in this project is entirely at your own risk.

### 7. Intellectual Property of Third Parties

The mention of third-party company names, protocol names, and trademarks in this project does not grant any rights in those trademarks. All third-party intellectual property remains the exclusive property of its respective owners. If you are a representative of any company mentioned and have concerns about how your brand is referenced, please contact hasretozan.sevim@unicam.it and any concern will be addressed promptly.

---

## About the Author

**Hasret Ozan Sevim** — PhD candidate (Blockchain and Distributed Ledger Technology with Finance and Econometrics specialization, University of Camerino and Catholic University of Sacred Heart) · Researcher specialising in blockchain interoperability, on-chain finance, and tokenized assets.

Research: Blockchain interoperability, on-chain finance, tokenization, AI agent governance, EU FinTech regulation.
Legal: MiCA, TFR/Travel Rule, PSD3/PSR, DORA, AI Act, GDPR, CARF, MiFID II.
Tech: Python, R, Stata, web3.py, Solidity.

→ [LinkedIn](https://www.linkedin.com/in/hasret-ozan-sevim/) | [Portfolio](https://stedipay.store) | [Email](hasretozan.sevim@unicam.it)

---

_© 2024–2026 Hasret Ozan Sevim — Business Source License 1.1. Commercial use requires a licence._
