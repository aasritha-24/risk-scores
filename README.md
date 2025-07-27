# Wallet Risk Scoring

This repository contains wallet risk scoring pipeline built using on-chain transaction data fetched via the Covalent API. It extracts comprehensive wallet behavior and borrowing features, then trains an XGBoost binary classifier to produce probability-based risk scores.

---

## Data Collection Method

- **Source:** Transaction data is fetched for each wallet address using the [Covalent API](https://www.covalenthq.com/docs/api/) on the Ethereum mainnet.
- **Scope:** All available historical transactions are retrieved by iterating through all API pages until no more data is available.
- **Logs Enabled:** The API call requests transaction logs (event logs) to enable detection of protocol-specific events such as borrows, supplies, repayments, and liquidations.
- **Robust Fetching:** The pipeline automatically handles pagination, rate limiting, and errors to ensure completeness.
- **Wallet List:** Input wallet addresses are read from a CSV file (`wallets.csv`) with a `wallet_id` column.

---

## Feature Selection Rationale

Features are engineered to capture key behavioral and financial signals linked to wallet risk, informed by current DeFi lending and blockchain risk research:

- **Basic Wallet Activity Features:**
  - *Transaction count, wallet age, average activity rate:* Measure wallet maturity and engagement.
  - *Success rate and failed transaction count:* Identify operational risk or misuse.
  - *Average ETH inflow/outflow ratios and net inflow:* Detect anomalous financial patterns.
  - *Average gas, fee, and transaction value:* Proxy for transaction complexity and wallet sophistication.
  - *Average time between transactions:* Captures behavioral regularity or burstiness.

- **Borrowing Profile Features:**
  - *Total amount borrowed and borrowing transaction count:* Indicate exposure and credit usage.
  - *Max borrow-to-collateral ratio:* Reflects leverage and potential insolvency risk.
  - *Recent borrow growth (last 30 days vs. older activity):* Detects rapid accumulation of debt, a known risk factor.

These features collectively create a holistic picture of wallet behavior, combining transactional, financial, and borrowing characteristics essential to risk assessment.

---

## Scoring Method

- **Model:** XGBoost binary classifier trained to distinguish risky vs. non-risky wallets.
- **Target (`y`):** Binary label combining on-chain borrowing exposure, transaction failures, and low activity to ensure meaningful class balance.
- **Training:** Stratified train-test split with class imbalance handled via the `scale_pos_weight` parameter.
- **Score Generation:** Predicted probability of being high-risk (`y=1`) is transformed into a risk score between 0 (lowest risk) to 1000 (highest risk) using a power transformation:
  
  \[
  \text{score} = \left(1 - \text{probability}\right)^{\gamma} \times 1000
  \]
  
  where \(\gamma=2\). This emphasizes risk differentiation and spreads scores for better interpretability.

- **Output:** A CSV file (`wallet_risk_scores_borrowing.csv`) listing each wallet and its corresponding risk score.

---

## Justification of Risk Indicators Used

- **Transaction Failure & Activity:** Wallets with frequent failed transactions or abnormally low activity often signal risk, misuse, or abandoned accounts—key early warnings in credit risk monitoring.
- **Borrowing Amount and Exposure:** Total borrowed amount and borrowing frequency quantify credit exposure—a fundamental risk driver in DeFi lending.
- **Leverage Ratio (Borrow-to-Collateral):** High leverage correlates strongly with default likelihood in both traditional and decentralized finance.
- **Recent Borrow Growth:** Rapid debt accumulation can precede insolvency or liquidation events.
- **Monetary Flow Ratios and Values:** On-chain inflow/outflow ratios and transaction sizes highlight unusual financial behaviors potentially linked to laundering, collateral manipulation, or volatility.
- **Behavioral Regularity (Time Between Transactions):** Erratic or bursty transaction timing may indicate bot activity or risky behavior.

