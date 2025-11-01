# TokenPresaleApp

A decentralized application (dApp) for managing a **token presale**.  
It combines **Solidity smart contracts** with a **Vite-powered frontend** to allow users to securely purchase tokens before the official launch.

---

## Tech Stack

- **Smart Contracts:** Solidity (ERC-20, presale logic)
- **Frontend:** React + Vite
- **Blockchain Interaction:** ethers.js
- **Build Tools:** Hardhat or Truffle (for contract deployment)
- **Wallets Supported:** MetaMask, WalletConnect

---

## Getting Started

### 1. Clone Repository
```bash
git clone https://github.com/trustllabs/Token-Presale-dApp.git
cd Token-Presale-dApp
```

### 2. Open the Project in Visual Studio Code
```bash
Launch Visual Studio Code and open the project directory to begin development.
```

### 3. Install Dependencies
```bash
npm install
```

### 4. Setup Environment Variables
Copy `.env.default` → `.env` and configure:

```env
VITE_ALCHEMY_API_KEY=your-key
VITE_CONTRACT_ADDRESS=0xYourDeployedPresaleAddress
VITE_TOKEN_ADDRESS=0xYourTokenAddress
VITE_NETWORK=sepolia
```

### 5. Run Development Server
```bash
npm run start
```
Frontend runs at `http://localhost:5173/`

### 6. Build for Production
```bash
npm run build
```

---

## Future Roadmap: Staking Platform

Our goal is to **evolve TokenStaking into a full rewards platform** after the presale.  

Planned features:
- **Staking Contract**
  - Users can stake tokens for rewards.
  - Multiple staking pools (different lock periods, APYs).
- **Reward Distribution**
  - Stakers earn additional or partner tokens.
- **Frontend Dashboard**
  - Display staked balance, rewards, and APR.
  - Claim/unstake functions.
- **Security**
  - Slashing protection.
  - Audit-ready architecture.

This will allow us to transition from **token distribution** → **long-term ecosystem utility**.