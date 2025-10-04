# Whale-Tx-Trap
```markdown
# 🐋 Whale Transaction Alert Trap

A Drosera trap that monitors and alerts on large token transfers (whale movements) on the Hoodi testnet.

![Drosera](https://img.shields.io/badge/Drosera-Network-blue)
![Solidity](https://img.shields.io/badge/Solidity-0.8.20-green)
![License](https://img.shields.io/badge/License-MIT-yellow)

## 📋 Overview

This trap detects and records whale transactions (large token transfers) on the blockchain. When a transfer exceeds the configured threshold, it triggers an alert and stores the transaction details on-chain.

### Features

- ✅ **Real-time Monitoring**: Continuously watches for large transfers
- ✅ **Configurable Threshold**: Default 1000 tokens (easily adjustable)
- ✅ **Alert History**: Stores all whale movements on-chain
- ✅ **Query API**: Retrieve alerts by count, index, or latest batch
- ✅ **Address Tracking**: Count alerts per wallet address

### Deployed Contracts (Hoodi Testnet)

| Contract | Address |
|----------|---------|
| WhaleTransferResponse | `0xE5701AE464d94449461D224b1f11D5b55be1EC0f` |
| WhaleTransferTrap | `0x9b06F678c4df0eF1282b03FF9FE804444F513d26` |

**Network**: Hoodi Testnet (Chain ID: 560048)  
**Whale Threshold**: 1000 tokens (1000 × 10¹⁸ wei)

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────┐
│          Hoodi Blockchain Network               │
│                                                  │
│  ┌──────────────┐         ┌──────────────┐     │
│  │   Transfer   │         │   Transfer   │     │
│  │  (500 tokens)│         │ (1500 tokens)│     │
│  └──────┬───────┘         └──────┬───────┘     │
│         │                        │              │
│         ▼                        ▼              │
│  ┌─────────────────────────────────────┐       │
│  │    WhaleTransferTrap.sol            │       │
│  │  ┌──────────────────────────────┐   │       │
│  │  │  collect() - Get transfers   │   │       │
│  │  │  shouldRespond() - Check     │   │       │
│  │  │  if > 1000 tokens            │   │       │
│  │  └──────────────────────────────┘   │       │
│  └──────────────┬──────────────────────┘       │
│                 │                               │
│                 │ Threshold exceeded            │
│                 ▼                               │
│  ┌─────────────────────────────────────┐       │
│  │  WhaleTransferResponse.sol          │       │
│  │  ┌──────────────────────────────┐   │       │
│  │  │  recordAlert(from, to,       │   │       │
│  │  │    amount, blockNumber)      │   │       │
│  │  │  ✅ Alert stored on-chain    │   │       │
│  │  └──────────────────────────────┘   │       │
│  └─────────────────────────────────────┘       │
└─────────────────────────────────────────────────┘
```

---

## 📁 Project Structure

```
whale-transaction-trap/
├── src/
│   ├── WhaleTransferTrap.sol          # Main trap logic
│   ├── WhaleTransferResponse.sol      # Alert recorder
│   └── IWhaleTransferResponse.sol     # Interface
├── script/
│   └── Deploy.s.sol                   # Deployment script
├── test/
│   └── (test files)
├── drosera.toml                       # Trap configuration
├── foundry.toml                       # Foundry config
├── .env.example                       # Environment template
├── .gitignore
└── README.md
```

---

## 🚀 Quick Start

### Prerequisites

- Ubuntu/Linux environment (WSL works)
- At least 4GB RAM
- Ethereum private key with Hoodi testnet ETH
- Git installed

### Installation

1. **Clone the repository**
```bash
git clone https://github.com/YOUR_USERNAME/whale-transaction-trap.git
cd whale-transaction-trap
```

2. **Install Foundry**
```bash
curl -L https://foundry.paradigm.xyz | bash
source ~/.bashrc
foundryup
```

3. **Install dependencies**
```bash
forge install foundry-rs/forge-std --no-commit
```

4. **Setup environment**
```bash
cp .env.example .env
nano .env  # Add your private key
```

5. **Build contracts**
```bash
forge build
```

---

## 🔧 Deployment Guide

### Step 1: Deploy Contracts

```bash
source .env

forge script script/Deploy.s.sol:DeployScript \
  --rpc-url $HOODI_RPC_URL \
  --private-key $PRIVATE_KEY \
  --broadcast -vvvv
```

**Save the contract addresses from the output!**

### Step 2: Update Configuration

Edit `drosera.toml` and update:
- `response_contract` = Your deployed Response contract address
- `whitelist` = Your operator wallet address

### Step 3: Apply Trap Config

```bash
DROSERA_PRIVATE_KEY=your_key drosera apply
```

### Step 4: Setup Operator

See full operator setup in [Drosera Documentation](https://dev.drosera.io/)

---

## 🔍 Query Alerts

### Get Alert Count
```bash
cast call 0xE5701AE464d94449461D224b1f11D5b55be1EC0f \
  "getAlertCount()(uint256)" \
  --rpc-url https://ethereum-hoodi-rpc.publicnode.com
```

### Get Specific Alert
```bash
cast call 0xE5701AE464d94449461D224b1f11D5b55be1EC0f \
  "getAlert(uint256)((address,address,uint256,uint256,uint256))" 0 \
  --rpc-url https://ethereum-hoodi-rpc.publicnode.com
```

### Get Latest 5 Alerts
```bash
cast call 0xE5701AE464d94449461D224b1f11D5b55be1EC0f \
  "getLatestAlerts(uint256)((address,address,uint256,uint256,uint256)[])" 5 \
  --rpc-url https://ethereum-hoodi-rpc.publicnode.com
```

### Check Whale Threshold
```bash
cast call 0x9b06F678c4df0eF1282b03FF9FE804444F513d26 \
  "getThreshold()(uint256)" \
  --rpc-url https://ethereum-hoodi-rpc.publicnode.com
```

---

## 🎯 How It Works

1. **collect()**: Gathers transfer data from blockchain
2. **shouldRespond()**: Checks if transfer amount ≥ 1000 tokens
3. **recordAlert()**: Stores whale transaction details on-chain
4. **Events**: Emits `AlertRecorded` event for off-chain monitoring

### Alert Data Structure

```solidity
struct Alert {
    address from;        // Sender address
    address to;          // Receiver address
    uint256 amount;      // Transfer amount
    uint256 blockNumber; // Block when detected
    uint256 timestamp;   // Unix timestamp
}
```

---

## ⚙️ Customization

### Change Whale Threshold

Edit `src/WhaleTransferTrap.sol`:

```solidity
uint256 public constant WHALE_THRESHOLD = 5000 * 10**18; // 5000 tokens
```

Then rebuild and redeploy:
```bash
forge build
forge script script/Deploy.s.sol:DeployScript --rpc-url $HOODI_RPC_URL --private-key $PRIVATE_KEY --broadcast -vvvv
```

### Monitor Specific Tokens

Extend the `collect()` function to filter by token address.

---

## 📊 Dashboard

View your trap on the Drosera Dashboard:
1. Visit: https://app.drosera.io/
2. Connect wallet
3. Switch to Hoodi network
4. Search for your trap by address

---

## 🧪 Testing

Run tests:
```bash
forge test -vvv
```

Run specific test:
```bash
forge test --match-test testWhaleThreshold -vvv
```

---

## 📚 Resources

- **Drosera Docs**: https://dev.drosera.io/
- **Drosera Discord**: https://discord.com/invite/drosera
- **Hoodi Testnet**: https://github.com/eth-clients/hoodi
- **Foundry Book**: https://book.getfoundry.sh/

---

## 🔐 Security

- Never commit `.env` files
- Keep private keys secure
- Use environment variables for sensitive data
- Audit contracts before mainnet deployment

---

## 🤝 Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

---

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## 🙏 Acknowledgments

- Built with [Drosera Network](https://drosera.io/)
- Powered by [Foundry](https://getfoundry.sh/)
- Deployed on Hoodi Testnet

---

## 🎉 Status

✅ **Active** - Monitoring whale transactions on Hoodi testnet

---

Made with ❤️ for the Drosera community
