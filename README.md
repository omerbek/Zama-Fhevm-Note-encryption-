# 🔐 Blockchain Note Simulator  

Welcome to **Blockchain Note Simulator** – a demo dApp that shows how notes can be securely “stored” on-chain using hashed values.  

⚠️ **Disclaimer:** This is only a simulation. It does **not** use real Full Homomorphic Encryption (FHE).  

---

## ✏️ How It Works  

### Write a Note  
1. Type your note in the input box (e.g., `My secret password is 123456`).  
2. Click **Encrypt & Save to Blockchain**.  
3. Your note is hashed and stored on the **Sepolia Testnet** via a smart contract.  
4. The contract’s `setNote()` function saves it **permanently on-chain**.  

### Retrieve a Note  
- Click **Retrieve Note Information**.  
- You will **NOT** see the original note.  

👉 Why?  
- Only the **hash** is stored.  
- Original text is never saved.  
- Hashes (SHA-3) are **one-way functions**, impossible to reverse.  

---

## 🔍 Example  

**Your input:**  
```
"Meeting my friend today"
```  

**Stored on blockchain:**  
```
0x8f4a5b3c9d2e1f0a8b7c6d5e4f3a2b1c0d9e8f7a6b5c4d3e2f1a0b9c8d7e6f5
```  

**What you see when retrieving:**  
```
Last updated: 12/25/2024, 14:30:45
```  

---

## 🔄 Simulation vs Real Zama FHE  

**Current Simulation**  
✅ Notes are hashed & secured on-chain  
✅ Nobody can read your note  
❌ You cannot read your own note  
❌ Not real encryption  

**With Real Zama FHE**  
✅ Notes are fully encrypted on-chain  
✅ Nobody can read your note  
✅ You can decrypt with your private key  
✅ True **Homomorphic Encryption**  

---

# 🚀 Zama-FHEVM Note Encryption  

## 📂 Create Project Directory  

```bash
mkdir zama
cd zama
```

## 📥 Clone Zama Template  

```bash
git clone https://github.com/zama-ai/fhevm-hardhat-template
```

## 📦 Install Dependencies & FHEVM SDK  

```bash
npm install
npm install @fhevm @fhevm-web3sdk/fhevm-web3sdk
```

---

## 📝 Smart Contract Development  

Create `contracts/EncryptedNotebook.sol`:  

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@fhevm/lib/TFHE.sol";

contract EncryptedNotebook {
    mapping(address => euint32) private userNotes;
    
    event NoteUpdated(address indexed user, string message);

    function setNote(euint32 calldata encryptedNote) public {
        userNotes[msg.sender] = encryptedNote;
        emit NoteUpdated(msg.sender, "Note encrypted and stored successfully!");
    }

    function getNote() public view returns (euint32) {
        return userNotes[msg.sender];
    }
    
    function getBalance() public view returns (uint256) {
        return address(this).balance;
    }
}
```

---

## ⚙️ Configure Hardhat  

Edit `hardhat.config.js`:  

```js
require("@nomicfoundation/hardhat-toolbox");

/** @type import('hardhat/config').HardhatUserConfig */
module.exports = {
  solidity: {
    version: "0.8.20",
    settings: {
      optimizer: { enabled: true, runs: 200 },
    },
  },
  networks: {
    // Zama Testnet
    zamatest: {
      url: "https://devnet.zama.ai",
      chainId: 8009,
      accounts: process.env.PRIVATE_KEY ? [process.env.PRIVATE_KEY] : [],
    },
    // Local development
    localhost: {
      url: "http://127.0.0.1:8545",
      chainId: 31337,
    }
  },
  paths: {
    sources: "./contracts",
    tests: "./test",
    cache: "./cache",
    artifacts: "./artifacts"
  },
};
```

---

## 📜 Create Deployment Script  

`scripts/deploy.js`:  

```js
const { ethers } = require("hardhat");
const { FhevmInstances } = require("@fhevm/web3sdk");

async function main() {
  console.log("🚀 Deploying Zama EncryptedNotebook dApp...");
  
  const EncryptedNotebook = await ethers.getContractFactory("EncryptedNotebook");
  const notebook = await EncryptedNotebook.deploy();
  await notebook.waitForDeployment();

  const address = await notebook.getAddress();
  console.log("✅ EncryptedNotebook deployed at:", address);
  console.log("📝 Tx hash:", notebook.deploymentTransaction().hash);
  
  const instance = await FhevmInstances.createInstance(ethers.provider);
  console.log("🔑 FHEVM Instance created successfully");
  
  return { notebook, instance, address };
}

main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error("❌ Deployment failed:", error);
    process.exit(1);
  });
```

---

## 🔧 Environment Variables  

Create `.env`:  

```bash
# Sepolia RPC URL (Infura or Alchemy)
SEPOLIA_RPC_URL=https://eth-sepolia.g.alchemy.com

# Private Key (0x...)
PRIVATE_KEY=0x2fc6888

# Etherscan API Key (for contract verification)
ETHERSCAN_API_KEY=BBVX99PXCF

# Frontend port
FRONTEND_PORT=5300
```

---

## 🚀 Deploy Contract  

```bash
npx hardhat run scripts/deploy.js --network zamatest
```

---

## 🎨 Frontend Setup  

```bash
mkdir zama-frontend
cd zama-frontend
npx create-react-app .
npm install ethers @fhevm/web3sdk axios
```

Then edit `src/App.js` with the provided frontend code.  

---

✨ Done! You now have a working simulation of **Encrypted Notes on Blockchain with Zama FHEVM template**.  
